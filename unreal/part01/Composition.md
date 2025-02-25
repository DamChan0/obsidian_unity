# Enum
---
언리얼 C++에는 열거형을 선언하면서 그 열거형에 대한 Meta Data를 추가해줄 수 있다 
```cpp
enum class  ECardType : uint8  
{  
    Student = 0 UMETA(DisplayName = "For Student"),  
    Teacher = 1 UMETA(DisplayName = "For Teacher"),  
    Staff = 2 UMETA(DisplayName = "For Staff"),  
    reserved = 3 UMETA(DisplayName = "Reserved"),  
};  
```
