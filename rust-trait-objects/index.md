# 트레이트 객체와 동적 디스패치 — dyn Trait의 내부 구조

Rust에서 다형성을 구현하는 방법은 두 가지다. 제네릭을 통한 정적 디스패치(monomorphization)와 트레이트 객체를 통한 동적 디스패치다. 각각의 트레이드오프를 이해하고, 동적 디스패치의 내부 메커니즘인 vtable을 분석해 본다.

## 정적 디스패치: 제네릭

제네릭 함수는 컴파일 타임에 구체적 타입별로 별도의 코드가 생성된다. 이를 단형화(monomorphization)라 한다.

```rust
trait Draw {
    fn draw(&self);
}

struct Circle { radius: f64 }
struct Rect { width: f64, height: f64 }

impl Draw for Circle {
    fn draw(&self) { println!("○ r={}", self.radius); }
}

impl Draw for Rect {
    fn draw(&self) { println!("□ {}×{}", self.width, self.height); }
}

// 정적 디스패치 — 타입별로 별도 함수가 생성됨
fn render<T: Draw>(shape: &T) {
    shape.draw();
}
```

`render::<Circle>`과 `render::<Rect>` 두 개의 함수가 별도로 생성된다. 함수 호출이 인라인될 수 있어 성능이 우수하지만, 코드 크기가 커질 수 있다.

### 단형화의 비용

제네릭이 많은 타입으로 인스턴스화되면 바이너리 크기가 상당히 증가할 수 있다. 특히 `HashMap<K, V>`처럼 두 개의 타입 파라미터를 가진 제네릭은 조합이 폭발적으로 늘어날 수 있다.

## 동적 디스패치: dyn Trait

런타임에 타입을 결정해야 하는 경우 트레이트 객체를 사용한다. `dyn Trait`는 fat pointer로 구현되며, 데이터 포인터와 vtable 포인터로 구성된다.

```rust
// 동적 디스패치 — 하나의 함수로 모든 타입 처리
fn render_dynamic(shape: &dyn Draw) {
    shape.draw();  // vtable을 통한 간접 호출
}

fn main() {
    let shapes: Vec<Box<dyn Draw>> = vec![
        Box::new(Circle { radius: 5.0 }),
        Box::new(Rect { width: 3.0, height: 4.0 }),
    ];

    for shape in &shapes {
        render_dynamic(shape.as_ref());
    }
}
```

이종 타입을 하나의 컬렉션에 담을 수 있다는 것이 트레이트 객체의 핵심 장점이다. 제네릭만으로는 `Vec<T>`에 서로 다른 타입을 담을 수 없다.

### vtable의 구조

`&dyn Draw`는 메모리에서 두 개의 포인터(16바이트)로 표현된다.

```text
&dyn Draw (fat pointer):
┌──────────────┬──────────────┐
│ data pointer │ vtable ptr   │
│ (8 bytes)    │ (8 bytes)    │
└──────────────┴──────────────┘

vtable for Circle:
┌──────────────────────────────┐
│ drop_in_place: fn(*mut ())   │
│ size: 8                      │
│ align: 8                     │
│ draw: fn(*const ()) → ()     │  ← Circle::draw
└──────────────────────────────┘
```

vtable에는 `drop` 함수, 타입의 크기와 정렬, 그리고 트레이트 메서드들의 함수 포인터가 포함된다. 메서드 호출 시 vtable을 역참조하여 실제 구현을 찾으므로, 간접 호출의 오버헤드(보통 수 나노초)가 발생한다.

## Object Safety

모든 트레이트가 트레이트 객체로 사용될 수 있는 것은 아니다. object-safe한 트레이트만 `dyn Trait`로 사용할 수 있다.

```rust
// Object-safe하지 않은 트레이트
trait NotObjectSafe {
    fn new() -> Self;                    // Self를 반환 — 크기를 알 수 없음
    fn compare<T>(&self, other: &T);     // 제네릭 메서드 — vtable에 넣을 수 없음
}

// Object-safe한 트레이트
trait ObjectSafe {
    fn process(&self);
    fn name(&self) -> &str;
}
```

핵심 조건은 (1) `Self: Sized` 바운드가 없어야 하고, (2) 메서드가 제네릭이 아니어야 하며, (3) `Self`를 반환 타입으로 사용하지 않아야 한다.

### where Self: Sized 트릭

트레이트의 일부 메서드만 object-safe하지 않은 경우, 해당 메서드에 `where Self: Sized` 바운드를 추가하면 트레이트 자체는 object-safe하게 만들 수 있다. 단, 트레이트 객체를 통해서는 그 메서드를 호출할 수 없게 된다.

## 실전 선택 기준

정적 디스패치와 동적 디스패치의 선택은 상황에 따라 다르다. 성능이 중요하고 타입이 컴파일 타임에 결정되면 제네릭을, 이종 컬렉션이 필요하거나 플러그인 아키텍처를 구현할 때는 `dyn Trait`를 사용한다.

Rust 표준 라이브러리에서도 `Iterator`는 주로 정적 디스패치로, `Error`와 `Read`/`Write`는 동적 디스패치(`Box<dyn Error>`)로 사용되는 패턴을 볼 수 있다.
