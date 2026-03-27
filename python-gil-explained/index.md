# GIL 완전 해부 — CPython의 동시성 한계와 우회 전략

Python의 GIL(Global Interpreter Lock)은 CPython 구현에서 가장 논쟁적인 설계 결정 중 하나다. 멀티코어 시대에 왜 여전히 존재하며, 어떻게 대처할 수 있는지 깊이 분석한다.

## GIL이란 무엇인가

GIL은 CPython 인터프리터 수준의 뮤텍스다. 한 번에 하나의 스레드만 Python 바이트코드를 실행할 수 있도록 제한한다. 멀티스레드 Python 프로그램이라도 CPU 바운드 작업에서는 실질적으로 단일 코어만 활용된다.

```python
import threading
import time

def cpu_bound(n):
    """CPU 집약적 작업"""
    total = 0
    for i in range(n):
        total += i * i
    return total

# 싱글 스레드
start = time.time()
cpu_bound(50_000_000)
cpu_bound(50_000_000)
print(f"Sequential: {time.time() - start:.2f}s")

# 멀티 스레드 — GIL 때문에 빨라지지 않음
start = time.time()
t1 = threading.Thread(target=cpu_bound, args=(50_000_000,))
t2 = threading.Thread(target=cpu_bound, args=(50_000_000,))
t1.start(); t2.start()
t1.join(); t2.join()
print(f"Threaded: {time.time() - start:.2f}s")
```

두 결과의 시간은 거의 동일하거나, 스레드 버전이 오히려 약간 느리다. GIL 획득/해제와 컨텍스트 스위칭 오버헤드 때문이다.

### GIL이 존재하는 이유

CPython의 메모리 관리는 참조 카운팅 기반이다. 모든 객체의 `ob_refcnt` 필드를 스레드 안전하게 관리하려면, 각 접근마다 원자적 연산이 필요하다. 개별 객체 잠금 대신 글로벌 잠금을 사용하는 것이 단순하고, 단일 스레드 성능을 저해하지 않는다.

또한 많은 C 확장 모듈이 GIL 존재를 가정하고 작성되어, 제거 시 호환성 문제가 크다.

## I/O 바운드에서의 GIL

GIL이 항상 문제인 것은 아니다. I/O 대기 중에는 GIL이 해제되므로, I/O 바운드 작업에서는 멀티스레딩이 효과적이다.

```python
import concurrent.futures
import urllib.request

urls = [
    "https://example.com",
    "https://httpbin.org/get",
    "https://jsonplaceholder.typicode.com/posts/1",
]

def fetch(url):
    with urllib.request.urlopen(url) as response:
        return len(response.read())

# I/O 바운드 — 스레드가 효과적
with concurrent.futures.ThreadPoolExecutor(max_workers=3) as pool:
    results = list(pool.map(fetch, urls))
    print(results)
```

`urllib.request.urlopen`은 내부적으로 소켓 I/O를 수행하며, 이때 GIL을 해제한다. 다른 스레드가 그 동안 Python 코드를 실행할 수 있다.

### GIL 해제 타이밍

CPython 3.12 기준, GIL은 다음 시점에 해제된다.

- I/O 연산 (파일, 소켓, 파이프)
- `time.sleep()` 호출
- C 확장에서 `Py_BEGIN_ALLOW_THREADS` 매크로 사용 시
- 약 5ms마다 자동 전환 (`sys.getswitchinterval()`)

## 우회 전략

### multiprocessing

가장 전통적인 우회법이다. 프로세스마다 독립된 인터프리터(와 GIL)를 가지므로, 진정한 병렬 실행이 가능하다.

```python
from multiprocessing import Pool

def cpu_bound(n):
    return sum(i * i for i in range(n))

with Pool(processes=4) as pool:
    results = pool.map(cpu_bound, [10_000_000] * 4)
    print(f"Total: {sum(results)}")
```

단점은 프로세스 간 데이터 전달에 직렬화/역직렬화 비용이 발생한다는 것이다. 공유 메모리(`multiprocessing.shared_memory`)로 일부 완화할 수 있다.

### Free-threaded Python (PEP 703)

Python 3.13부터 실험적으로 GIL 없는 빌드(`--disable-gil`)가 제공된다. `python3.13t`로 설치할 수 있으며, 진정한 멀티스레드 병렬 실행이 가능해진다.

```bash
# free-threaded Python 확인
python3.13t -c "import sys; print(sys._is_gil_enabled())"
# False
```

아직 실험 단계이며, 일부 C 확장의 호환성 문제가 있다. 하지만 Python의 미래 방향이 GIL 제거임은 분명하다.

## 성능 측정과 프로파일링

GIL 경합 상태를 진단하려면 `sys.getswitchinterval()`과 함께 `perf`나 `py-spy` 같은 프로파일러를 활용한다.

```bash
# py-spy로 GIL 경합 확인
py-spy top --pid 12345 --gil
```

CPU 바운드 멀티스레드 코드에서 GIL utilization이 100%에 가까우면, multiprocessing이나 C 확장으로의 전환을 고려해야 한다.
