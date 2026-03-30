# Go 인터페이스의 내부 구현 — iface와 eface 구조체

Go의 인터페이스는 구조적 타이핑(structural typing)을 사용한다. 명시적 선언 없이, 메서드 집합만 일치하면 인터페이스를 만족한다. 이 유연한 설계가 런타임에서 어떻게 구현되는지 들여다본다.

## 두 종류의 인터페이스

Go 런타임에서 인터페이스는 두 가지 구조체로 표현된다. 메서드가 있는 인터페이스(`iface`)와 빈 인터페이스(`eface`, `interface{}` / `any`)다.

```go
// runtime/runtime2.go 에서 (간략화)
type iface struct {
    tab  *itab          // 인터페이스-타입 쌍의 메타데이터
    data unsafe.Pointer // 실제 데이터 포인터
}

type eface struct {
    _type *_type         // 타입 메타데이터
    data  unsafe.Pointer // 실제 데이터 포인터
}
```

두 구조체 모두 16바이트(64비트 시스템)다. 인터페이스 값은 항상 이 크기의 fat pointer로 표현된다.

### itab 구조체

`itab`은 인터페이스 타입과 구체적 타입의 조합을 나타낸다. 인터페이스의 메서드 목록과 구체적 타입의 메서드를 매핑하는 함수 포인터 테이블을 포함한다.

```go
type itab struct {
    inter *interfacetype // 인터페이스 타입 정보
    _type *_type         // 구체적 타입 정보
    hash  uint32         // _type.hash의 복사본 (빠른 타입 스위치)
    _     [4]byte
    fun   [1]uintptr     // 가변 길이 — 메서드 함수 포인터 배열
}
```

`fun` 배열은 인터페이스의 메서드 수만큼 확장된다. Go 런타임은 itab을 전역 해시 테이블에 캐싱하여, 동일한 인터페이스-타입 쌍에 대해 반복 계산을 피한다.

## 인터페이스 할당과 이스케이프

값이 인터페이스에 할당될 때 힙 할당이 발생할 수 있다. 작은 스칼라 값은 `data` 포인터에 직접 저장되기도 하지만, 구조체 등은 힙에 복사된다.

```go
type Writer interface {
    Write([]byte) (int, error)
}

type MyWriter struct {
    buf []byte
}

func (w *MyWriter) Write(p []byte) (int, error) {
    w.buf = append(w.buf, p...)
    return len(p), nil
}

func main() {
    mw := &MyWriter{}
    var w Writer = mw  // iface{tab: *itab, data: mw} 생성
    w.Write([]byte("hello"))
}
```

포인터 리시버 메서드를 가진 타입은 포인터만 인터페이스에 할당할 수 있다. 값 리시버 메서드는 값과 포인터 모두 가능하다.

### 이스케이프 분석

컴파일러의 이스케이프 분석이 인터페이스 할당의 힙 할당 여부를 결정한다. `-gcflags="-m"` 플래그로 확인할 수 있다.

```bash
go build -gcflags="-m" main.go
# ./main.go:15:6: mw escapes to heap
# ./main.go:16:17: mw does not escape (인라인된 경우)
```

## 타입 어서션과 타입 스위치

타입 어서션(`v.(Type)`)은 `itab`의 타입 정보를 검사하여 수행된다. 타입 스위치는 `itab.hash`를 사용한 빠른 비교로 구현된다.

```go
func process(v interface{}) {
    switch val := v.(type) {
    case int:
        fmt.Printf("int: %d\n", val)
    case string:
        fmt.Printf("string: %s\n", val)
    case io.Reader:
        buf := make([]byte, 1024)
        val.Read(buf)
    default:
        fmt.Printf("unknown: %T\n", val)
    }
}
```

인터페이스에서 구체적 타입으로의 어서션은 해시 비교 한 번으로 O(1)에 수행된다. 인터페이스에서 다른 인터페이스로의 어서션은 메서드 집합 포함 관계를 확인해야 하므로 더 비용이 크다.

### nil 인터페이스의 함정

Go에서 가장 흔한 실수 중 하나는 nil 포인터를 담은 인터페이스가 nil이 아니라는 점이다.

```go
func returnsError() error {
    var p *MyError = nil
    return p  // error 인터페이스에 nil 포인터를 담아 반환
}

func main() {
    err := returnsError()
    fmt.Println(err == nil)  // false! iface{tab: *itab(MyError), data: nil}
}
```

인터페이스가 nil이려면 `tab`과 `data` 모두 nil이어야 한다. nil 포인터를 담으면 `tab`이 설정되어 nil 검사를 통과하지 못한다.

## 성능 고려사항

인터페이스를 통한 메서드 호출은 직접 호출보다 약간 느리다. vtable을 통한 간접 호출이 필요하고, 인라인이 불가능하기 때문이다. 하지만 대부분의 경우 이 오버헤드는 무시할 수 있는 수준이다.

성능이 극도로 중요한 핫 루프에서는 제네릭(Go 1.18+)을 사용하여 정적 디스패치를 활용하는 것이 좋다. 그 외에는 인터페이스의 유연성이 미미한 성능 비용을 충분히 상쇄한다.
