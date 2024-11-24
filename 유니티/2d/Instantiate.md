
# Instantiate를 이용한 다양한 spwan 방법

## spwanPoint

```cpp
public Transform[] spwanPoint;

Vector3 pos = spwanPoint[prefabIndex].position;
```

- Transform : 게임오브젝트의 Transform을 가져올수 있음
	- spwan 위치를 spawner가 아닌 다른 오브젝트를 기준으로 설정 할 수 있음

- GameObject obj = GetComponent\<Script_Name>().{function_Name}(변수);
	- 생성된 obj에 포함된 Script안에 있는 함수또는 변수에한하여 초기 값을 설정 할 수 있음



## 시간 지연 spwan

### delaTime 사용
```cs
//float 타입의 시간을 저장할 변수를 선언하고 

float time += Time.deltaTime

if( time >= 0.5f){
.....

}
```


### InvokeRepeating("methodName", time1, time2);
```cs
InvokeRepeating("SpwanOnPoint", 0.4f, 0.3f);
```
- spwanOnPoint 는 0.4초 뒤에 호출되고 0.3초 마다 계속 반복한다.
- 반복 멈춤
	- [[Instantiate#delaTime 사용]]을 사용하여 시간 스택을 쌓거나 WaitForSeconds(stopAfter);으로 특정 시간 후 멈추도록 할 수 있음
	- CancelInvoke("SpawnOnPoint"); // 반복 호출 멈춤



## Random 생성

### Random.Range(float a, float b)
```cs
float x = Random.Range(-5.0f, 5.0f);
float y = Random.Range(-5, 5);
Vector3 pos = new Vector3(x, y, 0);
Instantiate(prefab[index], pos, Quaternion.identity);
```

a와 b사이에서 랜덤 값을 생성한다. 
x,y 좌표를 각각 랜던하게 생성하고 Vector3변수에 대입하여 랜덤 생성 할 수 있음


## 총알 생성

- 플레이어가 발사하는 오브젝트도 만들 수 있다

```cs
if (Input.GetKeyDown(keycode.Space))
{
	GameObject bulletobj = Instantiate(bullet, transform.position, quaternion.identity);
}
```

- Space를 누를 때마다 bullet GameObject를 생성한다 
	- 이때 bullet은 prefab으로 하는것이 바람직하다