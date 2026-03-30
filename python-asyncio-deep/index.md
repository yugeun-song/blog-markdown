# asyncio 이벤트 루프 내부 — 셀렉터와 코루틴 스케줄링

Python의 `asyncio`는 단일 스레드에서 수천 개의 동시 연결을 처리할 수 있는 비동기 I/O 프레임워크다. GIL의 제약 안에서도 높은 I/O 처리량을 달성하는 비결은 이벤트 루프와 코루틴의 협력적 스케줄링에 있다.

## 이벤트 루프의 구조

asyncio의 핵심은 이벤트 루프다. 무한 루프 안에서 I/O 이벤트를 감시하고, 준비된 콜백을 실행하고, 예약된 타이머를 처리한다.

```python
# 이벤트 루프의 핵심 동작 (간략화)
class SimpleEventLoop:
    def __init__(self):
        self._ready = collections.deque()    # 즉시 실행할 콜백
        self._scheduled = []                  # 타이머 (힙큐)
        self._selector = selectors.DefaultSelector()

    def run_forever(self):
        while True:
            # 1. 타이머에서 만료된 콜백을 _ready로 이동
            self._process_timers()

            # 2. I/O 이벤트 대기 (select/epoll)
            timeout = self._calculate_timeout()
            events = self._selector.select(timeout)

            # 3. I/O 콜백을 _ready에 추가
            for key, mask in events:
                self._ready.append(key.data)

            # 4. 준비된 콜백 실행
            while self._ready:
                callback = self._ready.popleft()
                callback()
```

실제 CPython의 `asyncio`는 이보다 훨씬 복잡하지만, 기본 원리는 동일하다. I/O 멀티플렉싱과 콜백 큐가 핵심이다.

### 셀렉터 백엔드

Linux에서는 `epoll`, macOS에서는 `kqueue`, Windows에서는 `IOCP`가 사용된다. `selectors` 모듈이 플랫폼별 최적 구현을 자동 선택한다.

```python
import selectors
import socket

sel = selectors.DefaultSelector()
print(type(sel))  # <class 'selectors.EpollSelector'> (Linux)

# 소켓을 셀렉터에 등록
sock = socket.socket()
sock.setblocking(False)
sel.register(sock, selectors.EVENT_READ, data=my_callback)
```

epoll은 등록된 파일 디스크립터 수에 관계없이 O(1) 시간에 이벤트를 감지하므로, 수만 개의 동시 연결도 효율적으로 처리할 수 있다.

## 코루틴과 Task

`async def`로 정의된 코루틴은 `await` 지점에서 실행을 양보하고, 이벤트 루프가 다른 코루틴을 실행할 수 있게 한다.

```python
import asyncio

async def fetch_data(url: str) -> bytes:
    reader, writer = await asyncio.open_connection(url, 80)
    writer.write(b"GET / HTTP/1.1\r\nHost: " + url.encode() + b"\r\n\r\n")
    await writer.drain()

    data = await reader.read(4096)
    writer.close()
    await writer.wait_closed()
    return data

async def main():
    # 세 개의 요청이 동시에 진행
    results = await asyncio.gather(
        fetch_data("example.com"),
        fetch_data("httpbin.org"),
        fetch_data("jsonplaceholder.typicode.com"),
    )
    for r in results:
        print(f"Received {len(r)} bytes")

asyncio.run(main())
```

`asyncio.gather`는 여러 코루틴을 동시에 스케줄링한다. 각 코루틴이 I/O 대기(`await`)에 들어가면 다른 코루틴이 실행된다.

### Task의 내부

`asyncio.Task`는 코루틴을 감싸는 래퍼로, `Future`를 상속한다. Task가 생성되면 이벤트 루프의 `_ready` 큐에 추가되어 다음 루프 반복에서 실행된다.

```python
async def my_coroutine():
    await asyncio.sleep(1)
    return "done"

# Task 생성 — 즉시 스케줄링됨
task = asyncio.create_task(my_coroutine())

# Task는 Future이므로 완료 콜백 등록 가능
task.add_done_callback(lambda t: print(t.result()))
```

코루틴의 `send(None)` 호출로 실행이 시작되고, `await`에서 `StopIteration`이 아닌 다른 Future가 yield되면 해당 Future의 완료를 기다리도록 설정된다.

## 프로토콜과 트랜스포트

asyncio의 저수준 API는 프로토콜-트랜스포트 패턴을 사용한다. 트랜스포트는 데이터 전송 방법을, 프로토콜은 데이터 처리 로직을 담당한다.

```python
class EchoProtocol(asyncio.Protocol):
    def connection_made(self, transport):
        self.transport = transport
        peername = transport.get_extra_info("peername")
        print(f"Connection from {peername}")

    def data_received(self, data):
        self.transport.write(data)  # 에코

    def connection_lost(self, exc):
        print("Connection closed")

async def main():
    loop = asyncio.get_running_loop()
    server = await loop.create_server(EchoProtocol, "0.0.0.0", 8888)
    async with server:
        await server.serve_forever()
```

이 패턴은 콜백 기반이라 코루틴보다 복잡하지만, 바이트 수준의 세밀한 제어가 필요할 때 유용하다.

### Streams API

대부분의 경우 고수준 Streams API(`asyncio.open_connection`, `asyncio.start_server`)가 더 편리하다. 내부적으로 프로토콜-트랜스포트를 감싸고 있으며, `async`/`await`와 자연스럽게 통합된다.

## 디버깅과 주의사항

asyncio 코드에서 가장 흔한 실수는 블로킹 호출을 코루틴 안에서 직접 수행하는 것이다. `time.sleep()`, 동기 HTTP 요청 등은 이벤트 루프 전체를 멈추게 한다.

```python
# 나쁜 예 — 이벤트 루프를 1초간 블록
async def bad():
    time.sleep(1)  # asyncio.sleep(1)을 사용해야 함

# 디버그 모드 활성화
asyncio.run(main(), debug=True)
# 100ms 이상 블로킹 호출 시 경고 출력
```

`asyncio.run(debug=True)`를 사용하면 느린 콜백, 해제되지 않은 리소스 등을 감지할 수 있다. 프로덕션에서는 `PYTHONASYNCIODEBUG=1` 환경 변수를 활용한다.
