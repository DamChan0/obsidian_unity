
# 기본 틀 
```dart
class StateFull extends StatefulWidget {  
  const StateFull({super.key});  
  
  @override  
  State<StateFull> createState() => _StateFullState();  
}  
  
class _StateFullState extends State<StateFull> {  
  @override  
  Widget build(BuildContext context) {  
    return const Placeholder();  
  }  
}
```

## createState()
```dart
  State<StateFull> createState() => _StateFullState();  
```

이구문이 있는데 statefullWidget 클래스에서 필수적으로 해야하는 작업이다
- `State<StateFull>`은 이 메서드가 StateFull 위젯에 대한 State 객체를 반환한다는 의미를 가지고
	- 그것을 할당해주는 함수가 \_StateFullState(); 이며 화살표는 단일 표현식을 반환하는 문법


이떄 
```dart
abstract class StatefulWidget extends Widget 
abstract class State<T extends StatefulWidget> with Diagnosticable 
```

State\<T\>, StatefulWidget 은 추상 클래스이다

즉 각각 StateFull \_StateFullStat 에서 해당 클래스를 구현해주는 것이다 
