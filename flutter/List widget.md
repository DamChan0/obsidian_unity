
# widget 리스트 형태 

column 이나 row위젯은 배열을 하위 위젯으로 받는다 children이라는 키워드로 받게 되는데 
이때 이 타입이 `List<widget>` 이다 이것을 함수로 만들어서 할당 할 수 있다


```dart
enum InputType {  
  rock,  
  paper,  
  scissors;  
  
  String get imagePath => '$assaetPath/$name.png';  
}

List<Widget> inputList() {
  return InputType.values
      .map((type) => Expanded(
            child: Container(
              child: Image.asset(type.imagePath),
            ),
          ))
      .toList();
}
```

이때 `**type**` 은 `InputType` 이라는 eunm을 가리키는 변수다


# map 함수

```dart

enum InputType {  
  rock,  
  paper,  
  scissors;  
  
  String get imagePath => '$assaetPath/$name.png';  
}


class UserChoice extends StatelessWidget {  
  final bool isDone;  
  
  List<Widget> inputList() {  
    debugPrint('inputList() 함수 호출됨'); // 함수 호출 로그  
  
    return InputType.values.map((choices) {  
      debugPrint('매핑 중: ${choices.name}'); // 각 매핑 단계 로그  
      return Expanded(  
        child: Container(  
          child: Image.asset(choices.imagePath),  
        ),  
      );  
    }).toList();  
  }  
  
  const UserChoice({required this.isDone, super.key});  
  
  @override  
  Widget build(BuildContext context) {  
    return Row(  
      children: inputList(),  
    );  
  }  
}
```
![[Pasted image 20241004143250.png]]

위 코드를 실행하면 map 함수는 총 세번 동작을 한다 즉 , enum의 총 갯수 만큼 동작하면서 choices 변수에 enum 데이터를 할당한다 