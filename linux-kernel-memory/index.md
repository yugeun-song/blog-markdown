# 리눅스 커널의 메모리 관리 — 슬랩 할당자와 페이지 캐시

리눅스 커널의 메모리 관리는 하드웨어의 물리 메모리를 효율적으로 추상화하여 프로세스와 커널 자체에 제공하는 핵심 서브시스템이다. 이 글에서는 버디 시스템, 슬랩 할당자, 페이지 캐시를 중심으로 커널의 메모리 관리 전략을 깊이 들여다본다.

## 물리 메모리와 존(Zone)

커널은 물리 메모리를 여러 존으로 나누어 관리한다. x86_64에서 주요 존은 `ZONE_DMA`(0-16MB), `ZONE_DMA32`(0-4GB), `ZONE_NORMAL`(나머지)이다.

각 존은 독립적인 버디 시스템 인스턴스를 가진다. 이 구조는 DMA 전용 메모리처럼 특정 물리 주소 범위가 필요한 할당 요청을 효율적으로 처리하기 위함이다.

### 버디 시스템

버디 시스템은 물리 페이지를 2의 거듭제곱 크기의 블록으로 관리하는 할당자다. 현재 상태는 `/proc/buddyinfo`에서 확인할 수 있다.

```bash
cat /proc/buddyinfo
# Node 0, zone   Normal   1024  512  256  128  64  32  16  8  4  2  1
```

각 숫자는 해당 오더의 사용 가능한 프리 블록 수를 나타낸다. 오더 0은 4KB(단일 페이지), 오더 10은 4MB(1024 페이지)다.

할당 요청이 들어오면 요청 크기 이상의 가장 작은 블록을 찾아 분할하고, 해제 시에는 인접한 버디와 병합을 시도한다. 이 병합 과정이 외부 단편화를 줄이는 핵심 메커니즘이다.

## 슬랩 할당자

버디 시스템은 페이지 단위(4KB)로 할당하므로, 수십~수백 바이트 크기의 커널 오브젝트를 할당하기에는 낭비가 심하다. 슬랩 할당자는 이 문제를 해결한다.

```c
// 커널 내부에서 슬랩 캐시 생성
struct kmem_cache *my_cache;

my_cache = kmem_cache_create("my_objects",
                              sizeof(struct my_object),
                              0,          /* align */
                              SLAB_HWCACHE_ALIGN,
                              NULL);      /* constructor */

// 오브젝트 할당 및 해제
struct my_object *obj = kmem_cache_alloc(my_cache, GFP_KERNEL);
kmem_cache_free(my_cache, obj);
```

슬랩 할당자는 동일한 크기의 오브젝트를 미리 할당된 페이지 내에서 관리한다. `task_struct`, `inode`, `dentry` 등 자주 생성/소멸되는 커널 구조체마다 전용 캐시가 존재한다.

### SLUB 할당자

현재 리눅스의 기본 슬랩 구현은 SLUB이다. 기존 SLAB에 비해 메타데이터 오버헤드가 적고, NUMA 환경에서 더 나은 성능을 보인다.

SLUB의 상태는 `/sys/kernel/slab/` 아래에서 캐시별로 확인할 수 있다. `slabinfo` 명령으로 전체 캐시의 사용량 통계를 볼 수도 있다.

```bash
sudo slabinfo -s | head -10
# Name                   Objects Objsize    Space Slabs/Part/Cpu  O/S O %Fr %Ef Flg
# task_struct                412    9792  4587520      140/6/8   32  2   4  87
# dentry                   18340     192  3731456     1820/12/8  10  0   0  94
```

### kmalloc과 범용 캐시

특정 구조체 전용 캐시가 아닌 범용 메모리 할당에는 `kmalloc`을 사용한다. 내부적으로 `kmalloc-8`, `kmalloc-16`, ... `kmalloc-8192` 등 미리 정의된 크기별 캐시를 활용한다.

`kfree`로 해제하며, `GFP_KERNEL`(슬립 가능), `GFP_ATOMIC`(인터럽트 컨텍스트) 등의 플래그로 할당 컨텍스트를 지정한다.

## 페이지 캐시

페이지 캐시는 디스크 I/O 성능을 극적으로 향상시키는 메커니즘이다. 파일에서 읽은 데이터를 메모리에 캐싱하여 반복 접근 시 디스크를 건너뛴다.

```bash
# 메모리 사용량 확인 — Cached 항목이 페이지 캐시
free -h
#               total   used   free   shared  buff/cache   available
# Mem:           32Gi   8.2Gi  1.5Gi   256Mi      22Gi       23Gi

# 특정 파일의 캐시 상태 확인
vmtouch /var/log/syslog
# Files: 1
# Directories: 0
# Resident Pages: 1024/1024  4M/4M  100%
```

`read()` 시스템 콜이 호출되면 커널은 먼저 페이지 캐시에서 해당 페이지를 찾는다. 없으면 디스크에서 읽어 캐시에 추가(page fault)하고, 있으면 바로 유저 버퍼에 복사한다.

### Writeback 메커니즘

쓰기 작업은 즉시 디스크에 반영되지 않는다. 먼저 페이지 캐시의 페이지를 수정(dirty 마킹)하고, 별도의 writeback 스레드(`kworker`)가 주기적으로 또는 메모리 압박 시 디스크에 기록한다.

`/proc/sys/vm/dirty_ratio`와 `dirty_writeback_centisecs` 파라미터로 writeback 정책을 튜닝할 수 있다. 데이터베이스처럼 내구성이 중요한 경우 `O_DIRECT`로 페이지 캐시를 우회하기도 한다.

## 메모리 회수와 OOM

시스템 메모리가 부족해지면 커널은 적극적으로 메모리를 회수한다. 클린 페이지 캐시 드롭, 슬랩 캐시 수축, 스왑 아웃이 순서대로 시도된다.

모든 회수 노력에도 불구하고 할당이 실패하면 OOM Killer가 개입한다. 각 프로세스의 `oom_score`를 기반으로 가장 많은 메모리를 사용하는 프로세스를 선택해 종료한다.

### cgroup과 메모리 제한

컨테이너 환경에서는 cgroup v2의 `memory.max`로 프로세스 그룹별 메모리 한도를 설정한다. 한도 초과 시 해당 cgroup 내에서만 OOM이 발생하므로, 호스트 전체에 영향을 주지 않는다.

```bash
# cgroup v2 메모리 제한 설정
echo 512M > /sys/fs/cgroup/mygroup/memory.max
echo $$ > /sys/fs/cgroup/mygroup/cgroup.procs
```

이 구조를 이해하면 Docker나 Kubernetes의 메모리 리소스 관리가 어떻게 동작하는지 명확해진다.
