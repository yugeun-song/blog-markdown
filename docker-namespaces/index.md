# 컨테이너의 기반 — Linux namespace와 cgroup 직접 다루기

Docker, Podman 등 컨테이너 런타임의 기반 기술은 Linux 커널의 namespace와 cgroup이다. 이 두 메커니즘을 직접 다뤄보면 컨테이너가 "마법"이 아님을 이해할 수 있다.

## Linux Namespace

namespace는 커널 리소스를 격리하는 메커니즘이다. 각 namespace 안의 프로세스는 자신만의 독립된 리소스 뷰를 가진다.

현재 Linux는 8종의 namespace를 제공한다.

| Namespace | 격리 대상 | 플래그 |
|-----------|----------|--------|
| Mount | 파일시스템 마운트 | `CLONE_NEWNS` |
| UTS | 호스트명 | `CLONE_NEWUTS` |
| IPC | IPC 리소스 | `CLONE_NEWIPC` |
| PID | 프로세스 ID | `CLONE_NEWPID` |
| Network | 네트워크 스택 | `CLONE_NEWNET` |
| User | UID/GID | `CLONE_NEWUSER` |
| Cgroup | cgroup 루트 | `CLONE_NEWCGROUP` |
| Time | 시스템 시계 | `CLONE_NEWTIME` |

### unshare로 namespace 생성

`unshare` 명령으로 새 namespace를 생성하고 그 안에서 셸을 실행할 수 있다.

```bash
# 새 UTS + PID namespace에서 셸 실행
sudo unshare --uts --pid --mount-proc --fork bash

# 호스트명 변경 — 호스트에 영향 없음
hostname container-test
hostname
# container-test

# PID 1번이 bash
ps aux
# USER  PID  %CPU %MEM  COMMAND
# root    1   0.0  0.0  bash
# root   12   0.0  0.0  ps aux
```

PID namespace 안에서 `ps aux`를 실행하면 해당 namespace의 프로세스만 보인다. PID 1은 일반적으로 init이지만, 여기서는 우리의 bash가 PID 1이다.

### Network namespace

네트워크 namespace는 가상 네트워크 인터페이스, 라우팅 테이블, iptables 규칙을 격리한다. `veth` 페어로 namespace 간 통신을 구성한다.

```bash
# 네트워크 namespace 생성
sudo ip netns add myns

# veth 페어 생성 및 한쪽을 namespace에 할당
sudo ip link add veth0 type veth peer name veth1
sudo ip link set veth1 netns myns

# IP 주소 설정
sudo ip addr add 10.0.0.1/24 dev veth0
sudo ip link set veth0 up

sudo ip netns exec myns ip addr add 10.0.0.2/24 dev veth1
sudo ip netns exec myns ip link set veth1 up
sudo ip netns exec myns ip link set lo up

# 통신 확인
sudo ip netns exec myns ping 10.0.0.1
```

Docker의 bridge 네트워킹이 정확히 이 구조다. 각 컨테이너가 network namespace를 가지고, veth 페어로 호스트의 `docker0` 브릿지에 연결된다.

## cgroup v2

cgroup(control group)은 프로세스 그룹의 리소스 사용량을 제한하고 모니터링하는 메커니즘이다. cgroup v2는 통합된 계층 구조를 제공한다.

```bash
# cgroup v2 마운트 확인
mount | grep cgroup2
# cgroup2 on /sys/fs/cgroup type cgroup2

# 새 cgroup 생성
sudo mkdir /sys/fs/cgroup/mycontainer

# CPU 제한: 100ms 중 50ms만 사용 (50%)
echo "50000 100000" | sudo tee /sys/fs/cgroup/mycontainer/cpu.max

# 메모리 제한: 256MB
echo "268435456" | sudo tee /sys/fs/cgroup/mycontainer/memory.max

# 프로세스 할당
echo $$ | sudo tee /sys/fs/cgroup/mycontainer/cgroup.procs
```

### 리소스 모니터링

cgroup은 제한뿐만 아니라 사용량 통계도 제공한다.

```bash
# CPU 사용 통계
cat /sys/fs/cgroup/mycontainer/cpu.stat
# usage_usec 123456789
# user_usec 100000000
# system_usec 23456789

# 메모리 사용 통계
cat /sys/fs/cgroup/mycontainer/memory.current
# 52428800

# I/O 통계
cat /sys/fs/cgroup/mycontainer/io.stat
```

Docker의 `docker stats` 명령이 보여주는 리소스 사용량이 바로 이 cgroup 통계에서 오는 것이다.

## 미니 컨테이너 만들기

namespace와 cgroup을 조합하면 Docker 없이도 기본적인 컨테이너를 만들 수 있다. 루트 파일시스템만 있으면 된다.

```bash
# Alpine Linux 루트 파일시스템 다운로드
mkdir -p /tmp/rootfs
curl -sL https://dl-cdn.alpinelinux.org/alpine/v3.19/releases/x86_64/alpine-minirootfs-3.19.0-x86_64.tar.gz \
    | tar xz -C /tmp/rootfs

# 모든 namespace 격리 + chroot
sudo unshare --mount --uts --ipc --pid --fork \
    chroot /tmp/rootfs /bin/sh -c '
    mount -t proc proc /proc
    mount -t sysfs sys /sys
    hostname mini-container
    echo "Hello from container!"
    exec /bin/sh
'
```

이것이 컨테이너의 본질이다. OCI 런타임(runc)도 결국 이 과정을 더 정교하고 안전하게 자동화한 것에 불과하다.

### 보안 고려사항

프로덕션 환경에서는 seccomp 프로파일로 시스템 콜을 제한하고, AppArmor/SELinux로 추가 접근 제어를 적용한다. user namespace를 사용한 rootless 컨테이너는 루트 권한 없이도 격리를 제공하여 보안을 강화한다.
