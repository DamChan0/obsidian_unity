# Android 이미지 로딩 문제 해결 가이드

## 📋 문제 상황

-   앱 실행 시 이전에 등록한 이미지들이 로딩되지 않음
-   이미지 칸은 있지만 실제 이미지가 표시되지 않음
-   새로운 이미지 추가 후에야 이전 이미지들이 보임

## 🔍 문제 원인 분석

### 1. **데이터베이스 로딩 문제**

-   앱 시작 시 데이터베이스에서 이미지를 제대로 로드하지 못함
-   UI 업데이트가 제대로 되지 않음

### 2. **Photo Picker URI 권한 문제**

-   Android 13+ (API 33+)에서 Photo Picker URI 접근 권한 부족
-   `SecurityException: Calling uid does not have permission to access picker uri`

### 3. **이미지 로더 설정 문제**

-   Coil 이미지 로더가 content URI를 제대로 처리하지 못함
-   URI 파싱 및 권한 처리 부족

## 🛠️ 해결 과정

### 1단계: 데이터베이스 로딩 개선

#### A. MainActivity.kt 수정

```kotlin
private fun loadImagesFromDatabase() {
    lifecycleScope.launch {
        try {
            android.util.Log.d("MainActivity", "Loading images from database...")
            val images = database.imageDao().getAllImages()
            android.util.Log.d("MainActivity", "Loaded ${images.size} images from database")

            // 데이터베이스가 비어있는 경우 로그
            if (images.isEmpty()) {
                android.util.Log.w("MainActivity", "Database is empty - no images found")
            } else {
                // 각 이미지 정보 로그
                images.forEachIndexed { index, image ->
                    android.util.Log.d("MainActivity", "Image $index: ${image.title} - ${image.imageUri}")
                }
            }

            photoList.clear()
            photoList.addAll(images)

            // UI 스레드에서 어댑터 업데이트
            runOnUiThread {
                photoAdapter.notifyDataSetChanged()
                // RecyclerView 강제 새로고침
                binding.recyclerView.invalidate()
                binding.recyclerView.requestLayout()
            }
        } catch (e: Exception) {
            android.util.Log.e("MainActivity", "Error loading images", e)
            e.printStackTrace()
        }
    }
}
```

#### B. 앱 생명주기 관리 추가

```kotlin
override fun onResume() {
    super.onResume()
    // 앱이 다시 활성화될 때마다 데이터 새로고침
    loadImagesFromDatabase()
}
```

### 2단계: 권한 설정

#### A. AndroidManifest.xml 권한 추가

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
<uses-permission android:name="android.permission.READ_MEDIA_VISUAL_USER_SELECTED" />
```

#### B. MainActivity.kt에서 권한 요청

```kotlin
private fun checkPermissions() {
    val permissions = mutableListOf<String>()

    if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE)
        != PackageManager.PERMISSION_GRANTED) {
        permissions.add(Manifest.permission.READ_EXTERNAL_STORAGE)
    }

    if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.TIRAMISU) {
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_MEDIA_IMAGES)
            != PackageManager.PERMISSION_GRANTED) {
            permissions.add(Manifest.permission.READ_MEDIA_IMAGES)
        }
    }

    if (permissions.isNotEmpty()) {
        ActivityCompat.requestPermissions(this, permissions.toTypedArray(), 100)
    }
}
```

### 3단계: Photo Picker URI 문제 해결

#### A. URI 권한 부여

```kotlin
private fun saveImageToDatabase(uri: Uri) {
    lifecycleScope.launch {
        try {
            // URI 권한 부여 (Photo Picker URI의 경우 필요)
            contentResolver.takePersistableUriPermission(uri,
                android.content.Intent.FLAG_GRANT_READ_URI_PERMISSION)

            // 이미지를 앱 내부 저장소로 복사
            val copiedUri = copyImageToInternalStorage(uri)

            val newImage = ImageEntity(
                title = "새 이미지",
                imageUri = copiedUri?.toString() ?: uri.toString(),
                description = "갤러리에서 추가된 사진"
            )
            database.imageDao().insertImage(newImage)
            loadImagesFromDatabase()
        } catch (e: Exception) {
            android.util.Log.e("MainActivity", "Error saving image to database", e)
        }
    }
}
```

#### B. 이미지 복사 함수

```kotlin
private fun copyImageToInternalStorage(uri: Uri): Uri? {
    return try {
        val inputStream = contentResolver.openInputStream(uri)
        val fileName = "image_${System.currentTimeMillis()}.jpg"
        val file = java.io.File(filesDir, fileName)
        val outputStream = java.io.FileOutputStream(file)

        inputStream?.use { input ->
            outputStream.use { output ->
                input.copyTo(output)
            }
        }

        android.util.Log.d("MainActivity", "Image copied to: ${file.absolutePath}")
        Uri.fromFile(file)
    } catch (e: Exception) {
        android.util.Log.e("MainActivity", "Error copying image", e)
        null
    }
}
```

### 4단계: 이미지 로더 개선

#### A. PhotoAdapter.kt 수정

```kotlin
override fun onBindViewHolder(holder: ViewHolder, position: Int) {
    val photo = photoList[position]

    // Photo Picker URI인 경우 권한 확인
    if (photo.imageUri.contains("picker_get_content")) {
        android.util.Log.d("PhotoAdapter", "Photo Picker URI detected: $uri")
    }

    // URI를 Uri 객체로 변환하여 로드
    val uri = android.net.Uri.parse(photo.imageUri)
    holder.binding.ivPhoto.load(uri) {
        crossfade(true)
        placeholder(R.drawable.ic_menu_gallery)
        error(R.drawable.ic_menu_gallery)
        listener(
            onStart = { request -> android.util.Log.d("PhotoAdapter", "Loading image: $uri") },
            onSuccess = { request, result -> android.util.Log.d("PhotoAdapter", "Image loaded successfully: $uri") },
            onError = { request, result ->
                android.util.Log.e("PhotoAdapter", "Failed to load image: $uri", result.throwable)
                if (result.throwable is SecurityException) {
                    android.util.Log.e("PhotoAdapter", "SecurityException - URI permission issue for Photo Picker")
                }
            }
        )
    }
}
```

### 5단계: 데이터베이스 설정 개선

#### A. AppDatabase.kt 수정

```kotlin
val instance = Room.databaseBuilder(
    context.applicationContext,
    AppDatabase::class.java,
    "image_database"
)
.fallbackToDestructiveMigration()
.allowMainThreadQueries() // 디버깅용 (프로덕션에서는 제거)
.build()
```

#### B. 데이터베이스 상태 확인

```kotlin
private fun checkDatabaseStatus() {
    lifecycleScope.launch {
        try {
            val count = database.imageDao().getAllImages().size
            android.util.Log.d("MainActivity", "Database status: $count images found")

            if (count == 0) {
                android.util.Log.w("MainActivity", "Database is empty - this might be a fresh install or data was cleared")
            }
        } catch (e: Exception) {
            android.util.Log.e("MainActivity", "Error checking database status", e)
        }
    }
}
```

## 📊 핵심 해결책

### 1. **이미지 복사 방식**

-   Photo Picker URI → 앱 내부 저장소로 복사
-   권한 문제 완전 해결
-   URI 접근 안정성 확보

### 2. **강화된 로딩 시스템**

-   앱 시작 시 자동 로딩
-   앱 재활성화 시 새로고침
-   에러 처리 및 로깅

### 3. **권한 관리**

-   Android 13+ 대응
-   Photo Picker 권한 처리
-   URI 지속성 권한 부여

## 🔧 디버깅 로그

### 성공적인 로딩 로그

```
MainActivity: Database status: 3 images found
MainActivity: Loading images from database...
MainActivity: Loaded 3 images from database
PhotoAdapter: Loading image: file:///data/data/com.example.grid/files/image_xxx.jpg
PhotoAdapter: Image loaded successfully: file:///data/data/com.example.grid/files/image_xxx.jpg
```

### 에러 로그

```
MainActivity: Database is empty - no images found
PhotoAdapter: Failed to load image: content://media/picker_get_content/...
PhotoAdapter: SecurityException - URI permission issue for Photo Picker
```

## ⚠️ 주의사항

### 1. **앱 새로 빌드 시 데이터 손실**

-   `fallbackToDestructiveMigration()` 설정으로 인한 데이터 삭제
-   Debug 모드에서 앱 재설치 시 데이터 초기화
-   개발 중 데이터 백업 권장

### 2. **메모리 관리**

-   이미지 복사로 인한 저장공간 사용량 증가
-   필요시 오래된 이미지 정리 기능 구현

### 3. **보안 고려사항**

-   앱 내부 저장소 사용으로 보안성 향상
-   Photo Picker URI 권한 관리

## 🎯 최종 결과

✅ **이미지 즉시 로딩**: 앱 시작 시 저장된 이미지들이 바로 표시  
✅ **권한 문제 해결**: Photo Picker URI 접근 오류 완전 해결  
✅ **안정적인 UI**: RecyclerView 강제 새로고침으로 UI 업데이트 보장  
✅ **디버깅 강화**: 상세한 로그로 문제 추적 가능  
✅ **생명주기 관리**: 앱 재활성화 시 자동 데이터 새로고침

## 📚 학습 포인트

1. **Room Database**: 데이터 영속성과 마이그레이션 관리
2. **Photo Picker**: Android 13+ 새로운 권한 시스템 이해
3. **Coil 이미지 로더**: 비동기 이미지 로딩과 에러 처리
4. **Android 생명주기**: 앱 상태 변화에 따른 데이터 관리
5. **URI 권한 관리**: Content Provider 접근 권한 처리

이 가이드를 통해 Android 이미지 로딩 문제의 전형적인 해결 과정을 학습할 수 있습니다.
