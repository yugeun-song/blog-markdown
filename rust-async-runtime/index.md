# 직접 만드는 async 런타임 — Waker와 Future의 원리

Rust의 `async`/`await`는 문법적 설탕이다. 컴파일러가 async 함수를 상태 머신으로 변환하고, 런타임이 이를 폴링하여 실행한다. tokio나 async-std 같은 런타임의 핵심 메커니즘을 이해하기 위해, 최소한의 런타임을 직접 구현해 본다.

## Future 트레이트

모든 async 코드의 기반은 `Future` 트레이트다. 단 하나의 메서드 `poll`로 구성된다.

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

`Poll::Ready(value)`는 완료를, `Poll::Pending`은 아직 결과가 준비되지 않았음을 의미한다. `Pending`을 반환할 때는 반드시 나중에 `Waker`를 통해 다시 폴링해 달라고 요청해야 한다.

### async fn의 변환

`async fn`은 컴파일러에 의해 `Future`를 구현하는 익명 구조체로 변환된다. 각 `.await` 포인트가 상태 전이점이 된다.

```rust
// 이 코드는:
async fn fetch_data() -> String {
    let conn = connect().await;
    let data = conn.read().await;
    data
}

// 대략 이런 상태 머신으로 변환된다:
enum FetchData {
    State0 { /* 초기 상태 */ },
    State1 { conn: Connection },  // connect() 완료 후
    Done,
}
```

## Waker와 깨우기 메커니즘

`Waker`는 Future가 진행 가능해졌을 때 런타임에 알리는 메커니즘이다. 내부적으로 vtable 기반의 타입 소거를 사용한다.

```rust
use std::task::{RawWaker, RawWakerVTable, Waker};

fn dummy_raw_waker() -> RawWaker {
    fn no_op(_: *const ()) {}
    fn clone(ptr: *const ()) -> RawWaker {
        dummy_raw_waker()
    }

    let vtable = &RawWakerVTable::new(clone, no_op, no_op, no_op);
    RawWaker::new(std::ptr::null(), vtable)
}

fn dummy_waker() -> Waker {
    unsafe { Waker::from_raw(dummy_raw_waker()) }
}
```

실제 런타임에서는 `Waker`가 태스크 큐에 해당 태스크를 다시 넣는 로직을 수행한다. epoll이나 io_uring에서 I/O 완료 이벤트가 발생하면 해당 Waker를 호출한다.

### Context와 폴링

`Context`는 현재 `Waker`를 전달하는 컨테이너다. Future가 `Pending`을 반환하기 전에 이 Waker를 저장해 두었다가, 나중에 진행 가능해지면 `wake()`를 호출한다.

## 최소 런타임 구현

이제 실제로 동작하는 최소한의 런타임을 구현해 보자.

```rust
use std::collections::VecDeque;
use std::future::Future;
use std::pin::Pin;
use std::sync::{Arc, Mutex};
use std::task::{Context, Poll, Wake};

struct Task {
    future: Mutex<Pin<Box<dyn Future<Output = ()> + Send>>>,
}

impl Wake for Task {
    fn wake(self: Arc<Self>) {
        // 태스크를 다시 큐에 넣는 로직
        println!("Task woken!");
    }
}

struct MiniRuntime {
    queue: VecDeque<Arc<Task>>,
}

impl MiniRuntime {
    fn new() -> Self {
        Self { queue: VecDeque::new() }
    }

    fn spawn(&mut self, future: impl Future<Output = ()> + Send + 'static) {
        let task = Arc::new(Task {
            future: Mutex::new(Box::pin(future)),
        });
        self.queue.push_back(task);
    }

    fn run(&mut self) {
        while let Some(task) = self.queue.pop_front() {
            let waker = Waker::from(task.clone());
            let mut cx = Context::from_waker(&waker);
            let mut future = task.future.lock().unwrap();

            match future.as_mut().poll(&mut cx) {
                Poll::Ready(()) => {} // 태스크 완료
                Poll::Pending => {
                    drop(future);
                    self.queue.push_back(task);
                }
            }
        }
    }
}
```

이 구현은 완전하지 않지만, 런타임의 핵심 루프를 보여준다. 큐에서 태스크를 꺼내고, 폴링하고, 완료되지 않으면 다시 넣는다.

### 실제 런타임과의 차이

tokio 같은 프로덕션 런타임은 이 구조 위에 여러 최적화를 추가한다. 멀티스레드 work-stealing 스케줄러, epoll/io_uring 기반 I/O 리액터, 타이머 힐 등이다.

## Pin과 자기참조 구조체

`Pin<&mut Self>`가 왜 필요한지 이해하려면, async 상태 머신이 자기참조 구조체가 될 수 있다는 점을 알아야 한다.

```rust
async fn self_referential() {
    let data = vec![1, 2, 3];
    let reference = &data;  // data를 참조
    some_async_op().await;   // 여기서 상태 머신에 data와 reference 모두 저장
    println!("{:?}", reference);
}
```

`.await` 지점에서 `data`와 `reference`가 모두 상태 구조체에 저장되는데, `reference`가 같은 구조체 내의 `data`를 가리키게 된다. 구조체가 메모리에서 이동하면 포인터가 무효화되므로, `Pin`으로 이동을 방지해야 한다.

### Unpin 트레이트

자기참조가 없는 타입은 `Unpin`을 구현하며, `Pin` 안에서도 자유롭게 이동할 수 있다. 대부분의 기본 타입은 `Unpin`이다. 컴파일러가 생성한 async 상태 머신은 자기참조 가능성 때문에 `!Unpin`으로 마킹된다.
