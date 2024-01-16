## REST

REST는 **RE**presental **S**tate **T**ransfer의 약자로, 웹 서버스 실행을 위해 사용되는 스테이트리스(Stateless) 웹 아키텍처다. 이 아키텍처를 제공하는 웹 서비스를 **RESTful** 서비스라고 한다.

URI를 통해 표준화된 방식으로 RESTful 웹 서비스에 요청을 전송한다. URI는 이름으로 서버의 리소스를 식별한다.   
URL은 리소스가 존재하는 위치와 리소스를 가져오는 메커니즘을 지정하는 URI의 하위 집합이다.   
이 앱 코드에서 사용하는 URI와 URL은 다음과 같다:

- URI: android-kotlin-fun-mars-server.appspot.com
- URL: https://android-kotlin-fun-mars-server.appspot.com/realestate?hl=ko

### 웹 서비스 요청

1. **GET:** 서버 데이터를 가져온다.
2. **POST:** 서버에 새 데이터를 만든다.
3. **PUT:** 서버에 있는 기존 데이터를 업데이트한다.
4. **DELETE:** 서버에서 데이터를 삭제한다.

웹 서비스의 응답(response)은 **XML(eXtensible Markup Language)** 또는 **JSON(JavaScript Object Notation)** 과 같은 일반적인 데이터 형식 중 하나로 형식이 지정된다. JSON 형식은 key-value쌍으로 구조화된 데이터를 나타낸다.

<br>

## Retrofit

Retrofit 라이브러리는 REST 벡엔드와 앱 간 통신을 지원하는 외부 라이브러리다. [Retrofit의 문서를 참고하여 더 자세히 알아볼 수 있다.](https://github.com/square/retrofit)

### Retrofit 기본 종속 항목
Retrofit 라이브러리를 추가하려면 다음 코드를 앱 레벨의 gradle 파일의 의존성에 추가해야 한다.
```gradle
// Retrofit
implementation("com.squareup.retrofit2:retrofit:2.9.0")
// Retrofit with Scalar Converter
implementation("com.squareup.retrofit2:converter-scalars:2.9.0")
```
- 첫 번째 종속 항목은 Retrofit2 라이브러리 자체를 나타낸다
- 두 번째 종속 항목은 Retrofit 스칼라 변환기로, Retrofit이 JSON 결과를 String으로 반환할 수 있다.

Retrofit을 앱에 적용한 방법은 [여기서]() 볼 수 있다. 
