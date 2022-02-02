지금까지는 Groovy DSL을 통해 의존성을 관리하고 있었고, 이를 너무 당연하게 생각하고 있었습니다.
[Gradle 도움말 및 레시피](https://developer.android.com/studio/build/gradle-tips#kts)를 보던 중 코드 예시가 Groovy/Kotlin으로 나뉘어 설명되어 있는 것을 보게 되었고..! 😲 (역시 공부할 건 끝이 없다..)
그리하여 새로운 프로젝트에는 한번 Kotlin DSL을 적용해 의존성을 관리해보기로 했습니다.

## Kotlin-DSL ❓️
우선 DSL(Domain Specific Language)은 도메인 특화 언어를 말합니다.
즉, Kotlin-DSL은 코틀린의 언어적 특성을 살려 Gradle(빌드 배포 도구) 스크립트를 작성하는 것을 목적으로 하는 DSL인 것입니다.

### Groovy DSL과의 비교 + 장단점

- 익숙하지 않아 이해가 어려웠던 Groovy대신 Kotlin을 사용할 수 있습니다.
- "여러 명이 같이 공유해 작업하는 빌드 스크립트에서는 자유로운 표현방식 보다는 약간의 제약을 가하는 표현방식을 사용하는 것이 좋다" 는 관점에서는 Kotlin DSL이 더 적합합니다.
```groovy
// groovy로 문자열 작성 시 작은 따옴표(')와 큰 따옴표(") 둘 다 사용 가능
// 함수 호출 시 괄호 생략 가능
// 속성 할당 시 '=' 없이 할당 가능

android {
  ...
  defaultConfig {
    ...
    versionCode 1
  }
}

dependencies {
    testImplementation "junit:junit:4.12"
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
}
```
```kotlin
// kotlin으로 문자열 작성 시 모든 문자열을 큰따옴표(")로 작성
// 함수 호출 시 괄호 필요
// 속성 할당 시 '=' 필요

android {
  ...
  defaultConfig {
    ...
    versionCode = 1
  }
}

dependencies {
 testImplementation("junit:junit:4.12")
 androidTestImplementation("com.android.support.test.espresso:espresso-core:3.0.2")
}
```
- Kotlin DSL에서는 코드 자동완성과 참조 / 문법 오류 코드 강조, 리팩토링이 가능합니다.
   - 실행해봐야 오류를 알 수 있는 Groovy의 특징이 다소 불편..
- 빌드 시간은 Groovy DSL이 Kotlin DSL보다 빠릅니다.

## Groovy DSL ➡ Kotlin DSL로 마이그레이션 도전 !

![](https://images.velog.io/images/yuuuzzzin/post/b45078e2-78d1-49ce-93d9-7efa9ea6dd1c/image.png)

### ① root project에 buildSrc 디렉터리 생성하기

### ② buildSrc 디렉터리 안에 'build.gradle.kts' 파일 생성하기

아래 코드를 적어 줍니다.

```kotlin
plugins {
    `kotlin-dsl` // enable the Kotlin-DSL
}

repositories {
    google()
    mavenCentral()
}
```

### ③ src>main>java 폴더 생성하고 그 안에 앱 레벨, 버전 정보나 라이브러리 관련 버전 정보를 저장할 파일 생성하기

하나의 파일을 만들어 관리해주어도 좋지만, 저는 `AppConfig.kt`와 `Dependencies`파일을 만들어 분리해 관리하기로 했습니다.

ex)

```kotlin
// AppConfig.kt

object AppConfig {
    const val compileSdk = 31
    const val buildToolsVersion = "30.0.2"
    const val minSdk = 24
    const val targetSdk = 31
    const val versionCode = 1
    const val versionName = "0.0.1"
}
```

```kotlin
// Dependencies.kt
object Versions {

    // AndroidX
    const val APP_COMPAT = "1.4.1"
    const val MATERIAL = "1.5.0"
    const val CONSTRAINT_LAYOUT = "2.1.3"

    // KTX
    const val CORE = "1.7.0"

    // TEST
    const val JUNIT = "1.1.3"

    // Android Test
    const val ESPRESSO_CORE = "3.4.0"
}

object Libraries {

    object AndroidX {
        const val APP_COMPAT = "androidx.appcompat:appcompat:${Versions.APP_COMPAT}"
        const val MATERIAL = "com.google.android.material:material:${Versions.MATERIAL}"
        const val CONSTRAINT_LAYOUT = "androidx.constraintlayout:constraintlayout:${Versions.CONSTRAINT_LAYOUT}"
    }

    object KTX {
        const val CORE = "androidx.core:core-ktx:${Versions.CORE}"
    }

    object Test {
        const val JUNIT = "androidx.test.ext:junit:${Versions.JUNIT}"
    }

    object AndroidTest {
        const val ESPRESSO_CORE = "androidx.test.espresso:espresso-core:${Versions.ESPRESSO_CORE}"
    }

}

```

+) 참조한 medium글을 보니 아래 코드처럼 함수를 만들어 더욱 편리하게 사용할 수도 있는 것 같다. 중복되는 부분은 저렇게 하나로 묶어 선언해놓고 여러 모듈에서 가져다 쓰면 좋을 것 같다.

```kotlin
object Libraries {

 //android ui
    private val appcompat = "androidx.appcompat:appcompat:${Versions.appcompat}"
    private val coreKtx = "androidx.core:core-ktx:${Versions.coreKtx}"
    private val constraintLayout =
        "androidx.constraintlayout:constraintlayout:${Versions.constraintLayout}"

    val appLibraries = arrayListOf<String>().apply {
        add(kotlinStdLib)
        add(coreKtx)
        add(appcompat)
        add(constraintLayout)
    }
}

//util functions for adding the different type dependencies from build.gradle file
fun DependencyHandler.kapt(list: List<String>) {
    list.forEach { dependency ->
        add("kapt", dependency)
    }
}

fun DependencyHandler.implementation(list: List<String>) {
    list.forEach { dependency ->
        add("implementation", dependency)
    }
}

fun DependencyHandler.androidTestImplementation(list: List<String>) {
    list.forEach { dependency ->
        add("androidTestImplementation", dependency)
    }
}

fun DependencyHandler.testImplementation(list: List<String>) {
    list.forEach { dependency ->
        add("testImplementation", dependency)
    }
}
```

### ④ 모듈별 build.gradle -> build.gradle.kts 로 변경

![](https://images.velog.io/images/yuuuzzzin/post/c890b28d-5f54-4d9d-9541-3feee07d76bf/image.png)

- 파일 이름을 위와 같이 변경해줍니다.
- 변경 후엔 문법이 맞지 않으니 오류가 날텐데...! 이를 [Groovy에서 KTS로 빌드 구성 이전](https://developer.android.com/studio/build/migrate-to-kts?hl=ko)과 [groovy에서 kotlin dsl로 이전하기](https://docs.gradle.org/current/userguide/migrating_from_groovy_to_kotlin_dsl.html)를 참조해 적절하게 바꾸어줍니다.

ex) 나의 프레젠테이션 계층 모듈의 경우 (예시입니다!!!)

```kotlin
// build.gradle.kts(:presentation)

plugins {
    id("com.android.application")
    id("kotlin-android")
}

android {
    compileSdk = AppConfig.compileSdk
    buildToolsVersion = AppConfig.buildToolsVersion

    defaultConfig {
        applicationId = "com.yuuuzzzin.id"
        minSdk = AppConfig.minSdk
        targetSdk = AppConfig.targetSdk
        versionCode = AppConfig.versionCode
        versionName = AppConfig.versionName

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        getByName("release") {
            isMinifyEnabled = false
            proguardFiles(getDefaultProguardFile("proguard-android.txt"), "proguard-rules.pro")
        }
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = "1.8"
    }
}

dependencies {
    implementation(project(":data"))
    implementation(project(":domain"))

    // AndroidX
    implementation(Libraries.AndroidX.APP_COMPAT)
    implementation(Libraries.AndroidX.MATERIAL)
    implementation(Libraries.AndroidX.CONSTRAINT_LAYOUT)

    // KTX
    implementation(Libraries.KTX.CORE)

    // TEST
    testImplementation(Libraries.Test.JUNIT)

    // AndroidTest
    androidTestImplementation(Libraries.AndroidTest.ESPRESSO_CORE)
}
```

### ⑤ project 수준 build.gradle -> build.gradle.kts로 변경, 마찬가지로 settings.gradle -> settings.gradle.kts로 변경

ex) 나의 경우

```kotlin
// build.gradle.kts (project 수준)

buildscript {
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath ("com.android.tools.build:gradle:7.0.4")
        classpath ("org.jetbrains.kotlin:kotlin-gradle-plugin:1.6.10")
    }
}

tasks.register("clean", Delete::class) {
    delete(rootProject.buildDir)
}
```

```kotlin
// settings.gradle.kts

rootProject.name = "ProjectName"

/**
 * presentation: 프레젠테이션 모듈
 * data: 데이터 모듈
 * domain: 도메인 모듈
 */
include(":presentation", ":data", ":domain")
```

## 공부하면서 느낀점
- 그동안 생각없이 라이브러리 버전 복사 붙여넣기..!를 하며 작성했던 빌드스크립트를 좀 더 효율적으로 사용할 수 있는 방법을 연구하면서 다루어보니 보기에도 훨씬 직관적이고 정돈된 느낌이었습니다.
- 단일모듈의 프로젝트만 진행해보다가 멀티모듈의 프로젝트를 진행하면서 모듈마다의 의존성 관리를 어떻게 해주어야하나 고민이었습니다.
하지만 buildSrc를 통해 한 곳에서 버전과 라이브러리 의존성 관련 코드를 작성하고 전체 모듈들이 이를 공유할 수 있게 해주니 훨씬 중복이 줄고 잦은 수정을 피할 수 있겠다라는 생각이 들었습니다.
- 생소한 부분을 공부해보고나니 한 3개월 더 늙은 기분입니다...

> 참고
> 
> [Groovy에서 KTS로 빌드 구성 이전](https://developer.android.com/studio/build/migrate-to-kts?hl=ko)
> 
> [우아한 형제들 기술 블로그 - ‘Gradle Kotlin DSL’ 이야기](https://techblog.woowahan.com/2625/)
> 
> [Gradle 공식 문서 - Kotlin DSL](https://docs.gradle.org/current/userguide/kotlin_dsl.html)
> 
> [Kotlin DSL: Gradle scripts in Android made easy](https://medium.com/android-dev-hacks/kotlin-dsl-gradle-scripts-in-android-made-easy-b8e2991e2ba)
> 