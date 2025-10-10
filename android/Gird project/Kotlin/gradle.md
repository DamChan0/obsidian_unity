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