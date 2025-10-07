## `Default` trait 
- `std::default::Default` trait은 `default()` 메서드를 제공해서 “기본값”을 만들 수 있게 함.
- 표준 라이브러리나 제네릭 코드에서 `T: Default` 제약이 걸린 경우, `T::default()`로 인스턴스를 만들 수 있음.

```rust 
struct UiState {
    color: String,
    count: u32,
    level: String,
}

impl Default for UiState {
    fn default() -> Self {
        Self {
            color: "#22c55e".into(),
            count: 0,
            level: "mid".into(),
        }
    }
}
```

### **사용 시나리오** 
- `UiState::default()`로 바로 생성.
    
- `vec![T::default(); n]` 같은 제네릭 컨테이너 초기화.
    
- `#[derive(Default)]`로 자동 구현 가능.
    
- C++에서 “기본 생성자” 호출과 비슷하지만, Rust는 생성자라는 개념이 없고, 대신 trait 기반으로 구현함.

### **반환 값**
- 인스턴스가 지정한 초기값으로 생성됨.



# New 
**역할**

- Rust에서 생성자라는 키워드가 없기 때문에 관례상 `new()`라는 연관 함수(associated function)를 정의해 인스턴스를 만드는 경우가 많음.
    
- **`Default`는 trait이지만, `new`는 그냥 네이밍 규칙**이자 편의 함수임.
    

**형식**

```rust
impl UiState {
    pub fn new(color: &str, count: u32, level: &str) -> Self {
        Self {
            color: color.into(),
            count,
            level: level.into(),
        }
    }
}

```

### **사용 시나리오**

- `UiState::new("#22c55e", 0, "mid")`처럼 파라미터를 직접 지정할 때.
    
- 파라미터 유효성 검증, 복잡한 초기화 절차를 넣을 때.
    

### **반환 값**

- 호출 시 전달받은 값들로 인스턴스를 생성함.



---
즉 Rust에서는

- **`Default` → 기본 생성 규격화 (trait 기반)**
    
- **`new` → 관례상 쓰는 생성자 역할 함수 (자유도 높음)**
    

보통 단순 기본값이 필요하면 `Default`, 파라미터나 검증 로직이 필요하면 `new`를 같이 둔다.