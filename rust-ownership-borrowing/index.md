# 소유권과 빌림 — Rust 메모리 모델의 핵심

Rust는 가비지 컬렉터 없이 메모리 안전성을 보장하는 유일한 주류 프로그래밍 언어다. 이를 가능하게 하는 핵심 메커니즘이 바로 소유권(ownership)과 빌림(borrowing) 시스템이다.

## 소유권의 세 가지 규칙

Rust의 소유권 모델은 세 가지 단순한 규칙으로 요약된다.

1. 모든 값에는 정확히 하나의 소유자(owner)가 있다.
2. 소유자가 스코프를 벗어나면 값이 드롭(drop)된다.
3. 소유권은 이동(move)될 수 있지만, 복사되지 않는다(Copy 트레이트 미구현 시).

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;  // s1의 소유권이 s2로 이동

    // println!("{}", s1);  // 컴파일 에러! s1은 더 이상 유효하지 않음
    println!("{}", s2);     // OK
}
```

`String`은 힙에 할당된 데이터를 가리키는 포인터, 길이, 용량으로 구성된다. 소유권 이동 시 이 세 필드만 복사되고, 힙 데이터는 공유된다. 이중 해제(double free)는 원천적으로 불가능해진다.

### Copy와 Clone

`i32`, `f64`, `bool` 같은 스택 전용 타입은 `Copy` 트레이트를 구현하여 이동 대신 복사가 일어난다. 힙 할당이 없으므로 복사 비용이 매우 저렴하기 때문이다.

```rust
let x = 42;
let y = x;    // Copy — x도 여전히 유효
println!("{} {}", x, y);  // 둘 다 사용 가능

let s1 = String::from("hello");
let s2 = s1.clone();  // 명시적 깊은 복사
println!("{} {}", s1, s2);  // clone 후에는 둘 다 유효
```

`Clone`은 명시적 깊은 복사를 위한 트레이트로, 호출 시 힙 데이터까지 전부 복사한다.

## 빌림(Borrowing)

소유권을 이동시키지 않고 값을 사용하려면 참조(reference)를 빌려야 한다. Rust의 참조는 두 종류다.

```rust
fn calculate_length(s: &String) -> usize {
    s.len()  // 불변 참조로 읽기만 가능
}

fn add_suffix(s: &mut String) {
    s.push_str(" world");  // 가변 참조로 수정 가능
}

fn main() {
    let mut greeting = String::from("hello");

    let len = calculate_length(&greeting);     // 불변 빌림
    add_suffix(&mut greeting);                  // 가변 빌림

    println!("{} ({})", greeting, len);
}
```

불변 참조(`&T`)는 여러 개가 동시에 존재할 수 있지만, 가변 참조(`&mut T`)는 오직 하나만 존재할 수 있다. 이 규칙이 데이터 경쟁을 컴파일 타임에 방지한다.

### 빌림 규칙의 직관

빌림 규칙은 "여러 명이 동시에 읽는 건 OK, 한 명이 쓰는 동안 다른 접근은 금지"라는 읽기-쓰기 잠금(RwLock)의 컴파일 타임 버전이다.

```rust
let mut v = vec![1, 2, 3];
let first = &v[0];       // 불변 빌림
// v.push(4);             // 컴파일 에러! 가변 빌림과 불변 빌림 공존 불가
println!("{}", first);
v.push(4);                // first 사용 후이므로 OK (NLL 덕분)
```

NLL(Non-Lexical Lifetimes)은 Rust 2018부터 도입된 기능으로, 참조의 수명을 렉시컬 스코프가 아닌 실제 사용 범위로 좁혀 더 유연한 코드를 허용한다.

## 라이프타임

참조가 유효한 기간을 라이프타임이라 한다. 대부분의 경우 컴파일러가 자동으로 추론하지만, 함수 시그니처에서 명시해야 하는 경우가 있다.

```rust
// 두 문자열 슬라이스 중 더 긴 것을 반환
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}

fn main() {
    let result;
    let s1 = String::from("long string");
    {
        let s2 = String::from("xyz");
        result = longest(&s1, &s2);
        println!("{}", result);  // OK — s2가 아직 유효
    }
    // println!("{}", result);  // 에러 가능 — s2 드롭 후
}
```

`'a`는 라이프타임 파라미터로, 반환된 참조가 입력 참조들 중 더 짧은 쪽의 수명을 따른다는 것을 표현한다.

### 라이프타임 생략 규칙

세 가지 생략 규칙(elision rules) 덕분에 대부분의 함수에서 라이프타임을 명시하지 않아도 된다. 하지만 구조체에 참조를 담거나, 여러 참조를 받아 참조를 반환하는 함수에서는 명시가 필요하다.

## 소유권 패턴 실전

실제 코드에서 자주 만나는 소유권 관련 패턴들이 있다. `String` vs `&str`, `Vec<T>` vs `&[T]`, 그리고 `Cow<'a, T>`를 통한 지연 복사 등이다.

API를 설계할 때는 "가능한 한 빌려쓰고, 필요할 때만 소유하라"가 기본 원칙이다. `AsRef`, `Into` 트레이트를 활용하면 호출자에게 유연성을 제공할 수 있다.
