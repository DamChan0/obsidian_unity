> [!note]+ 
> 원본을 보호하면서 오브젝트를 복사하여 모든 오브젝트가 사라지는 것 막기위한 방법


> [!tip]+ 생성
> prefab을 이용한 오브젝트생성

## [[Spwaner_Instantiate]]

```cs
 Instantiate(T original, Vector3 position, Quaternion rotation)
 // 객체, 위치, 회전 순
```

# 회전 방식

![[Pasted image 20240727192513.png]]

Ouaternion으로 하면 deg값의 각도로 표현하기에는 어려움 따라서 ouaternion을 euler로 변환시켜야


