# 델리케이트 
---
c언어의 함수 포인터와 같은 기능을 하지만 완전한 안전성을 보장한다. 


# 발행 구독 디자인 패턴
___
제작자는 발행자를 생성한다

제작자 |  발행자    (발행) ===>   구독자
	        	(구독) <=== 	
	        
- 이때 발행자가 델리케이터가 된다
![[Drawing 2024-11-17 15.21.02.excalidraw|700]]



# 선언시 고려사항
---
- **어떤 데이터를 전달하고 받을 것인가?** 인자의 수와 각각의 타입을 설계
    
    - 몇 개의 인자를 전달할 것인가?
    - 어떤 방식으로 전달할 것인가?
        - 일대일로 전달
        - 일대다로 전달
- **프로그래밍 환경 설정**
    
    - C++ 프로그래밍에서만 사용할 것인가?
    - UFUNCTION으로 지정된 블루프린트 함수와 사용할 것인가?
- **어떤 함수와 연결할 것인가?**
    
    - 클래스 외부에 설계된 C++ 함수와 연결
    - 전역에 설계된 정적 함수와 연결
    - 언리얼 오브젝트의 멤버 함수와 연결 (대부분의 경우에 이 방식을 사용)


# 선언형태 : 유형  DECALRE_{유형}DELEGATE{함수정보}
## 유형 
---
### 델리게이트 유형: 어떤 유형의 델리게이트인지 구상한다

- **일대일 형태로 C++만 지원한다면** 유형을 공란으로 둔다.  
    `DECLARE_DELEGATE`
    
- **일대다 형태로 C++만 지원한다면** MULTICAST를 선언한다.  
    `DECLARE_MULTICAST`
    
- **일대일 형태로 블루프린트를 지원한다면** DYNAMIC을 선언한다.  
    `DECLARE_DYNAMIC`
    
- **일대다 형태로 블루프린트를 지원한다면** DYNAMIC과 MULTICAST를 조합한다.  
    `DECLARE_DYNAMIC_MULTICAST`

### 함수 정보: 연동 될 함수 형태를 지정한다

- **인자가 없고 반환값도 없으면** 공란으로 둔다.  
    예) `DECLARE_DELEGATE`
    
- **인자가 하나고 반환값이 없으면** OneParam으로 지정한다.  
    예) `DECLARE_DELEGATE_OneParam`
    
- **인자가 세 개고 반환값이 있으면** RetVal_ThreeParams로 지정한다.  
    예) `DECLARE_DELEGATE_RetVal_ThreeParams` (MULTICAST는 반환값을 지원하지 않음)
    
- 파라메터는 **최대 9개까지 지원함**


## 예시 
학생과 학사정보에 대한 예시를 만들어본다
- 학생은 학사 정보의 변경을 알림으로 받을 수 있다
	- 매개변수 : 알림의 주체(1), 알림 내용(2), = 총 2개
	  
- 변경 내용은 모든 학생에대 발송
	- MULTICAST
	  
- C++ 에서만 사용  => DYNAMIC 사용 X

### 학사 정보 클래스는 학생 클래스의 상호 의존성을 없앤다.

- 학사정보 class는 델리게이트를 선언하고 알림을 보내는 역할만한다
- 학생은 알림을 받기로한다
- ![[Unreal Delegate 2024-11-17 15.30.33.excalidraw|800]]

# 예제
```CPP
DECLARE_MULTICAST_DELEGATE_TwoParams(FCourseInfoOnChangedSignature, const FString&, const FString&);
```
위의 방법으로 델리게이트 함수를 선언하고 `FCourseInfoOnChangedSignature` 타입으로 선언된 함수를 사용하여 콜백이될 객체와 해당 객체의 멤버변수를 연동될 함수로 바인딩한다

```CPP
UCLASS()
class UNREALDELEGATE_API UCourseInfo : public UObject
{
	GENERATED_BODY()

public:
	UCourseInfo();
	FCourseInfoOnChangedSignature OnChanged;

	void ChangeCourseInfo(const FString& InSchoolName, const FString& NewContents);

private:
	FString Content;
};
```

```cpp
void UMyGameInstance::Init()
{
	Super::Init();
	CourseInfo = NewObject<UCourseInfo>();

	UStudent* Student1 = NewObject<UStudent>();
	UStudent* Student2 = NewObject<UStudent>();
	UStudent* Student3 = NewObject<UStudent>();

	Student1->SetName(TEXT("학생1"));
	Student2->SetName(TEXT("학생2"));
	Student3->SetName(TEXT("학생3"));


	CourseInfo->OnChanged.AddUObject(Student1, &UStudent::GetCourseNoti);
	CourseInfo->OnChanged.AddUObject(Student2, &UStudent::GetCourseNoti);
	CourseInfo->OnChanged.AddUObject(Student3, &UStudent::GetCourseNoti);

	CourseInfo->ChangeCourseInfo(SchoolName, TEXT("2024 2학기 학사"));
	
}

```