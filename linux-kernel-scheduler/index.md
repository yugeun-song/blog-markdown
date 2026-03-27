# CFS 스케줄러 깊이 읽기 — 레드블랙 트리와 가상 런타임

리눅스 커널의 Completely Fair Scheduler(CFS)는 2.6.23부터 기본 스케줄러로 사용되고 있다. 이름 그대로 "완전히 공정한" 스케줄링을 목표로 하며, 이를 위해 가상 런타임(vruntime)이라는 핵심 개념을 도입했다.

## 가상 런타임의 개념

CFS의 핵심 아이디어는 단순하다. 모든 태스크가 동일한 양의 CPU 시간을 받아야 한다. 실제로는 우선순위(nice 값)에 따라 가중치가 다르므로, 물리적 실행 시간 대신 가상 런타임을 추적한다.

```c
// kernel/sched/fair.c 에서 vruntime 계산 (간략화)
static inline u64 calc_delta_fair(u64 delta, struct sched_entity *se)
{
    if (unlikely(se->load.weight != NICE_0_LOAD))
        delta = __calc_delta(delta, NICE_0_LOAD, &se->load);
    return delta;
}
```

nice 0인 태스크는 물리 시간 = 가상 시간이다. nice 값이 낮을수록(우선순위 높음) 가상 시간이 느리게 증가하여 더 많은 실제 CPU 시간을 받게 된다.

### 가중치 테이블

CFS는 nice 값(-20 ~ +19)을 가중치로 변환하는 테이블을 사용한다. 한 단계 차이가 약 10%의 CPU 시간 차이를 만든다.

```c
static const int prio_to_weight[40] = {
 /* -20 */ 88761, 71755, 56483, 46273, 36291,
 /* -15 */ 29154, 23254, 18705, 14949, 11916,
 /* -10 */  9548,  7620,  6100,  4904,  3906,
 /*  -5 */  3121,  2501,  1991,  1586,  1277,
 /*   0 */  1024,   820,   655,   526,   423,
 /*   5 */   335,   272,   215,   172,   137,
 /*  10 */   110,    87,    70,    56,    45,
 /*  15 */    36,    29,    23,    18,    15,
};
```

nice 0의 가중치가 1024이고, nice -1은 1277이다. 이 비율이 CPU 시간 분배의 기준이 된다.

## 레드블랙 트리 기반 런큐

CFS는 실행 대기 중인 태스크를 vruntime 기준으로 정렬된 레드블랙 트리에 저장한다. 스케줄링 결정 시 트리의 최좌측 노드(vruntime이 가장 작은 태스크)를 선택하면 된다.

레드블랙 트리의 삽입, 삭제, 탐색 모두 O(log n)이므로, 태스크 수가 많아져도 스케줄링 오버헤드가 예측 가능하다. 이전 O(1) 스케줄러보다 이론적 복잡도는 높지만, 실제로는 캐시 친화적인 구조 덕분에 성능이 우수하다.

### 스케줄링 결정 과정

타이머 인터럽트가 발생할 때마다 `scheduler_tick()`이 호출되고, 현재 태스크의 vruntime이 갱신된다. 이후 `check_preempt_tick()`에서 현재 태스크의 실행 시간이 이상적 시간 슬라이스를 초과했는지 검사한다.

```c
static void check_preempt_tick(struct cfs_rq *cfs_rq,
                               struct sched_entity *curr)
{
    unsigned long ideal_runtime, delta_exec;

    ideal_runtime = sched_slice(cfs_rq, curr);
    delta_exec = curr->sum_exec_runtime - curr->prev_sum_exec_runtime;

    if (delta_exec > ideal_runtime)
        resched_curr(rq_of(cfs_rq));
}
```

초과했다면 `TIF_NEED_RESCHED` 플래그를 설정하여 다음 기회에 컨텍스트 스위치가 일어나도록 한다.

## 그룹 스케줄링

cgroup을 통해 태스크 그룹별로 CPU 대역폭을 제한할 수 있다. CFS는 계층적 구조를 지원하여 그룹 단위의 공정성도 보장한다.

```bash
# cgroup v2에서 CPU 대역폭 제한
echo "50000 100000" > /sys/fs/cgroup/mygroup/cpu.max
# 100ms 주기 중 50ms만 사용 가능 → 50% CPU
```

이 기능은 컨테이너 환경에서 필수적이다. Kubernetes의 `resources.limits.cpu`가 정확히 이 메커니즘을 사용한다.

### NUMA 인식 스케줄링

멀티소켓 시스템에서 CFS는 NUMA 토폴로지를 고려한다. 원격 노드 메모리 접근은 로컬 대비 수배 느리므로, 태스크를 데이터가 있는 노드에 배치하려는 자동 NUMA 밸런싱이 동작한다.

`/proc/sys/kernel/numa_balancing` 값이 1이면 활성화된 상태이며, 커널이 주기적으로 페이지 접근 패턴을 분석하여 태스크 마이그레이션을 결정한다.

## EEVDF: CFS의 후계자

커널 6.6부터 EEVDF(Earliest Eligible Virtual Deadline First)가 CFS를 대체하기 시작했다. vruntime 기반 공정성에 더해 데드라인 개념을 도입하여, 레이턴시에 민감한 태스크를 더 잘 처리한다.

기존 CFS에서 `sched_latency`와 `sched_min_granularity` 같은 튜닝 파라미터에 의존하던 부분이 EEVDF에서는 자연스러운 데드라인 계산으로 대체된다. 장기적으로 더 직관적이고 예측 가능한 스케줄링이 가능해질 전망이다.

### 실시간 스케줄링 클래스

CFS(SCHED_NORMAL)와 별도로, 리눅스는 SCHED_FIFO와 SCHED_RR 실시간 스케줄링 클래스를 제공한다. 이들은 CFS보다 항상 높은 우선순위를 가지며, 오디오 처리나 산업 제어 같은 엄격한 시간 요구사항에 사용된다.

`chrt` 명령으로 프로세스의 스케줄링 정책을 변경할 수 있다.
