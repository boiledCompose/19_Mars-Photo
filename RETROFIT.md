# RETROFIT으로 인터넷에 연결하기

기본적인 RETROFIT 서비스 API 구현 방법은 다음과 같다:

1. 데이터 소스인 `AppApiService` 클래스를 만든다.
2. 기본 URL과 문자열을 변환하는 `변환기 팩토리`가 포함된 Retrofit 객체를 만든다.
3. Retrofit이 웹 서버와 통신하는 방법을 설명하는 인터페이스를 만든다.
4. Retrofit 서비스를 만들고 API 서비스에 관한 인스턴스를 앱의 나머지 부분에 노출한다.

## Manifest 파일

Android 앱이 인터넷에 접근하려면 `INTERNET` 권한이 필요하다. 이를 위해선 개발자가 `AndroidManifest.xml` 파일에 인터넷 접근 권한을 명시적으로 선언해야 한다.

- `manifests/AndroidManifest.xml`을 열고, `<application>` 태그 위에 다음 줄을 추가한다.
   ```xml
   <uses-permission android:name="android.permission.INTERNET" />
   ```

## ApiService 클래스

network 관련 클래스를 모아놓고 관리할 패키지를 만들어준다. 그 밑에 `MarsApiService.kt` 파일을 생성한다.

다음 코드를 `MarsApiService.kt` 파일에 작성한다.

1. 기본 URL 상수를 추가한다.
   
   ```kotlin
   private const val BASE_URL = "https://android-kotlin-fun-mars-server.appspot.com"
   ```
   
2.  Retrofit 객체를 빌드한다.
   
    - `addConverterFactor()`는 변환기 팩토리를 지정하는 메서드다. 변환기는 웹 서비스에서 얻은 데이터로 해야 할 일을 Retrofit에 알린다.
     `ScalarsConverter`는 웹 서비스의 JSON 응답을 문자열과 기타 기본 유형으로 변환해준다.
    - `build()`를 호출하여 객체를 빌드한다.
      
   ```kotlin
   private val retrofit = Retrofit.Builder()
       .addConverterFactory(ScalarsConverterFactory.create())
       .baseUrl(BASE_URL)
       .build()
   ```
  
3. `MarsApiService`라는 인터페이스를 정의한다. Retrofit이 HTTP 요청을 사용하여 웹 서버와 통신하는 방법을 정의하는 인터페이스다.
   
    - `getPhotos()` 메서드는 응답 문자열을 가져와 반환한다.
    - `@GET` 주석을 사용하여 Retrofit에 이 요청이 GET 요청임을 알리고 URL 엔드포인트를 지정한다.
      
   ```kotlin
   interface MarsApiService {
       @GET("photos")
       fun getPhotos(): String
   }
   ```

<br>

## 객체 선언

Retrofit 객체는 하나만 생성해서 관리하는 것이 좋다. 따라서 싱글톤 객체로 지정해도 된다.

- `object`는 싱글톤 객체를 나타낸다.
- `lazy`는 지연 초기화 객체 속성으로, 객체가 최초로 사용될 때 초기화되도록 설정할 수 있다. 
```kotlin
object MarsApi {
    val retrofitService : MarsApiService by lazy {
       retrofit.create(MarsApiService::class.java)
    }
}
```

<br>

## MarsViewModel에서 웹 서비스 호출

`viewModel`에선 REST 서비스를 호출한 다음 반환된 JSON 문자열을 처리하는 메서드를 구현한다. 

### ViewModelScope

`viewModelScope`는 앱의 각 `ViewModel`을 대상으로 정의된 기본 제공 코루틴 범위다. 이 범위에서 실행된 모든 코루틴은 `ViewModel`이 삭제되면 자동으로 취소된다.

`viewModelScope`를 사용하여 코루틴을 실행하고 백그라운드에서 웹 서비스를 요청할 수 있다. `viewModelScope`는 `ViewModel`에 속하기 때문에 구성 변경의 영향을 받지 않는다.

1. `MarsApiService.kt`에서 선언한 `getPhoto()`를 `suspend` 함수로 바꾼다. 이를 통해 코루틴에서 호출할 수 있게 된다.
   ```kotlin
   @GET("photo")
   suspend fun getPhotos(): String
   ```
2. `getMarsPhotos()` 메서드를 생성하고 `viewModelScope`를 실행한다. 
   ```
   private fun getMarsPhotos() {
      viewModelScope.launch {}
   }
   ```
3. `viewModelScope`에서 웹 서버에 접근하는 메서드를 호출하고 그 결과를 `uiState`에 저장한다.
   ```
   private fun getMarsPhotos() {
      viewModelScope.launch {
         val listResult = MarsApi.retrofitService.getPhotos()
         marsUiState = listResult
      }
   }
   ```

## 상태 UI 추가하기

- **Loading**: 앱이 데이터를 기다리는 중
- **Success**: 웹 서비스에서 데이터를 성공적으로 가져옴
- **Error**: 네트워크 오류 또는 연결 오류

1. 봉인 인터페이스 `sealed interface`를 사용하면 가능한 값을 제한하여 상태를 관리할 수 있다. 위와 같이 상태를 정의하고 선언된 상태들로 제한할 수 있다.
   ```kotlin
   sealed interface MarsUiState {
      data class Success(val photos: String) : MarsUiState
      object Error : MarsUiState
      object Loading : MarsUiState
   }
   ```
2. `MarsViewModel` 내에서 `marsUiState` 정의를 업데이트한다. 유형을 `MarsUiState`로 바꾸고 setter를 비공개로 설정하여 직접적인 변경을 금한다.
3. `getMarsPhotos()` 메서드에서 `marsUiState`의 값을 `MarsUiState.Success`로 초기화하도록 바꾼다.
4. `getMarsPhotos()`에서 예외처리를 통해 오류가 발생하면 `marsUiState`의 값을 `MarsUiState.Error`로 초기화하도록 바꾼다.
   ```
   private fun getMarsPhotos() {
      viewModelScope.launch {
          marsUiState = try {
              val listResult = MarsApi.retrofitService.getPhotos()
              MarsUiState.Success(listResult)
          } catch (e: IOException) {
              MarsUiState.Error
          }
      }
   }
   ```
5. UI를 보여주는 코틀린 파일에서 `MarsUiState` 값에 따라 나타나는 화면이 달라지도록 코드를 구현한다. `when`을 사용해서 구현을 하는데, 봉인 인터페이스로 인해 3개의 객체값에 대해서만 조건 처리를 하면 된다. `else`도 필요없다.

>[!NOTE]
> 위 코드 예제들은 웹 서버로부터 받은 JSON 형식의 결과를 그대로 화면에 표시하도록 구현되었다.   
> 불러온 응답 메세지를 태그별로 저장할 필요가 있다.

<br>

## kotlinx.serialization / JSON response parsing

**직렬화**는 애플리케이션에서 사용하는 데이터를 네트워크를 통해 전송할 수 있는 형식으로 변환하는 프로세스이다.   
**역직렬화**는 직렬화의 반대 개념으로, 외부 소스에서 데이터를 읽어 런타임 객체로 변환하는 프로세스이다. 

**kotlinx.serialization**은 JSON 문자열을 Kotlin 객체로 변환(역직렬화)하는 일련의 라이브러리를 제공한다.   
**kotlinx.serialization**은 외부 라이브러리임으로 의존성을 추가해야 한다.

### 의존성 추가

1. 앱 레벨의 `gradle` 파일의 `plugin` 블록에 플러그인을 추가한다.
   ```gradle
   id("org.jetbrains.kotlin.plugin.serialization") version "1.8.10"
   ```
2. `dependencied`에 라이브러리 의존성을 추가한다.
   ```gradle
   // Kotlin serialization
   implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.5.1")
   ```
3. 기존의 스칼라 변환기를 지우고 `serialization` 변환기 의존성을 추가한다.
   ```gradle
   //removed
   //implementation("com.squareup.retrofit2:converter-scalars:2.9.0")

   //added
   implementation("com.jakewharton.retrofit:retrofit2-kotlinx-serialization-converter:1.0.0")
   implementation("com.squareup.okhttp3:okhttp:4.11.0")
   ```

### 데이터 클래스 구현

데이터 클래스는 JSON 응답 항목과 동일하게 구성해야 한다. 그리고 `@Serializable` 태그를 통해 직렬화를 가능하도록 한다.

JSON 응답의 키 이름이 Kotlin의 권장 코딩 스타일과 일치하지 않을 수 있다. 예를 들어 JSON에서는 밑줄을 통해 단어를 구분하지만 Kotlin에서는 대소문자를 사용하여 단어를 구분하는 카멜 표기법을 사용한다. 이러한 혼란을 피하기 위해 `@SerialName`이라는 태그를 사용한다.

`@SerialName` 태그를 통해 데이터 클래스에 JSON 응답의 키 이름과 다른 변수 이름을 사용할 수 있다.

```
@Serializable
data class MarsPhoto(
   val id:String,

   @SerialName(value = "img_src")
   val imgSrc: String
)
```
