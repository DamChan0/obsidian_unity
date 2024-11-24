
characterController 를 이용한 방법과 키 입력에 따라서 vector의 변화를 체크하여 이동 시키는 방법이 있다.

- 단순한 이동의 경우 characterController를 이용하면 오버헤드가 발생할 수 도 있다
- 그러나 플레이어나 몹 등 충돌이나 상호 작용등이 있는 경우는 그냥 characterController 가 더 유용하다



# characterController

```cpp
    controller = GetComponent<CharacterController>();
    controller.Move(playerProperty.Direction * speed * Time.deltaTime);
```
오브젝트의 CharacterController 컴포넌트를 Get 한 후 Move 함수를 사용하여 이동 시킬 수 있다 

## 과정
방향을 계속 업데이트 해줘야한다
