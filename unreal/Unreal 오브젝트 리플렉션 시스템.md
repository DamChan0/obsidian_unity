# 개념

![[Pasted image 20241005181044.png|600]]



## U 오브젝트
언리얼에서 사용하는 cpp 오브젝트를 구분하기 위해서 U를 붙혀서 사용한다

### 특징
![[Pasted image 20241005181145.png|600]]

## 필수 헤더 
```cpp
#include CoreMinimal.h
#include UObject/NoExportTypes.h
```
\
위의 두가지 헤더는 U오브젝트를 선언하기 위해서 필수로 필요한 부분

```cpp
#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "MyObject1.generated.h"

/**
 * 
 */
UCLASS()
class UNREALCPPSTRING_API UMyObject1 : public UObject
{
	GENERATED_BODY()
	
};

```

`UNREALCPPSTRING_API`  =  이 키워드를 통해서 다른 모듈에서도 `UMyObject1` 클래스를 참조 할 수 있다

GENERATED_BODY() =  fileName_genertated.h 에서 매크로로 구성되어있는 기능들을 사용하도록 해주는 기본함수 


## 언리얼 오브젝트의 구성

## UPROPERTY
관리되는 클래스 멤버 변수

## UFUNCTION
관리되는 클래스 멤버 함수

![[Pasted image 20241005200129.png|700]]


## CDO (class Default object)
UClass 를 사용해서 자신의 프로퍼티의 정보를 조회할 수 있다

- 언리얼 클래스에는 언리얼 객체의 기본 값을 저장하는 객체인CDO class default object 가 포함되어있다
- ![[Pasted image 20241005200310.png|700]]
- UClass는 CDO로부터 기본 값을 가져와서 초기화 한다
- CDO가 변경되면 엔진은 해당 클래스의 인스턴스 로드시 일괄 적용한다


![[Pasted image 20241012142708.png|800]]


---

# 예제
### person
### Student
### Teacher


위 3가지 클래스로 리플렉션 시스템을 구현해보도록한다

## person

```cpp
UPerson::UPerson()
{
	Name = TEXT("홍길동");
	Age = 20;
}

void UPerson::DoLesson()
{

	UE_LOG(LogTemp, Log, TEXT("%s님이 수업에 참여하였습니다."),Name);
}

const FString& UPerson::GetName() const
{
	// TODO: insert return statement here
	return Name;
}

void UPerson::SetName(const FString& NewName)
{
}

```

위와 같이 cpp 파일을 구성 한 후 다른 클래스들에서 Person을 상속 받도록 하는 것이 중요하다 그건은 아래에서 확인하자ㅣ

## students

```cpp

#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "Student.generated.h"
#include "Person.h"

/**
 * 
 */
UCLASS()
class MYPROJECT_STUDY_API UStudent : public UObject
{
	GENERATED_BODY()
	
};

```


위처럼 실행할 시 <span style="color:rgb(192, 0, 0)">에러가</span> 나게된다.
- 반드시 다른 클래스를 상속하기 위한 include 는 아래처럼  <span style="color:rgb(192, 0, 0)">#include "xxxxx.generated.h"</span> 전에 수행해야한다
```cpp
#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "Person.h"
#include "Student.generated.h"

```