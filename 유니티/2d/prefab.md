> [!note]+ 
> 원본을 보호하면서 오브젝트를 복사하여 모든 오브젝트가 사라지는 것 막기위한 방법

## [[Instantiate]] : prefab을 이용한 오브젝트생성

```cs
 Instantiate(T original, Vector3 position, Quaternion rotation)
 // 객체, 위치, 회전 순
```

### 회전 방식

![[Pasted image 20240727192513.png|500]]

Ouaternion으로 하면 deg값의 각도로 표현하기에는 어려움 따라서 ouaternion을 euler로 변환시켜야한다



# 적 개체 prefab화 한 후 생성해보기

## 개체 생성하기
1. Hierachy에 기본이 되는 오브젝트 추가하기 
2. 필요한 component 추가하고 설정하기  
	- ![[Pasted image 20250127224459.png|300]]
	- sprite render : 객체의 형상(이미지)를 나타내기 위해서 추가
	- 접근 및 충돌 액션을 주로 사용하는 경우 추가
	- 물리 동작 및 위치등의 조작이 자주 필요한 경우 추가
3. 


