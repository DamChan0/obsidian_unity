

``` kotlin
    private lateinit var binding: ActivityMainBinding
```

- 변수 선언
- lateinit: 나중에 초기화 하겠다. 
- `ActivityMainBinding`: 안드로이드 빌드 시스템이 `activity_main.xml` 레이아웃 파일을 기반으로 자동으로 생성해준 클래스로 ViewBinding 이라고 한다 
	- `activity_main.xml`이라는 레이아웃 파일이 있으면,
	- 빌드시 `ActivityMainBinding`이라는 클래스가 `build/generated/data_binding_base_class_source_out/...` 경로에 자동 생성됨.
	- 생성 클래스 이름은 XML 파일 이름을 **UpperCamelCase + “Binding”**으로 변환한 것.
	- class 내용
	  ```kotlin
	    class ActivityMainBinding private constructor(
			val root: View,
			    val recyclerView: RecyclerView
			) {
			companion object {
				fun inflate(inflater: LayoutInflater): ActivityMainBinding { ... }
		i		fun bind(view: View): ActivityMainBinding { ... }
			}
		}
				```
- `inflate()`는 새로 뷰를 메모리에 로드할 때 사용한다.

