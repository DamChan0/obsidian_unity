# 인터페이스란
---
객체가 반드시 구현해야할 행동을 지정하는 모던 객체지형 설계의 기본 

## 언리얼의 인터페이스 

### Actor : 
월드에 배치가 가능한 모든 오브젝트 

### Pawn :
Actor를 상속 받은 움직임이 필요한 객체 
- ex :INavaAgentInterFace를 구현하여 움직임을 구현하도록 한다

![[Pasted image 20241102154514.png|500]]

## 특징

- 언리얼에서 인터페이스를 생성하면 U와 I 로 시작하는 클래스가 생긴다
	- I 는 실제 구현부에 해당하는 클래스
	- U 는 클래스 타입 정보를 제공하는 타입 전용 클래스 
- C++ 인터페이스는 클래스를 사용해서 인터페이스를 구현하기 떄문에 기본 클래스에도 구현이 가능


## 생성
생성은 언리얼 에디터에서 interface class를 선택하여 생성 할 수 있음
![[Pasted image 20241109201347.png|500]]
### 생성된 결과물
```cpp
class ULessenInterface : public UInterface  
{  
    GENERATED_BODY()  
};
```
해당 내용은 정보를 담기위한 내용으로 구현을 따로 할 필요는 없다

```cpp
class UNREALINTERFACE_API ILessenInterface  
{  
    GENERATED_BODY()  
  
    // Add interface functions to this class. This is the class that will be inherited to implement this interface.  
public:  
};
```

I로 시작하는 위 클래스에 구현내용을 작성하면 된다 

public:

안에 virtual로 함수를 선언하게 되면 해당 클래스를 상속하는 모든 자식 클래스는 함수를 구현해주어야 컴파일이된다

- 하지만 아래처럼 기초 구현을 추가한다면 반드시 구현할 필요는 없다
```cpp
class UNREALINTERFACE_API ILessenInterface  
{  
    GENERATED_BODY()  
  
    // Add interface functions to this class. This is the class that will be inherited to implement this interface.  
public:  
    virtual void DoLessen()  
    {       UE_LOG(LogTemp, Warning, TEXT("수업을 듣습니다.")); // 인터페이스의 기본 구현  
    }// 자식 클래스들은 이 함수를 반드시 구현해야한다.  
};

```

## 상속여부의 체크

Cast 함수를 사용하여 각 객체들이 상속 여부를 체크할 클래스로 케스팅이 되었는지 여부를 판단하여 확인할 수 있다 
```cpp
for(const auto person : Persons)  
{  
    ILessenInterface* LessenInterface = Cast<ILessenInterface>(person);  
    if(!LessenInterface)  
    {        UE_LOG(LogTemp, Warning, TEXT("%s 는 수업을 들을 수 없습니다,"), *person->GetName());  
    }    else  
    {  
        UE_LOG(LogTemp, Warning, TEXT("%s 는 수업을 들을 수 있습니다."), *person->GetName());  
        LessenInterface->DoLessen();  
    }}

```
