서버 주소나 API key값과 정보들은 협업하는 팀원이 아니면 노출되어선 안 되는 민감한 정보입니다.

그러므로 그 값을 소스 코드에 그대로 노출시키지 않고 안전하게 숨겨야 할 필요가 있습니다.

## local.properties를 통해 값 숨기기

### ① local.properties 파일에 값을 저장하기

처음 local.properties 퍄일을 열면 SDK 경로가 적혀있을텐데 그 아래에 내가 사용할 API Key값을 정의해줍니다.

```java
// local.properties

sdk.dir = Sdk 경로

API_KEY = "API Key Value"
```

local.properties는 기본적으로 gitignoere로 설정되어 있어 깃허브에도 올라가지 않습니다.

### ②-1 Groovy 사용하는 경우

build.gradle(:app)을 다음과 같이 수정해줍니다.

```groovy
Properties properties = new Properties() 
properties.load(project.rootProject.file('local.properties').newDataInputStream()) 

android {

	... 

	defaultConfig {
    
    	... 
        
        buildConfigField "String", "SAMPLE_API_KEY", properties["API_KEY"] 
        
        } 
 }
```

### ②-2 Kotlin-DSL 사용하는 경우

build.gradle.kts(:app)을 다음과 같이 수정해 local.properties의 변수를 불러옵니다. 저는 getApiKey라는 함수를 만들어 local.properties의 변수를 불러오도록 만들어주었습니다.

```kotlin
import com.android.build.gradle.internal.cxx.configure.gradleLocalProperties

android {

	...
    
    defaultConfig {
    
    	...
        
 		buildConfigField("String", "API_KEY", getApiKey("API_KEY"))

    }
    
}

fun getApiKey(propertyKey: String): String {
    return gradleLocalProperties(rootDir).getProperty(propertyKey)
}
```

②-1과 ②-2 둘 다 BuildConfig에 추가해주는 방법입니다. 

### ③ 프로젝트 빌드해주기

빌드 후에 BuildConfig.java 파일에 해당 키값이 생성된 것을 볼 수 있습니다.

### ④ BuildConfigField 값 호출하기

```kotlin
// BuildConfigField 값 호출
const val API_KEY = BuildConfig.API_KEY
```

위와 같은 방식으로 호출해 사용할 수 있습니다.

```kotlin
/* 네트워크 통신을 위한 모듈 */

@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    private const val BASE_URL = BuildConfig.BASE_URL

	...
    
    /* Retrofit2 통신 모듈 */
    @Singleton
    @Provides
    fun provideRetrofit(
        okHttpClient: OkHttpClient
    ): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addCallAdapterFactory(RxJava3CallAdapterFactory.create())
            .addConverterFactory(
                TikXmlConverterFactory.create(
                    TikXml.Builder().exceptionOnUnreadXml(false).build()
                )
            )
            .client(okHttpClient)
            .build()
    }
```

이런 식으로 서버 주소를 저장하고 가져와 통신 모듈에 넣어줄 수도 있습니다.

> 참고
> [Gradle 도움말 및 레시피](https://developer.android.com/studio/build/gradle-tips)