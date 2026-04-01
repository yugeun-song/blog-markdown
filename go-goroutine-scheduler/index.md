# Go 스케줄러 내부 — M:N 스레딩과 work stealing

Go의 동시성 모델은 고루틴이라는 경량 스레드에 기반한다. 수십만 개의 고루틴이 소수의 OS 스레드 위에서 효율적으로 실행될 수 있는 비결은 런타임의 GMP 스케줄러에 있다.

## GMP 모델

Go 스케줄러는 세 가지 핵심 엔티티로 구성된다.

- **G (Goroutine)**: 고루틴 — 실행할 코드와 스택을 가진 경량 스레드
- **M (Machine)**: OS 스레드 — 실제 코드를 실행하는 주체
- **P (Processor)**: 논리 프로세서 — 로컬 런큐를 가지며 M과 G를 연결

```text
P0 ─── M0          P1 ─── M1
│                   │
├── G1 (running)    ├── G4 (running)
├── G2 (runnable)   ├── G5 (runnable)
└── G3 (runnable)   └── G6 (runnable)

        Global Run Queue: [G7, G8, G9]
```

P의 수는 `GOMAXPROCS`로 결정되며, 기본값은 CPU 코어 수다. 각 P는 로컬 런큐(최대 256개)를 가지며, 오버플로우 시 글로벌 런큐로 이동한다.

### 고루틴의 스택

고루틴의 초기 스택 크기는 단 2KB(Go 1.4+)다. OS 스레드의 기본 스택이 1~8MB인 것과 비교하면 극히 작다. 스택이 부족해지면 런타임이 자동으로 더 큰 스택을 할당하고 기존 내용을 복사한다(copyable stacks).

```go
// 수십만 개의 고루틴도 문제없다
func main() {
    var wg sync.WaitGroup
    for i := 0; i < 100_000; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            time.Sleep(time.Second)
        }(i)
    }
    wg.Wait()
}
```

100,000개의 고루틴이 각 2KB 스택을 사용하면 약 200MB다. 같은 수의 OS 스레드를 생성하면 수백 GB가 필요할 것이다.

## Work Stealing

P의 로컬 런큐가 비면 다른 P에서 실행 대기 중인 고루틴을 "훔쳐온다". 이것이 work stealing 알고리즘이다.

```text
steal 순서:
1. 로컬 런큐 확인
2. 글로벌 런큐 확인
3. 네트워크 폴러 확인
4. 다른 P의 로컬 런큐에서 절반을 훔쳐옴
```

이 알고리즘은 부하를 자동으로 분산시켜, 특정 P에 작업이 몰리는 상황을 방지한다. 런타임 소스의 `runtime.findRunnable()` 함수에서 이 로직을 확인할 수 있다.

### 스케줄링 포인트

고루틴 전환은 특정 지점에서만 일어난다. 주요 스케줄링 포인트는 다음과 같다.

```go
// 채널 연산
ch <- value     // 블로킹 시 고루틴 전환
value := <-ch   // 블로킹 시 고루틴 전환

// 시스템 콜
syscall.Read()  // M이 블록되면 P를 다른 M에 넘김

// runtime.Gosched()
runtime.Gosched()  // 명시적 양보

// 함수 호출 프롤로그
// Go 1.14+: 비협조적 선점 (시그널 기반)
```

Go 1.14부터는 시그널 기반 비협조적 선점(asynchronous preemption)이 도입되어, 오랫동안 함수 호출 없이 실행되는 루프도 선점할 수 있게 되었다.

## 시스템 콜과 핸드오프

고루틴이 블로킹 시스템 콜을 수행하면, 해당 M은 블록된다. 이때 P를 유휴 M(또는 새 M)에 핸드오프하여 다른 고루틴이 계속 실행될 수 있도록 한다.

```text
[시스템 콜 진입]
P0 ─── M0(blocked, G1)
       ↓ P0 핸드오프
P0 ─── M2(new)
│
├── G2 (now running)
└── G3 (runnable)

[시스템 콜 완료]
M0 ── G1: P 다시 획득 시도
```

네트워크 I/O는 다르게 처리된다. `netpoller`가 epoll/kqueue를 사용하여 비블로킹으로 관리하므로, M이 블록되지 않는다.

### GOMAXPROCS 튜닝

대부분의 경우 기본값(CPU 코어 수)이 최적이다. 하지만 I/O 바운드 워크로드에서는 P 수를 늘려 처리량을 높일 수 있다.

```go
import "runtime"

func init() {
    runtime.GOMAXPROCS(runtime.NumCPU() * 2)
}
```

## 디버깅 도구

Go는 스케줄러 동작을 관찰하기 위한 도구를 내장하고 있다.

```bash
# 스케줄러 트레이스
GODEBUG=schedtrace=1000 ./myapp
# 1000ms마다 스케줄러 상태 출력

# 상세 트레이스
GODEBUG=scheddetail=1,schedtrace=1000 ./myapp

# Go execution tracer
go tool trace trace.out
```

`schedtrace`는 각 P의 상태, 실행 중인 고루틴 수, 글로벌 런큐 길이 등을 출력한다. 고루틴 누수나 성능 병목을 진단할 때 유용하다.
