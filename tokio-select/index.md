# tokio::select! 로 비동기 I/O 분기 처리하기

비동기 프로그래밍에서 가장 흔한 패턴 중 하나는 "여러 작업을 동시에 기다리되, 먼저 끝나는 것부터 처리"하는 것입니다. Rust의 tokio 런타임은 이를 위해 `select!` 매크로를 제공합니다.

## 기본 개념

`tokio::select!`는 여러 비동기 브랜치를 동시에 폴링하고, 가장 먼저 완료된 브랜치의 결과를 실행합니다. 나머지 브랜치는 드롭됩니다.

```rust
use tokio::time::{sleep, Duration};
use tokio::sync::mpsc;

#[tokio::main]
async fn main() {
    let (tx, mut rx) = mpsc::channel::<String>(32);

    // 생산자: 2초 후에 메시지 전송
    tokio::spawn(async move {
        sleep(Duration::from_secs(2)).await;
        tx.send("hello from task".to_string()).await.unwrap();
    });

    tokio::select! {
        msg = rx.recv() => {
            println!("메시지 수신: {:?}", msg);
        }
        _ = sleep(Duration::from_secs(5)) => {
            println!("5초 타임아웃");
        }
    }
}
```

위 코드에서 `select!`는 채널 수신과 타임아웃을 동시에 기다립니다. 2초 후 메시지가 도착하면 첫 번째 브랜치가 실행되고, 타임아웃 Future는 자동으로 드롭됩니다.

## 취소 안전성 (Cancel Safety)

`select!`의 핵심 주의사항은 **취소 안전성**입니다. 선택되지 않은 브랜치의 Future는 드롭되기 때문에, 부분적으로 진행된 상태가 유실될 수 있습니다.

```rust
use tokio::io::AsyncReadExt;
use tokio::net::TcpStream;

async fn read_exact_or_timeout(
    stream: &mut TcpStream,
    buf: &mut [u8],
    timeout: Duration,
) -> Result<(), &'static str> {
    tokio::select! {
        // 주의: read_exact는 cancel-safe하지 않음!
        result = stream.read_exact(buf) => {
            result.map_err(|_| "read failed")?;
            Ok(())
        }
        _ = sleep(timeout) => {
            Err("timeout")
        }
    }
}
```

`read_exact`는 cancel-safe하지 않습니다. 타임아웃이 먼저 발생하면 일부 바이트만 읽힌 상태에서 Future가 드롭되어 데이터가 유실됩니다.

### cancel-safe한 대안

```rust
use tokio::io::AsyncBufReadExt;
use tokio::io::BufReader;

async fn read_line_or_timeout(
    reader: &mut BufReader<TcpStream>,
    timeout: Duration,
) -> Result<String, &'static str> {
    let mut line = String::new();
    tokio::select! {
        // read_line은 cancel-safe
        result = reader.read_line(&mut line) => {
            result.map_err(|_| "read failed")?;
            Ok(line)
        }
        _ = sleep(timeout) => {
            Err("timeout")
        }
    }
}
```

## 루프 안에서의 select!

장기 실행 서비스에서는 `loop` 안에서 `select!`를 사용하는 패턴이 일반적입니다.

```rust
use tokio::sync::mpsc;
use tokio::signal;

async fn run_service(mut rx: mpsc::Receiver<String>) {
    let mut msg_count: u64 = 0;

    loop {
        tokio::select! {
            Some(msg) = rx.recv() => {
                msg_count += 1;
                println!("[{}] 처리: {}", msg_count, msg);
            }
            _ = signal::ctrl_c() => {
                println!("종료 신호 수신, 총 {} 메시지 처리", msg_count);
                break;
            }
        }
    }
}
```

여기서 `rx.recv()`는 cancel-safe합니다. 채널에서 아직 값을 꺼내지 않은 상태로 드롭되기 때문에 데이터 유실이 없습니다.

## biased 모드

기본적으로 `select!`는 브랜치를 무작위 순서로 폴링합니다. 특정 브랜치에 우선순위를 주려면 `biased`를 사용합니다.

```rust
use tokio::sync::mpsc;

async fn drain_priority(
    mut high: mpsc::Receiver<String>,
    mut low: mpsc::Receiver<String>,
) {
    loop {
        tokio::select! {
            biased;

            Some(msg) = high.recv() => {
                println!("[HIGH] {}", msg);
            }
            Some(msg) = low.recv() => {
                println!("[LOW]  {}", msg);
            }
            else => break,
        }
    }
}
```

`biased` 모드에서는 위에서 아래로 순서대로 폴링합니다. 고우선순위 채널에 메시지가 있으면 항상 먼저 처리됩니다.

## 정리

| 항목 | 설명 |
|------|------|
| `select!` | 여러 Future 중 첫 완료를 처리 |
| Cancel safety | 드롭 시 상태 유실 여부 확인 필수 |
| `biased` | 브랜치 우선순위 지정 |
| 루프 패턴 | 장기 서비스에서 이벤트 루프 구성 |

`select!`는 강력하지만, 각 브랜치의 cancel safety를 항상 확인해야 합니다. tokio 공식 문서의 [cancel safety 표](https://docs.rs/tokio/latest/tokio/)를 참고하세요.
