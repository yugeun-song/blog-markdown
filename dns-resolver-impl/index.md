# DNS 리졸버 직접 구현하기 — UDP 패킷 파싱부터 캐시까지

DNS(Domain Name System)는 인터넷의 전화번호부다. 도메인 이름을 IP 주소로 변환하는 이 시스템은 겉보기에 단순하지만, 프로토콜 수준에서 들여다보면 정교한 설계가 숨어있다. 직접 리졸버를 구현하며 DNS의 내부를 이해해 보자.

## DNS 패킷 구조

DNS 메시지는 질의(query)와 응답(response) 모두 동일한 형식을 사용한다. 헤더(12바이트) + Question + Answer + Authority + Additional 섹션으로 구성된다.

```
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                      ID                         |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|QR|   Opcode  |AA|TC|RD|RA|   Z    |   RCODE   |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                    QDCOUNT                      |
|                    ANCOUNT                      |
|                    NSCOUNT                      |
|                    ARCOUNT                      |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
```

헤더의 주요 필드: `ID`는 요청-응답 매칭용, `QR`은 질의/응답 구분, `RD`는 재귀 요청, `RCODE`는 응답 코드(0=성공, 3=NXDOMAIN)다.

### Python으로 DNS 패킷 생성

```python
import struct
import socket

def build_query(domain: str, qtype: int = 1) -> bytes:
    """DNS 질의 패킷을 생성한다."""
    # 헤더
    header = struct.pack(
        "!HHHHHH",
        0x1234,  # ID
        0x0100,  # Flags: RD=1 (재귀 요청)
        1,       # QDCOUNT
        0, 0, 0  # ANCOUNT, NSCOUNT, ARCOUNT
    )

    # Question 섹션
    question = b""
    for label in domain.split("."):
        question += bytes([len(label)]) + label.encode()
    question += b"\x00"  # 루트 레이블

    question += struct.pack("!HH", qtype, 1)  # QTYPE, QCLASS(IN)

    return header + question
```

도메인 이름은 길이-접두사 레이블의 연속으로 인코딩된다. `example.com`은 `\x07example\x03com\x00`이 된다.

## 응답 파싱

DNS 응답의 리소스 레코드를 파싱하는 것이 구현의 핵심이다. 특히 이름 압축(name compression) 처리가 까다롭다.

```python
def parse_name(data: bytes, offset: int) -> tuple[str, int]:
    """DNS 이름을 파싱한다. 포인터 압축을 처리한다."""
    labels = []
    jumped = False
    original_offset = offset

    while True:
        length = data[offset]

        if length == 0:
            offset += 1
            break

        # 상위 2비트가 11이면 포인터 (압축)
        if (length & 0xC0) == 0xC0:
            if not jumped:
                original_offset = offset + 2
            pointer = struct.unpack("!H", data[offset:offset+2])[0] & 0x3FFF
            offset = pointer
            jumped = True
            continue

        offset += 1
        labels.append(data[offset:offset+length].decode())
        offset += length

    final_offset = original_offset if jumped else offset
    return ".".join(labels), final_offset
```

이름 압축은 중복되는 도메인 접미사를 포인터로 참조하여 패킷 크기를 줄인다. 이 최적화는 UDP 512바이트 제한(EDNS 이전) 시절에 특히 중요했다.

### 리소스 레코드 파싱

```python
def parse_record(data: bytes, offset: int) -> tuple[dict, int]:
    """리소스 레코드 하나를 파싱한다."""
    name, offset = parse_name(data, offset)
    rtype, rclass, ttl, rdlength = struct.unpack(
        "!HHIH", data[offset:offset+10]
    )
    offset += 10

    if rtype == 1:  # A 레코드
        ip = socket.inet_ntoa(data[offset:offset+4])
        rdata = ip
    elif rtype == 28:  # AAAA 레코드
        ip = socket.inet_ntop(socket.AF_INET6, data[offset:offset+16])
        rdata = ip
    elif rtype in (2, 5):  # NS, CNAME
        rdata, _ = parse_name(data, offset)
    else:
        rdata = data[offset:offset+rdlength].hex()

    return {"name": name, "type": rtype, "ttl": ttl, "data": rdata}, offset + rdlength
```

## 재귀 리졸빙

재귀 리졸버는 루트 네임서버부터 시작하여 도메인 계층을 따라가며 최종 답을 찾는다.

```python
ROOT_SERVERS = ["198.41.0.4", "199.9.14.201", "192.33.4.12"]

def resolve(domain: str, qtype: int = 1) -> str:
    """재귀적으로 도메인을 리졸브한다."""
    nameserver = ROOT_SERVERS[0]

    while True:
        response = send_query(nameserver, domain, qtype)

        # Answer가 있으면 완료
        if response["answers"]:
            return response["answers"][0]["data"]

        # NS 레코드의 glue record에서 다음 네임서버 IP 획득
        if response["additionals"]:
            for rec in response["additionals"]:
                if rec["type"] == 1:  # A 레코드
                    nameserver = rec["data"]
                    break
        elif response["authorities"]:
            # glue가 없으면 NS 이름을 먼저 리졸브
            ns_name = response["authorities"][0]["data"]
            nameserver = resolve(ns_name)
        else:
            raise Exception(f"Resolution failed for {domain}")
```

`example.com`을 조회하면: 루트 → `.com` TLD 서버 → `example.com` 권한 서버 순으로 3번의 왕복이 필요하다.

### TTL 기반 캐싱

실전 리졸버에서 캐시는 필수다. 각 레코드의 TTL(Time To Live)을 존중하여 캐시 만료를 관리한다.

```python
import time

class DNSCache:
    def __init__(self):
        self._cache = {}

    def put(self, key: str, records: list[dict]):
        min_ttl = min(r["ttl"] for r in records)
        self._cache[key] = {
            "records": records,
            "expires": time.time() + min_ttl,
        }

    def get(self, key: str) -> list[dict] | None:
        entry = self._cache.get(key)
        if entry and time.time() < entry["expires"]:
            return entry["records"]
        return None
```

캐싱 덕분에 반복 질의는 네트워크 왕복 없이 즉시 응답할 수 있다. 대부분의 인기 도메인 TTL은 수분에서 수시간이다.

## 보안: DNSSEC과 DoH

평문 DNS는 스푸핑과 도청에 취약하다. DNSSEC은 응답의 무결성을 암호학적으로 검증하고, DNS over HTTPS(DoH)와 DNS over TLS(DoT)는 통신 자체를 암호화한다.

직접 구현한 리졸버에 이런 보안 계층을 추가하는 것은 상당히 복잡하므로, 프로덕션에서는 Unbound나 CoreDNS 같은 검증된 구현을 사용하는 것이 바람직하다.
