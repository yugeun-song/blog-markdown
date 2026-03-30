# 리눅스 네트워킹 스택 — sk_buff에서 소켓까지

리눅스의 네트워킹 스택은 커널에서 가장 복잡한 서브시스템 중 하나다. 네트워크 인터페이스 카드(NIC)에 도착한 패킷이 애플리케이션의 `recv()` 호출까지 전달되는 과정을 단계별로 추적해 본다.

## sk_buff: 패킷의 컨테이너

`sk_buff`(socket buffer)는 네트워킹 스택에서 패킷을 표현하는 핵심 구조체다. 패킷 데이터 자체뿐만 아니라 프로토콜 헤더 포인터, 라우팅 정보, 타임스탬프 등 다양한 메타데이터를 포함한다.

```c
struct sk_buff {
    struct sk_buff      *next, *prev;
    struct sock         *sk;
    struct net_device   *dev;

    unsigned char       *head, *data, *tail, *end;

    __u16               transport_header;
    __u16               network_header;
    __u16               mac_header;
    // ... 수십 개의 필드
};
```

`head`부터 `end`까지가 할당된 버퍼 공간이고, `data`부터 `tail`까지가 현재 유효한 패킷 데이터다. 프로토콜 계층을 올라가며 `skb_pull()`로 헤더를 벗기고, 내려가며 `skb_push()`로 헤더를 추가한다.

### 제로카피와 scatter-gather

성능 최적화를 위해 `sk_buff`는 실제 데이터를 연속된 메모리에 두지 않을 수 있다. `skb_shared_info` 구조체의 `frags[]` 배열을 통해 여러 페이지에 분산된 데이터를 참조할 수 있으며, 이것이 scatter-gather I/O의 기반이다.

`sendfile()` 시스템 콜이나 TCP zero-copy 전송은 이 메커니즘을 활용한다.

## 패킷 수신 경로

패킷이 NIC에 도착하면 다음과 같은 경로를 거친다.

### NAPI와 인터럽트 처리

현대 NIC 드라이버는 NAPI(New API)를 사용한다. 첫 패킷 도착 시 하드웨어 인터럽트가 발생하고, 이후 인터럽트를 비활성화한 채 폴링 모드로 전환하여 배치 처리한다.

```c
// NAPI 폴링 함수 (드라이버 구현)
static int my_poll(struct napi_struct *napi, int budget)
{
    int work_done = 0;

    while (work_done < budget) {
        struct sk_buff *skb = get_next_packet(priv);
        if (!skb)
            break;

        skb->protocol = eth_type_trans(skb, netdev);
        napi_gro_receive(napi, skb);
        work_done++;
    }

    if (work_done < budget)
        napi_complete_done(napi, work_done);

    return work_done;
}
```

`budget`은 한 번의 폴링에서 처리할 최대 패킷 수로, 기본값은 64다. 처리할 패킷이 남아있으면 소프트IRQ 컨텍스트에서 다시 폴링이 스케줄된다.

### GRO와 패킷 병합

`napi_gro_receive()`는 Generic Receive Offload를 수행한다. 같은 플로우에 속하는 작은 패킷들을 하나의 큰 패킷으로 병합하여 상위 스택의 처리 오버헤드를 줄인다.

병합된 패킷은 `netif_receive_skb()`를 통해 프로토콜 핸들러로 전달된다.

## 넷필터와 라우팅

IP 계층에서 패킷은 넷필터 훅 포인트를 거친다. `iptables`나 `nftables` 규칙이 이 훅에 등록되어 패킷 필터링, NAT, 마킹 등을 수행한다.

```bash
# nftables로 패킷 흐름 확인
sudo nft add rule inet filter input \
    meta nfproto ipv4 tcp dport 80 counter accept

# 카운터로 매칭된 패킷 수 확인
sudo nft list ruleset
```

주요 훅 포인트는 `PREROUTING` → `INPUT`(로컬 전달) 또는 `FORWARD`(포워딩) → `POSTROUTING` 순서로 실행된다.

### 라우팅 테이블 조회

`ip_rcv()`에서 넷필터 PREROUTING 훅을 거친 후, `ip_route_input()`으로 라우팅 결정이 이루어진다. FIB(Forwarding Information Base)를 조회하여 패킷이 로컬로 전달될지, 다른 인터페이스로 포워딩될지 결정한다.

## TCP 계층 처리

로컬 전달 패킷은 `tcp_v4_rcv()`에서 소켓을 찾고, 시퀀스 번호 검증, 윈도우 관리, ACK 처리 등을 수행한다. 최종적으로 데이터는 소켓의 수신 버퍼(`sk->sk_receive_queue`)에 적재된다.

애플리케이션이 `recv()`를 호출하면 `tcp_recvmsg()`가 수신 큐에서 데이터를 유저스페이스 버퍼로 복사한다. 큐가 비어있으면 태스크는 슬립 상태에 들어간다.

### TCP 튜닝 파라미터

네트워크 성능 튜닝에 자주 사용되는 커널 파라미터들이 있다.

```bash
# 소켓 버퍼 크기 (최소/기본/최대)
sysctl net.ipv4.tcp_rmem="4096 131072 6291456"
sysctl net.ipv4.tcp_wmem="4096 16384  4194304"

# TCP 혼잡 제어 알고리즘
sysctl net.ipv4.tcp_congestion_control=bbr

# 커넥션 추적 테이블 크기
sysctl net.netfilter.nf_conntrack_max=262144
```

고대역폭 환경에서는 버퍼 크기를 키우고 BBR 혼잡 제어를 사용하는 것이 일반적인 최적화 패턴이다.
