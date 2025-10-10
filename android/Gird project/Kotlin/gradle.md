# 주요 계층

프로젝트 루트/
│
├── settings.gradle.kts        👈 최상위 설정 (프로젝트 구성, 저장소 설정 등)
├── build.gradle.kts           👈 루트 빌드 설정 (공통 설정, 플러그인, 의존성 관리)
│
└── app/                       👈 모듈(예: 앱)
    └── build.gradle.kts       👈 모듈별 설정 (Android 설정, 의존성, 빌드 설정 등)





## 1️⃣ **`settings.gradle.kts`**

📍 **프로젝트의 전역 구조와 저장소 설정을 담당**

### 주요 역할

- 어떤 모듈들을 포함할지 정의 (`include(":app")`)
    
- 저장소(repository) 설정 (예: `google()`, `mavenCentral()`, `maven("https://jitpack.io")`)
    
- Gradle 버전 관리 및 프로젝트 이름 설정

- 예시
	```
	pluginManagement {
	    repositories {
	        gradlePluginPortal()
	        google()
	        mavenCentral()
	    }
	}
	
	dependencyResolutionManagement {
	    repositoriesMode.set(RepositoriesMode.PREFER_SETTINGS) // ⚠️ 중요
	    repositories {
	        google()
	        mavenCentral()
	        maven(url = "https://www.jitpack.io") // ✅ 외부 라이브러리 저장소
	    }
	}
	
	rootProject.name = "Grid"
	include(":app")
	
	```

## 2️⃣ **루트 `build.gradle.kts`**

📍 **전체 프로젝트에 공통으로 적용되는 설정**

### 주요 역할

- 모든 모듈에 적용되는 **공통 plugin**이나 **버전 관리** 정의
    
- `buildscript` 블록 (예전 방식에서는 Gradle Plugin 의존성 선언)
    
- 공통 설정 (`allprojects {}` 또는 `subprojects {}` 블록 등)

- 예시
	``` kt
	plugins {
		id("com.android.application") version "8.5.0" apply false
		id("org.jetbrains.kotlin.android") version "1.9.23" apply false
	}
	
	tasks.register("clean", Delete::class) {
		delete(rootProject.buildDir)
	}
	```


## 3️⃣ **모듈(`app`)의 `build.gradle.kts`**

📍 **실제 앱 코드 빌드에 필요한 설정**

### 주요 역할

- `android {}` 블록 내에 SDK 버전, 빌드 타입 등 정의
    
- `dependencies {}` 에서 모듈별 의존성 관리

- 예시 
	```
	plugins {
	    id("com.android.application")
	    id("org.jetbrains.kotlin.android")
	}
	
	android {
	    namespace = "com.example.grid"
	    compileSdk = 34
	
	    defaultConfig {
	        applicationId = "com.example.grid"
	        minSdk = 24
	        targetSdk = 34
	        versionCode = 1
	        versionName = "1.0"
	    }
	
	    buildTypes {
	        release {
	            isMinifyEnabled = false
	        }
	    }
	}
	
	dependencies {
	    implementation("androidx.core:core-ktx:1.10.1")
	    implementation("androidx.appcompat:appcompat:1.6.1")
	    implementation("com.github.bumptech.glide:glide:4.15.1") // 예: JitPack 의존성
	}
	
	```



# 정리 

## 🔍 정리 요약

|파일명|역할|주요 내용|
|---|---|---|
|`settings.gradle.kts`|프로젝트 구조 + 저장소 관리|`include(":app")`, `repositories {}`|
|`build.gradle.kts` (루트)|공통 빌드 설정|플러그인 버전, 공용 태스크|
|`app/build.gradle.kts`|모듈 빌드 설정|Android 세부 설정, 의존성|
