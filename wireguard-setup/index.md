# WireGuard VPN 구성 — 키 생성부터 라우팅까지

WireGuard는 차세대 VPN 프로토콜이다. 코드베이스가 4,000줄에 불과하면서도(OpenVPN의 100분의 1), Noise 프로토콜 프레임워크 기반의 강력한 암호화를 제공한다. Linux 5.6부터 커널에 내장되어 설치조차 필요 없다.

## WireGuard의 특징

WireGuard는 UDP 기반으로 동작하며, 연결 상태를 유지하지 않는 스테이트리스 설계를 채택했다. 키 교환, 암호화, 인증이 Noise_IK 핸드셰이크로 통합되어 있다.

사용하는 암호화 프리미티브는 고정되어 있다. Curve25519(키 교환), ChaCha20-Poly1305(대칭 암호화/인증), BLAKE2s(해싱), SipHash24(해시테이블). 알고리즘 협상 과정이 없어 공격 표면이 줄어든다.

### OpenVPN과 비교

| 항목 | WireGuard | OpenVPN |
|------|-----------|---------|
| 코드 줄 수 | ~4,000 | ~100,000 |
| 암호화 | 고정 (최신 프리미티브) | 협상 가능 (구식 포함) |
| 프로토콜 | UDP | UDP/TCP |
| 성능 | 커널 공간 실행 | 유저 공간 실행 |
| 핸드셰이크 | 1-RTT | 다중 RTT |

## 키 생성

WireGuard는 SSH와 유사한 공개키 인증을 사용한다. 각 피어가 키 페어를 생성하고, 서로의 공개키를 교환한다.

```bash
# 서버 키 생성
wg genkey | tee /etc/wireguard/server_private.key | wg pubkey > /etc/wireguard/server_public.key
chmod 600 /etc/wireguard/server_private.key

# 클라이언트 키 생성
wg genkey | tee client_private.key | wg pubkey > client_public.key

# 선택: 사전 공유 키 (양자 컴퓨팅 대비 추가 보안)
wg genpsk > preshared.key
```

프라이빗 키는 절대 네트워크로 전송되지 않는다. 공개키만 설정 파일에 명시하면 된다.

### 키 관리 주의사항

프라이빗 키 파일의 퍼미션은 반드시 `600`(소유자만 읽기/쓰기)으로 설정해야 한다. `wg-quick`은 느슨한 퍼미션을 감지하면 경고를 출력한다.

## 서버 설정

서버 측 WireGuard 인터페이스를 구성한다. `/etc/wireguard/wg0.conf` 파일을 작성한다.

```ini
# /etc/wireguard/wg0.conf (서버)
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <서버 프라이빗 키>
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
# 클라이언트 1
PublicKey = <클라이언트 공개키>
PresharedKey = <사전 공유 키>  # 선택사항
AllowedIPs = 10.0.0.2/32
```

`PostUp`/`PostDown`은 인터페이스 활성화/비활성화 시 실행되는 명령이다. NAT 마스커레이딩을 설정하여 VPN 클라이언트가 서버를 통해 인터넷에 접근할 수 있게 한다.

### IP 포워딩 활성화

서버가 라우터 역할을 하려면 IP 포워딩이 필수다.

```bash
# 즉시 활성화
sysctl -w net.ipv4.ip_forward=1

# 영구 설정
echo "net.ipv4.ip_forward = 1" > /etc/sysctl.d/99-wireguard.conf
sysctl --system
```

## 클라이언트 설정

클라이언트 측 설정은 서버보다 간단하다.

```ini
# /etc/wireguard/wg0.conf (클라이언트)
[Interface]
Address = 10.0.0.2/24
PrivateKey = <클라이언트 프라이빗 키>
DNS = 1.1.1.1, 9.9.9.9

[Peer]
PublicKey = <서버 공개키>
PresharedKey = <사전 공유 키>
Endpoint = server.example.com:51820
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 25
```

`AllowedIPs = 0.0.0.0/0`은 모든 트래픽을 VPN으로 라우팅한다(풀 터널). 특정 서브넷만 VPN으로 보내려면 해당 대역만 지정한다(스플릿 터널).

### 연결 시작과 확인

```bash
# 인터페이스 활성화
sudo wg-quick up wg0

# 상태 확인
sudo wg show
# interface: wg0
#   public key: ...
#   listening port: 51820
#
# peer: ...
#   endpoint: 203.0.113.1:51820
#   allowed ips: 0.0.0.0/0
#   latest handshake: 12 seconds ago
#   transfer: 1.23 MiB received, 456.78 KiB sent

# 부팅 시 자동 시작
sudo systemctl enable wg-quick@wg0
```

`latest handshake`가 표시되면 연결이 정상이다. WireGuard는 핸드셰이크를 2분마다 갱신하며, 데이터가 없으면 핸드셰이크도 하지 않는다(스텔스 특성).

## 고급 구성

### 다중 피어와 사이트 간 VPN

여러 클라이언트를 하나의 서버에 연결하거나, 사이트 간 VPN을 구성할 수 있다. 각 피어의 `AllowedIPs`로 라우팅 테이블이 자동 생성된다.

```ini
# 서버에 두 번째 피어 추가
[Peer]
PublicKey = <클라이언트2 공개키>
AllowedIPs = 10.0.0.3/32

# 사이트 간 VPN — 원격 네트워크 대역 포함
[Peer]
PublicKey = <원격 사이트 공개키>
AllowedIPs = 10.0.0.3/32, 192.168.1.0/24
Endpoint = remote.example.com:51820
```

### 모바일 환경

WireGuard는 로밍을 자연스럽게 지원한다. 클라이언트의 IP가 바뀌어도(Wi-Fi에서 셀룰러로 전환 등) 서버 측에서 자동으로 새 엔드포인트를 인식한다. `PersistentKeepalive = 25`를 설정하면 NAT 뒤의 클라이언트도 연결이 유지된다.
