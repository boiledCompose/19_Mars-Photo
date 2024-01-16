## RETROFIT으로 인터넷에 연결하기

기본적인 RETROFIT 서비스 API 구현 방법은 다음과 같다:

1. 데이터 소스인 `AppApiService` 클래스를 만든다.
2. 기본 URL과 문자열을 변환하는 `변환기 팩토리`가 포함된 Retrofit 객체를 만든다.
3. Retrofit이 웹 서버와 통신하는 방법을 설명하는 인터페이스를 만든다.
4. Retrofit 서비스를 만들고 API 서비스에 관한 인스턴스를 앱의 나머지 부분에 노출한다.

### ApiService 클래스

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

### 객체 선언

Retrofit 객체는 하나만 생성해서 관리하는 것이 좋다. 따라서 싱글톤 객체로 지정해도 된다.

- `object`는 싱글톤 객체를 나타낸다.
- `lazy`는 지연 초기화 객체 속성으로, 객체가 최초로 사용될 때 초기화되도록 설정할 수 있다. 
```
object MarsApi {
    val retrofitService : MarsApiService by lazy {
       retrofit.create(MarsApiService::class.java)
    }
}
```

<br>

### MarsViewModel에서 웹 서비스 호출

