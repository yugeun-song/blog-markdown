# eBPF 입문 — 커널을 재컴파일하지 않고 관찰하기

eBPF(extended Berkeley Packet Filter)는 리눅스 커널 내에서 샌드박스된 프로그램을 실행할 수 있게 해주는 기술이다. 원래 패킷 필터링 용도였지만, 현재는 트레이싱, 보안, 네트워킹 등 거의 모든 커널 서브시스템에서 활용된다.

## eBPF란 무엇인가

eBPF 프로그램은 커널 내부의 가상 머신에서 실행되는 바이트코드다. 커널에 로드되기 전에 검증기(verifier)가 안전성을 체크하므로, 커널 패닉을 유발하지 않는다는 것이 보장된다.

### 아키텍처 개요

eBPF 프로그램은 이벤트 기반으로 동작한다. 특정 커널 이벤트(kprobe, tracepoint, 소켓 이벤트 등)에 연결되어, 해당 이벤트 발생 시 자동으로 실행된다.

```
User Space                    Kernel Space
┌─────────┐                  ┌──────────────┐
│ bpftool │──bpf()───────────│ BPF Verifier │
│ libbpf  │  syscall         │      ↓       │
│ bcc     │                  │  JIT Compile  │
└─────────┘                  │      ↓       │
                             │ Attach to    │
                             │ hook point   │
                             └──────────────┘
```

검증 통과 후 JIT 컴파일러가 네이티브 기계어로 변환하므로 실행 성능이 뛰어나다.

## bpftrace로 빠르게 시작하기

bpftrace는 eBPF를 위한 고수준 트레이싱 언어로, awk와 유사한 문법을 제공한다. 복잡한 C 코드 없이 한 줄짜리 명령으로 커널 이벤트를 분석할 수 있다.

```bash
# 시스템 콜 빈도 상위 10개
sudo bpftrace -e 'tracepoint:raw_syscalls:sys_enter {
    @syscall[args->id] = count();
}
END { print(@syscall, 10); }'

# 프로세스별 read 바이트 합계
sudo bpftrace -e 'tracepoint:syscalls:sys_exit_read /args->ret > 0/ {
    @bytes[comm] = sum(args->ret);
}'
```

`tracepoint`은 커널 소스에 정적으로 정의된 훅 포인트로, 커널 버전 간 안정적인 인터페이스를 제공한다. `kprobe`는 거의 모든 커널 함수에 동적으로 붙일 수 있지만 인터페이스 안정성이 보장되지 않는다.

### 유용한 원라이너

시스템 분석에 자주 쓰이는 bpftrace 명령들이다.

```bash
# TCP 연결 추적
sudo bpftrace -e 'kprobe:tcp_connect {
    printf("%s → %s\n", comm, ntop(((struct sock *)arg0)->__sk_common.skc_daddr));
}'

# 블록 I/O 레이턴시 히스토그램
sudo bpftrace -e 'tracepoint:block:block_rq_issue {
    @start[args->dev, args->sector] = nsecs;
}
tracepoint:block:block_rq_complete /@start[args->dev, args->sector]/ {
    @usecs = hist((nsecs - @start[args->dev, args->sector]) / 1000);
    delete(@start[args->dev, args->sector]);
}'
```

## libbpf와 CO-RE

프로덕션 환경에서는 libbpf를 사용한 C 기반 eBPF 프로그램이 선호된다. CO-RE(Compile Once, Run Everywhere)를 통해 다양한 커널 버전에서 재컴파일 없이 동작한다.

```c
// hello.bpf.c — eBPF 커널 프로그램
#include <linux/bpf.h>
#include <bpf/bpf_helpers.h>

SEC("tracepoint/syscalls/sys_enter_execve")
int trace_execve(struct trace_event_raw_sys_enter *ctx)
{
    char comm[16];
    bpf_get_current_comm(&comm, sizeof(comm));
    bpf_printk("execve: %s", comm);
    return 0;
}

char LICENSE[] SEC("license") = "GPL";
```

`SEC()` 매크로로 프로그램 타입과 연결 포인트를 지정한다. `bpf_printk`의 출력은 `/sys/kernel/debug/tracing/trace_pipe`에서 확인할 수 있다.

### BPF 맵

eBPF 프로그램과 유저스페이스 간 데이터 교환은 BPF 맵을 통해 이루어진다. 해시맵, 배열, 링버퍼 등 다양한 맵 타입이 제공된다.

```c
struct {
    __uint(type, BPF_MAP_TYPE_HASH);
    __uint(max_entries, 1024);
    __type(key, u32);    /* PID */
    __type(value, u64);  /* count */
} syscall_count SEC(".maps");
```

유저스페이스에서는 `bpf_map_lookup_elem()`으로 맵 데이터를 읽어 대시보드에 표시하거나 파일로 저장할 수 있다.

## 실전 활용 사례

eBPF는 이제 단순한 트레이싱 도구를 넘어선다. Cilium은 eBPF 기반 Kubernetes 네트워킹과 보안을 제공하고, Falco는 런타임 보안 모니터링에 eBPF를 활용한다.

성능 분석 도구인 `perf`도 eBPF를 백엔드로 사용할 수 있으며, systemd-oomd는 eBPF를 통해 메모리 압박 상태를 모니터링한다.

### 제한사항과 주의점

eBPF 검증기는 무한 루프, 범위 밖 메모리 접근, 과도한 스택 사용을 거부한다. 프로그램 크기는 최대 100만 인스트럭션, 스택은 512바이트로 제한된다. 이런 제약 내에서 프로그래밍하는 것이 eBPF 개발의 핵심 도전이다.
