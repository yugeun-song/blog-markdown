# TCP 혼잡 제어 알고리즘 비교 — Reno에서 BBR까지

인터넷의 안정성은 TCP 혼잡 제어에 크게 의존한다. 모든 송신자가 가능한 한 빠르게 데이터를 보내면 네트워크는 즉시 붕괴된다. 혼잡 제어 알고리즘은 네트워크 용량을 공정하게 분배하면서 높은 처리량을 유지하는 역할을 한다.

## 혼잡 제어의 기본 개념

TCP 송신자는 혼잡 윈도우(cwnd)를 유지한다. 한 번에 전송할 수 있는 미확인 데이터의 최대량이다. 혼잡 제어 알고리즘은 네트워크 상태를 추정하여 이 cwnd를 조절한다.

```
전송률 ≈ cwnd / RTT

cwnd = 10 segments, RTT = 50ms → 약 200 segments/sec
cwnd = 100 segments, RTT = 50ms → 약 2000 segments/sec
```

핵심은 cwnd를 얼마나 빠르게 증가시키고, 혼잡을 감지했을 때 얼마나 줄이느냐다.

### Slow Start

모든 혼잡 제어 알고리즘은 Slow Start로 시작한다. 초기 cwnd(보통 10 MSS)에서 시작하여, 매 ACK마다 1 MSS씩 증가한다. 결과적으로 RTT마다 cwnd가 두 배로 늘어나는 지수 증가다.

```python
# Slow Start 시뮬레이션
cwnd = 10  # 초기 cwnd (MSS 단위)
ssthresh = 64  # Slow Start 임계값

for rtt in range(10):
    if cwnd < ssthresh:
        cwnd *= 2  # Slow Start: 지수 증가
        print(f"RTT {rtt}: cwnd={cwnd} (slow start)")
    else:
        cwnd += 1  # Congestion Avoidance: 선형 증가
        print(f"RTT {rtt}: cwnd={cwnd} (congestion avoidance)")
```

cwnd가 `ssthresh`(slow start threshold)에 도달하면 혼잡 회피(congestion avoidance) 단계로 전환된다.

## 손실 기반 알고리즘

### TCP Reno

Reno는 패킷 손실을 혼잡의 신호로 해석한다. 3개의 중복 ACK를 받으면 cwnd를 절반으로 줄이고(fast recovery), 타임아웃이 발생하면 cwnd를 1로 리셋한다.

AIMD(Additive Increase, Multiplicative Decrease) 패턴으로, 혼잡이 없으면 RTT마다 1 MSS씩 선형 증가, 손실 시 절반으로 감소한다. 톱니 형태의 cwnd 그래프가 특징이다.

### TCP CUBIC

CUBIC은 Linux의 기본 혼잡 제어 알고리즘이다. Reno의 선형 증가를 3차 함수(cubic function)로 대체하여, 높은 대역폭 환경에서 더 빠르게 용량을 활용한다.

```bash
# 현재 혼잡 제어 알고리즘 확인
sysctl net.ipv4.tcp_congestion_control
# net.ipv4.tcp_congestion_control = cubic

# 사용 가능한 알고리즘 목록
sysctl net.ipv4.tcp_available_congestion_control
# cubic reno bbr
```

CUBIC의 윈도우 함수는 `W(t) = C * (t - K)^3 + W_max`로, 이전 최대 윈도우(`W_max`) 근처에서는 천천히 탐색하고, 멀리서는 빠르게 접근한다.

## BBR: 모델 기반 접근

Google이 개발한 BBR(Bottleneck Bandwidth and Round-trip propagation time)은 패킷 손실 대신 대역폭과 RTT를 직접 측정하여 최적 전송률을 계산한다.

```bash
# BBR 활성화
sudo sysctl -w net.ipv4.tcp_congestion_control=bbr
sudo sysctl -w net.core.default_qdisc=fq

# BBR 상태 확인
ss -tin | grep bbr
# bbr wscale:7,7 rto:204 rtt:2.5/0.5 ... delivery_rate 125.0Mbps
```

BBR의 목표는 `BtlBw * RTprop` — 병목 대역폭과 최소 RTT의 곱 — 만큼 데이터를 인플라이트로 유지하는 것이다. 이 지점이 최대 처리량과 최소 지연을 동시에 달성하는 최적점이다.

### BBR의 상태 머신

BBR은 네 가지 상태를 순환한다.

1. **Startup**: 대역폭을 빠르게 탐색 (Slow Start와 유사)
2. **Drain**: Startup에서 쌓인 큐를 비움
3. **ProbeBW**: 대역폭 변화를 주기적으로 탐색
4. **ProbeRTT**: 최소 RTT를 주기적으로 재측정

ProbeBW 상태에서는 8 RTT 주기로 전송률을 1.25배, 0.75배, 1배로 변동시켜 가용 대역폭의 변화를 감지한다.

## 알고리즘 비교

| 특성 | Reno | CUBIC | BBR |
|------|------|-------|-----|
| 혼잡 신호 | 패킷 손실 | 패킷 손실 | 대역폭/RTT 측정 |
| 윈도우 증가 | 선형 | 3차 함수 | 모델 기반 |
| 버퍼블로트 | 유발 | 유발 | 완화 |
| 손실 환경 | 민감 | 민감 | 내성 있음 |

### 어떤 알고리즘을 선택할까

대부분의 서버 환경에서 BBR(v3)이 좋은 선택이다. 특히 장거리 고대역폭 링크에서 CUBIC 대비 월등한 성능을 보인다. 하지만 공정성 문제(BBR이 CUBIC 플로우의 대역폭을 빼앗는 현상)가 완전히 해결되지는 않았으므로, 환경에 맞게 테스트해야 한다.

`iperf3`로 A/B 테스트를 수행하면 실제 환경에서의 성능 차이를 확인할 수 있다.
