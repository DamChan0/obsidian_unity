# Room Database - 데이터 다루기

## 1. 기본 구조

### Entity (데이터 모델)

```kotlin
@Entity(tableName = "images")
data class ImageEntity(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    val title: String,
    val imageUri: String,
    val description: String? = null,
    val timestamp: Long = System.currentTimeMillis()
)
```

- `@Entity`: Room 데이터베이스 테이블로 사용
- `tableName = "images"`: 실제 DB 테이블 이름
- `@PrimaryKey(autoGenerate = true)`: 자동 증가 기본키

### DAO (Data Access Object)

```kotlin
@Dao
interface ImageDao {
    // 조회
    @Query("SELECT * FROM images ORDER BY id DESC")
    suspend fun getAllImages(): List<ImageEntity>
    
    // 삽입
    @Insert
    suspend fun insertImage(image: ImageEntity)
    
    // ID 리스트로 삭제
    @Query("DELETE FROM images WHERE id IN (:ids)")
    suspend fun deleteByIds(ids: List<Int>)
    
    // 엔티티로 삭제
    @Delete
    suspend fun deleteImages(images: List<ImageEntity>)
}
```

## 2. 핵심 개념

### @Query 작동 원리

```kotlin
@Query("DELETE FROM images WHERE id IN (:ids)")
suspend fun deleteByIds(ids: List<Int>)
```

1. **개발자 작성**: SQL + 함수 선언 (구현 없음)
2. **빌드 시**: Room이 `ImageDao_Impl` 클래스 자동 생성
3. **실행 시**: 생성된 구현체가 실제로 동작

### 테이블 이름 참조

- SQL 쿼리의 `images`는 `@Entity(tableName = "images")`에서 정의한 테이블 이름
- Package 이름과는 무관
- Room이 컴파일 시점에 Entity 타입과 테이블 이름을 매핑

### suspend 함수

```kotlin
suspend fun deleteByIds(ids: List<Int>)
```

- **코루틴에서만 호출 가능**한 일시 중단 함수
- DB 작업처럼 시간이 걸리는 작업에 사용
- 메인 스레드 블로킹 방지

**사용 예시:**

```kotlin
// ViewModel에서 사용
viewModelScope.launch {
    imageDao.deleteByIds(listOf(1, 2, 3))
}
```

### 같은 Package = Import 불필요

```kotlin
// ImageEntity.kt
package com.example.grid.data.local

// ImageDao.kt  
package com.example.grid.data.local  // ← 같은 패키지

interface ImageDao {
    suspend fun getAllImages(): List<ImageEntity>  
    // ↑ import 없이 ImageEntity 사용 가능
}
```

## 3. 전체 흐름

```
[UI] → [ViewModel] → [Repository] → [DAO] → [Database]

1. 사용자 액션 (버튼 클릭)
   ↓
2. ViewModel에서 코루틴 시작
   viewModelScope.launch { ... }
   ↓
3. DAO의 suspend 함수 호출
   imageDao.deleteByIds(ids)
   ↓
4. Room이 자동 생성한 구현체 실행
   ↓
5. SQLite 데이터베이스에서 실제 삭제
```

## 4. 주요 어노테이션

| 어노테이션 | 용도 | 예시 |
|-----------|------|------|
| `@Entity` | 테이블 정의 | `@Entity(tableName = "images")` |
| `@PrimaryKey` | 기본키 지정 | `@PrimaryKey(autoGenerate = true)` |
| `@Dao` | DAO 인터페이스 표시 | `@Dao interface ImageDao` |
| `@Query` | SQL 쿼리 실행 | `@Query("SELECT * FROM images")` |
| `@Insert` | 삽입 작업 | `@Insert suspend fun insert(...)` |
| `@Delete` | 삭제 작업 | `@Delete suspend fun delete(...)` |
| `@Update` | 업데이트 작업 | `@Update suspend fun update(...)` |

## 5. 코드 생성 위치

Room이 자동 생성한 구현체는 다음 위치에서 확인 가능:

```
app/build/generated/ksp/debug/kotlin/com/example/grid/data/local/ImageDao_Impl.kt
```

## 6. 실제 프로젝트 예시 (Grid 앱)

### ImageEntity.kt

```kotlin
package com.example.grid.data.local

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "images")
data class ImageEntity(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    val title: String,
    val imageUri: String,
    val description: String? = null,
    val timestamp: Long = System.currentTimeMillis()
)
```

### ImageDao.kt

```kotlin
package com.example.grid.data.local

import androidx.room.Dao
import androidx.room.Delete
import androidx.room.Insert
import androidx.room.Query

@Dao
interface ImageDao {
    @Query("SELECT * FROM images ORDER BY id DESC")
    suspend fun getAllImages(): List<ImageEntity>

    @Insert
    suspend fun insertImage(image: ImageEntity)

    @Query("DELETE FROM images WHERE id IN (:ids)")
    suspend fun deleteByIds(ids: List<Int>)

    @Delete
    suspend fun deleteImages(images: List<ImageEntity>)
}
```

---

#android #kotlin #room #database #jetpack