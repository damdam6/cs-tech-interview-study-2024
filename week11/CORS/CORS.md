## **CORS가 없다면?**

1. 공격자는 자신의 사이트에 악성 스크립트를 심어놓을 수 있다. 피해자는 해당 사이트를 정상적으로 이용하는 평범한 사람이다. 피해자가 해당 사이트를 이용하게 되면서, 공격자가 심어놓은 악성 스크립트가 동작하게 된다.
2. 예를 들어 악성스크립트로 공격자는 피해자가 '[google.com](http://google.com/)'에 접속한 화면을 볼 수 있을 뿐만 아니라, (가능하다면) 쿠키 값을 가져와 세션을 탈취할 수도 있다.

## SOP의 등장

- 위와 같은 공격을 막기 위해 등장한 것이 SOP(Same-Origin Policy)이다. 브라우저는 기본적으로 SOP 정책을 따르고 있다.
- SOP는 2011년 RFC 6454에서 등장한 보안 정책으로 "같은 출처에서만 리소스를 공유할 수 있다"라는 규칙을 가진 정책이다.
- 모든 브라우저는 기본적으로 SOP 정책을 따른다.
- 쉽게 말해서, https://velog.io/write?id=something 이라는 URL이 있다면 https://velog.io/ 까지만 검사한다는 뜻이다.
- SOP가 적용되면서 위와 같은 fetch는 '다른 출처'에서 데이터를 받아오기 때문에 차단된다. 이 때, 차단하는 주체는 클라이언트의 브라우저이다.
    - 클라이언트와 서버가 정상적으로 통신은 완료하지만(code: 200), 브라우저가 클라이언트의 javascript에게 결과값을 넘겨주지 않는 방식으로 동작한다.
- 하지만 SOP는 개발자들에게 엄청난 불편함를 가져오게 된다.
    - 개발할 때 '다른 출처'의 API 서버를 두는 경우가 많기 때문
    - 개발할 때 '다른 출처'의 외부 리소스를 가져다 쓰는 경우가 많기 때문 이다.
- 그래서 SOP의 불편함을 해소하면서, 보안을 지키기 위해 등장한 것이 CORS(Cross-Origin Resource Sharing)이다. CORS를 지키면, '다른 출처'에서도 데이터를 불러올 수 있도록 해준다라는 것이다.

## **CORS란?(Cross-Origin Resource Sharing)**

한국어로 직역하면 '교차 출처 리소스 공유'라고 해석할 수 있다. 여기서 '교차 출처(Cross-Origin)'이란 '다른 출처'를 의미한다. 즉, Cross-Origin의 Resrouce를 공유하는 정책이라고 볼 수 있다.

즉, **CORS란 도메인이 다른 서버끼리 리소스를 주고 받을 때 보안을 위해 설정된 정책**이라고 생각하면 된다.

예를 들어, 웹 사이트 A가 API 서버 B에서 데이터를 가져오려 할 때, API 서버 B에서 CORS 허용 설정이 되어 있지 않으면 웹 브라우저에서 API 접근이 거부될 수 있다.

또한 프론트엔드와 백엔드가 협업하면서 각자 따로 서버를 띄우게 되었을 경우도 마찬가지다. 서로 다른 React 서버(3000 포트)와 Springboot(8080 포트) 서버가 리소스를 주고 받으려 한다면 포트가 달라 서로 다른 출처로 판단되어 CORS 위반 에러가 발생한다.

### 기본 동작 방식

서버에서 응답시 `Access-Control-Allow-Origin` 헤더를 설정해주면 CORS를 적용할 수 있고, SOP를 우회할 수 있다. 여기서 출처를 비교하는 로직이 브라우저에 구현되어 있으므로 요청에 대한 응답을 받은 후에 브라우저에서 그 응답을 무시하는 식으로 작동하게 된다. 그러므로 브라우저를 통하지 않고 서버 간 통신을 할 때는 이 정책이 적용되지 않는다.

### **Cross-Origin (다른 출처) 판단 기준**

- 두 개의 출처가 서로 같다고 판단하는 로직은, 두 URL의 구성 요소 중 Scheme(프로토콜), Host(도메인), Port, 이 3가지만 동일하면 된다.

![Untitled](https://velog.velcdn.com/images/effirin/post/4d367bb4-355b-4fd0-8a72-41d375bd85a9/image.png)

- 따라서, 일반적으로는 same-origin이란 scheme(프로토콜), host(도메인), 포트가 같다는 말이며, 이 3가지 중 하나라도 다르면 cross-origin이다.
- 각 브라우저들의 독자적인 출처 비교 로직을 따라가는 경우도 있다.
    
    **예) `https://evan-moon.github.io` 와 `https://evan-moon.github.io:8000`**
    
    예시로 든 출처의 경우 포트 번호가 포함되지 않았기 때문에 판단하기가 애매하다. RFC 6454의 Comparing Origins 섹션에는 “만약 출처가 스킴/호스트/포트의 삼중 체계라면…” 이라는 전제가 붙어있기 때문에 어떻게 해석하냐에 따라 구현이 달라질 수 있기 때문이다.
    
    그래서 이런 경우에는 각 브라우저들의 독자적인 출처 비교 로직을 따라가게 된다.
    

### CORS의 동작 시나리오 3가지(세부 동작)

1. **Simple Request**
    
    다음과 같은 사항에 부합되는 일반적인 요청에 대해서는 CORS 정책 검사를 하지 않는다.
    
    > 
    > 
    > 1. 요청의 메소드는 **`GET`**, **`HEAD`**, **`POST`** 중 하나여야 한다.
    > 2. Request Header에는 다음 속성만 허용**`Accept`**, **`Accept-Language`**, **`Content-Language`**, **`Content-Type`**, **`DPR`**, **`Downlink`**, **`Save-Data`**, **`Viewport-Width`**, **`Width`**
    >     1. `Authorization`헤더를 사용하면 Simple Request에 해당되지 않음
    >     2. 그 외에 다른 커스텀 헤더 같은 것들이 포함되면 Simple Request에 해당되지 않음
    > 3. 만약 **`Content-Type`**를 사용하는 경우에는 **`application/x-www-form-urlencoded`**, **`multipart/form-data`**, **`text/plain`**만 허용된다.
    >     1. **`text/xml`**이나 **`application/json`는 조건에 해당되지 않음**
    > 4. 요청에 사용된 XMLHttpRequestUpload 객체에는 이벤트 리스너가 등록되어 있지 않다. 이들은 [XMLHttpRequest.upload](https://developer.mozilla.org/ko/docs/Web/API/XMLHttpRequest/upload) 프로퍼티를 사용하여 접근한다.
    > 5. 요청에 [ReadableStream](https://developer.mozilla.org/ko/docs/Web/API/ReadableStream) 객체가 사용되지 않는다.
2. **Preflight Request**
    - 이 방식은 일반적으로 우리가 웹 어플리케이션을 개발할 때 가장 많이 마주치는 시나리오로 이에 해당하는 상황일 때 브라우저는 요청을 한번에 보내지 않고 **예비 요청과 본 요청**으로 나누어서 서버로 전송한다.
    - Simple Request와 같은 요청이 아닌 경우 브라우저는 접근할 리소스를 가지고 있는 서버에 **preflight Request (예비 요청)을 보낸다.**
    - 이 예비 요청에는 HTTP 메소드 중 **`OPTIONS`** 메소드가 사용된다. 예비 요청의 역할은 본 요청을 보내기 전에 **브라우저 스스로 이 요청을 보내는 것이 안전한지 확인하는 것**이다. 이때 내용물은 없이 헤더만 전송
    - Preflight Request 시나리오의 흐름 과정
        
        ![Untitled](https://velog.velcdn.com/images/effirin/post/7f4848b8-a230-458a-ba09-3b8d9d982370/image.png)
        
        1. Cross-Origin 요청인 것을 판단하여 아래와 같이 요청에 "Origin: [https://foo.example](https://foo.example/)" 헤더를 추가한다. 브라우저는 요청에 실제 헤더 정보를 추가하여 외부 서버로 예비 요청을 보낸다.
            1. **Access-Control-Request-Method:** 실제 요청에서 어떤 HTTP 메서드를 사용할 것인지 서버에 알립니다.
            2. **Access-Control-Request-Headers:** 실제 요청에서 어떤 헤더들이 포함될지를 서버에 알립니다.
        2. 서버는 이 예비 요청에 대한 응답으로 현재 자신이 어떤 것들을 허용하고 있는지에 대한 정보를 response header에 담아서 브라우저에게 다시 보내주게 된다.
            1. **Access-Control-Allow-Origin:** 허가된 Origin
            2. **Access-Control-Allow-Methods:** 허가된 메소드
            3. **Access-Control-Allow-Headers:** 허가된 헤더
            4. **Access-Control-Max-Age:** 응답 캐시 유효 시간
        3. 위 예시에서 표시된 정보는 **'해당 API가 Cross-Origin에 대해서 POST, GET, OPTIONS와 커스텀 헤더인 X-PINGOTHER 그리고 Content-Type을 허용한다'**는 의미이다. 이에 해당되는 것들은 안전하다고 판단하여 CORS 위반으로 간주하지 않고, 브라우저는 POST인 본 요청을 브라우저는 외부 서버로 보낸다.
3. **Credential Request**
    - 이 시나리오는 헤더에 인증과 관련된 정보(쿠키, 토큰 등)를 담아서 보내는 Credential Request (인증된 요청)을 사용하는 방법이다.
    - CORS의 기본적인 방식이라기 보다는 다른 출처 간 통신에서 좀 더 보안을 강화하고 싶을 때 사용하는 방법이다.
    - 예를 들어, 자바스크립트의 fetch API를 사용하거나 Axios, Ajax 등을 사용할 때 서버로 쿠키를 함께 전송해야 하는 경우가 있는데, 요청에 쿠키가 담기게 되면 Credentialed Request 허용이 되어 있어야 한다.
    - 즉, **인증과 관련된 정보를 담을 수 있게 해주는 옵션 'credentials'**를 줘야하는데, 이 때 서버 쪽에서 응답 헤더에 Access-Control-Allow-Credentials: true를 보내주지 않는다면 브라우저에서 응답을 받는 것을 거부하게 된다.

### 참고(출처)

[https://velog.io/@effirin/CORS란-무엇인가](https://velog.io/@effirin/CORS%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80)

[https://velog.io/@hunjison/SOP-CORS는-보안에서-왜-필요할까](https://velog.io/@hunjison/SOP-CORS%EB%8A%94-%EB%B3%B4%EC%95%88%EC%97%90%EC%84%9C-%EC%99%9C-%ED%95%84%EC%9A%94%ED%95%A0%EA%B9%8C)
