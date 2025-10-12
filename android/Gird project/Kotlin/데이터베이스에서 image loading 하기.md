
## 데이터베이스에서 이미지 로드
[[Android 이미지 로딩 문제 해결 가이드#Android 이미지 로딩 문제 해결 가이드#1단계 데이터베이스 로딩 개선]]

- 데이터베이스 : app 내부 데이터베이스
### 함수 개요 
- **비동기 처리:** `lifecycleScope.launch { ... }`
    
- **데이터 취득:** `database.imageDao().getAllImages()`
    
- **UI 갱신:** `photoAdapter.notifyDataSetChanged()`, `invalidate()`, `requestLayout()`  모두 recyclerView안에 구현되어있는 함수들 
	- **`notifyDataSetChanged()`**: 어댑터에게 데이터가 바뀌었음을 알림.
	- **`invalidate()`**: RecyclerView 다시 그리기 요청.
	- **`requestLayout()`**: RecyclerView 레이아웃 강제 재계산.


### [[코루틴]] 