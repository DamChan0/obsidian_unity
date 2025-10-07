# Rust Lifetime (수명) 정리

## 1. Lifetime이란?
- **변수나 참조가 유효한 범위(scope)를 컴파일러가 추적하는 개념**
- Rust는 메모리를 자동 관리하지만, **참조(reference)**가 잘못된 메모리를 가리키지 않도록
  **"수명 검사기(borrow checker)"**가 lifetime을 체크함.
- 런타임이 아니라 **컴파일 타임**에만 존재하는 개념.

---

## 2. Lifetime의 필요성
- Dangling reference(죽은 참조) 방지
- 참조가 소유자보다 오래 살지 않게 보장
- 예:
```rust
fn main() {
    let r;
    {
        let x = 5;
        r = &x; // ❌ x는 여기서 소멸, r은 유효하지 않음
    }
    println!("{}", r);
}

```

에러 발생
`error[E0597]: `x` does not live long enough`


## 3. Lifetime 표기법

- `'a`, `'b`처럼 **작은따옴표 + 이름**으로 표기
    
- 함수, 구조체, impl 등에 붙일 수 있음
    
- 예:
```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}

```
여기서 `'a`는 **x와 y, 그리고 반환값이 모두 같은 수명을 가진다**는 의미.


## 4. Lifetime 추론(Elision)

- 대부분의 경우 수명 표기를 생략할 수 있음
    
- Rust 컴파일러는 3가지 기본 규칙으로 lifetime을 추론:
    
    1. 참조 매개변수가 1개면, 반환값의 lifetime은 그 매개변수와 동일
        
    2. 참조 매개변수가 여러 개면, 반환값 lifetime은 명시적으로 지정 필요
        
    3. `&self`나 `&mut self`를 받는 메서드는 반환값 lifetime이 자동으로 `self`와 동일
- 예:
```rust
fn first(s: &str) -> &str { s } // lifetime 명시 불필요p
```


## 5. 구조체와 Lifetime

- 참조를 필드로 가지는 구조체는 lifetime을 명시해야 함
```rust
struct Book<'a> {
    title: &'a str,
}
```


## 6. `'static` Lifetime

- 프로그램 전체 실행 기간 동안 살아있는 데이터
    
- 문자열 리터럴이 대표적인 예
```rust
let s: &'static str = "Hello, world!";
```

## 7. 요약

- Lifetime은 참조의 유효 범위를 컴파일러가 추적하기 위한 **타입 시스템의 일부**
    
- 런타임에 영향 없음 (메모리 주소 추적용)
    
- 기본 규칙 덕분에 대부분 생략 가능
    
- 필요한 경우 `'a`처럼 명시적으로 지정
    
- `'static`은 프로그램 전체 기간 유효