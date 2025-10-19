## 인자로 받는 함수(람다)

```kotlin
inner class ViewHolder(val binding: ItemPhotoBinding) :  
    RecyclerView.ViewHolder(binding.root) {  
    init {  
        binding.root.setOnLongClickListener {  
            val clickPos = bindingAdapterPosition  
            if (clickPos == RecyclerView.NO_POSITION) {  
                return@setOnLongClickListener false  
            } else {  
                onItemLongClick(clickPos)  
                true  
            }  
        }  
    }  
}
```


### 명확하게 사용하는 형태 
```kotlin
binding.root.setOnLongClickListener { view ->
    val position = bindingAdapterPosition

    // ① 유효하지 않은 포지션일 때 → 즉시 false 반환
    if (position == RecyclerView.NO_POSITION) {
        return@setOnLongClickListener false
    }

    // ② 롱클릭 시 실행할 동작 호출
    onItemLongClick(position)

    // ③ true 반환 → 이벤트를 처리했음을 Android에 알림
    return@setOnLongClickListener true
}

```


-  view -> 의 의미
	- `view ->` 는 Kotlin 람다식 문법에서 **“매개변수 선언부(parameter list)”** 를 의미한다.  
	- 즉 `setOnLongClickListener` 함수가 **람다 인자(익명 함수)** 를 받는 구조이기 때문에 필요한 문법이다.``` kotlin 
		``` Kotlin
		 init {  
		        binding.root.setOnLongClickListener {  
		```
	- 에서 setOnLongClickListener의 매개변수를 정의한것이다 
	- `fun setOnLongClickListener(l: (View) -> Boolean)` 이 setOnLongClickListener의 원형 이기때문에 반환 값으로 true 또는 false를 반환하게 설계되었다.
	- 
