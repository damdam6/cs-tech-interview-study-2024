## **서블릿(Servlet)이란?**

> 서블릿(Servlet)이란 클라이언트의 요청을 처리하고, 그 결과를 반환하는 Servlet 클래스의 구현 규칙을 지킨 자바 웹 프로그래밍 기술이다. 서블릿은 웹 요청과 응답의 흐름을 간단한 메서드 호출만으로 체계적으로 다룰 수 있게 해준다.
> 

### 주요 특징

- 클라이언트의 요청에 대해 동적으로 작동하는 웹 어플리케이션 컴포넌트(**CGI**)
- html을 사용하여 요청에 응답한다.
- JAVA의 스레드를 이용하여 동작
- MVC패턴에서 컨트롤러로 이용됨
- 컨테이너에서 실행

### ✔️CGI(Common Gateway Interface)

**CGI**는 별도로 제작된 웹서버와 프로그램간의 교환방식. CGI방식은 어떠한 프로그래밍언어로도 구현이가능하며, 별도로 만들어 놓은 프로그램에 HTML의 Get or Post 방법으로 클라이언트의 데이터를 환경변수로 전달하고, 프로그램의 표준 출력 결과를 클라이언트에게 전송하는 것.

즉, 동적인 페이지를 생성하는 어플리케이션이 **CGI**이고, 서블릿은 자바로 구현 된 **CGI**이다.

## 서블릿의 동작 과정

![Untitled](https://velog.velcdn.com/images/falling_star3/post/4fabf50a-d3d7-4391-8eb5-0cb436379d71/image.png)

## **서블릿의 생명주기**

> **서블릿도 자바 클래스이므로 실행하면 초기화부터 서비스 수행 후 소멸하기까지의 과정**을 거친다. 이 과정을 **서블릿의 생명주기**라하며 각 단계마다 호출되어 기능을 수행하는 콜백 메서드를 서블릿 생명주기 메서드라한다.
> 
1. 클라이언트의 요청이 들어오면 컨테이너는 해당 서블릿이 메모리에 있는지 확인하고, 없을 경우 init()메서드를 호출하여 메모리에 적재한다. init()은 처음 한번만 실행되기 때문에, 서블릿의 스레드에서 공통적으로 사용해야 하는 것이 있다면 오버라이딩 하여 구현하면 된다. 실행 중 서블릿이 변경될 경우, 기존 서블릿을 destroy()하고 init()을 통해 새로운 내용을 다시 메모리에 적재한다.
2. init()이 호출된 후 클라이언트의 요청에 따라서 service() 메소드를 통해 요청에 대한 응답이 doGet()과 doPost()로 분기된다. 이 때 서블릿 컨테이너가 클라이언트의 요청이 오면 가장 먼저 처리하는 과정으로 생성된 HttpServletRequest, HttpServleResponse에 의해 request와 response 객체가 제공된다.
3. 컨테이너가 서블릿에 종료 요청을 하면 destroy() 메소드가 호출되는데 마찬가지로 한번만 실행되며, 종료시에 처리해야 하는 작업들은 destroy() 메소드를 오버라이딩하여 구현하면 된다.

### 서블릿 클래스 형식

```java
public class FirstServlet extends HttpServlet {
		//아래와 같은 메소드를 서블릿 생명주기 메소드라고 한다.
		@Override
    public void init() {
    ...
    //서블릿 요청 시 맨 처음 한 번만 호출
    //서블릿 생성 시 초기화 작업을 주로 수행
		}
    
    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp) {
    ...
    //서블릿 요청 시 매번 호출
    //실제로 클라이언트가 요청하는 작업을 수행
    }
    
    @Override
    public void doPost(HttpServletRequest req, HttpServletResponse resp) {
    ...
    }
    
    @Override
    public void destroy() {
    ...
    //서블릿이 기능을 수행하고 메모리에서 소멸될 때 호출
    //서블릿의 마무리 작업을 주로 수행
    }
}
```

## 서블릿 컨테이너

> 서블릿 컨테이너란, **구현되어 있는 servlet 클래스의 규칙에 맞게 서블릿을 담고 관리해주는 컨테이너**다. 서블릿 컨테이너는 클라이언트의 요청을 받아주고 응답할 수 있게, 웹서버와 소켓으로 통신하며 대표적인 예로 **톰캣**이 있다. (밑에 추가 설명)
클라이언트에서 요청을 하면 컨테이너는 **HttpServletRequest**, **HttpServletResponse** 두 객체를 생성하여 post, get여부에 따라 동적인 페이지를 생성하여 응답을 보낸다.
> 
- **IoC 컨테이너**
    
    서블릿의 실행 순서는 개발자가 관리하는게 아닌 서블릿 컨테이너가 관리를 한다. 즉 서블릿 컨테이너에 의해 사용자가 정의한 서블릿 객체가 생성되고 호출되고 사라진다.
    
- **웹서버와의 통신 지원**
    
    서블릿 컨테이너는 서블릿과 웹서버가 손쉽게 통신할 수 있게 해준다. 그래서 개발자가 서블릿에 구현해야 할 비지니스 로직에 대해서만 초점을 두게끔 도와준다.
    
- **서블릿 객체는 싱글톤으로 관리**
- **JSP도 서블릿으로 변환 되어서 사용**
    - 서블릿은 자바 소스코드 속에 HTML코드가 들어가는 형태인데, JSP는 이와 반대로 **HTML 소스코드 속에 자바 소스코드가 들어가는 구조**를 갖는 웹어플리케이션 프로그래밍 기술
    - 서블릿 규칙은 꽤나 복집하기 때문에 JSP가 나오게 되었는데, JSP는 서블릿 컨테이너에 의하여 서블릿 클래스로 변환하여 사용
- **서블릿 생명주기 관리**
    
    서블릿 컨테이너는 서블릿 클래스를 로딩하여 인스턴스화하고, 초기화 메소드를 호출하고, 요청이 들어오면 적절한 서블릿 메소드를 호출한다. 또한 수명이 다 된 서블릿을 적절하게 가비지 콜렉터를 호출하여 필요없는 자원 낭비를 막아준다.
    
- **멀티쓰레드 지원 및 관리**
    
    서블릿 컨테이너는 Request가 올 때마다 새로운 자바 쓰레드를 하나 생성하는데, HTTP 서비스 메소드를 실행하고 나면, 쓰레드는 자동으로 죽게 된다.
    
- **선언적인 보안 관리**
    
    서블릿 컨테이너를 사용하면 개발자는 보안에 관련된 내용을 서블릿 또는 자바 클래스에 구현해 놓지 않아도 된다. 일반적으로 보안관리는 XML 배포 서술자에다가 기록하므로, 보안에 대해 수정할 일이 생겨도 자바 소스 코드를 수정하여 다시 컴파일 하지 않아도 보안관리가 가능하다.
    

### ✔️WAS와 서블릿 컨테이너의 차이

- **웹서버**
    
    웹은 태생적으로 변하지 않는 고정된 문서, 파일을 쉽게 읽기 위해 개발한 기술로 사용자가 요청한 파일을 그대로 전달하는 서버를 사용했는데 이를 웹서버
    
- **WAS(Web Application Server)**
    - **WAS** = Web Server + 웹 컨테이너(=서블릿 컨테이너)
    - 웹이 발달하며 사용자의 상황에 따라 동적으로 바뀌는 문서를 제공해야 했고, 이런 기능을 하는 별도의 특화 서버를 사용하는데, 이를 **WAS(웹 어플리케이션 서버)**
    - **WAS**는 J2EE의 스펙을 구현하여, 서블릿(Servlet)이나 JSP로 작성된 애플리케이션을 실행하는 소프트웨어
- **WAS와 서블릿 컨테이너 비교**
    
    ![Untitled](https://cdn.inflearn.com/public/comments/fc2e6ae0-a258-4c22-aa6b-a57e37e4e56d/%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%83%E1%85%A1%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%A5%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%86%B7.drawio.png)
    
    - **Tomcat**
        - 톰캣은 서블릿 컨테이면서도 WAS의 역할을 어느정도 수행
        - 하지만, 톰캣은 J2EE 스펙을 전부 구현하고 있지는 않기 때문에(**EJB**기능을 지원해주지 않는다.) 엄밀히 말하자면 서블릿 컨테이너에 더 가깝다.
        - **EJB(Enterprise Java Beans)**: Java에서 제공하는 분산 컴포넌트 기술로 비즈니스 로직이나 데이터, 메시지를 처리할 수 있습니다.(?)

## Dispatcher-Servlet

> 클라이언트가 요청을 주면, 서블릿 컨테이너가 요청을 받는데, 이때 제일 앞에서 서버로 들어오는 모든 요청을 처리하는 Front Controller라는 것을 Spring에서 정의하였고, 이를 Dispatcher-Servlet이라고 한다.
즉, Dispatcher-Servlet을 이용한다는 것은 스프링에서 제공하는 **MVC모델**을 이용하겠다는 것
> 

### 장점

- Dispatcher-Servlet은 web.xml의 역할을 상당히 축소시켜주었음. 과거에는 모든 Servlet에 대해 URL매핑을 활용하기 위해서 web.xml에 모두 등록해 주어야 했지만, Dispatcher-Servlet이 해당 어플리케이션으로 들어오는 모든 요청을 핸들링해 주면서 작업의 효율을 높였다.

### 정적 자원의 처리

Dispatcher-Servlet이 모든 요청을 처리하다보니 이미지나 HTML/CSS/JavaScript 등과 같은 정적 파일에 대한 요청마저 모두 가로채는 까닭에 정적 자원을 불러오지 못하는 상황도 발생

이를 해결하기 위한 2가지 방법

1. **정적 자원 요청과 애플리케이션 요청을 분리**
    1. 모든 요청에 대해서 앞에 추가적인 URL을 붙여주어야 하므로 직관적인 설계가 될 수 없다.
2. **애플리케이션 요청을 탐색하고 없으면 정적 자원 요청으로 처리**
    1. Dispatcher-Servlet이 요청을 처리할 컨트롤러를 먼저 찾고, 요청에 대한 컨트롤러를 찾을 수 없는 경우에, 2차적으로 설정된 자원 경로를 탐색하여 자원을 탐색하는 방법
    2. 이렇게 영역을 분리하면 효율적인 리소스 관리를 지원할 뿐 아니라 추후에 확장을 용이하게 해준다는 장점

### 참고(출처)

- [https://velog.io/@falling_star3/Tomcat-서블릿Servlet이란](https://velog.io/@falling_star3/Tomcat-%EC%84%9C%EB%B8%94%EB%A6%BFServlet%EC%9D%B4%EB%9E%80)
- https://mangkyu.tistory.com/14
- [https://www.inflearn.com/community/questions/667363/서블릿-컨테이너-was-차이?srsltid=AfmBOorG07iisT9xBPTgDyGv-AjL7T3Bq0zBQul49Ce5FrfV4SCHobY0](https://www.inflearn.com/community/questions/667363/%EC%84%9C%EB%B8%94%EB%A6%BF-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-was-%EC%B0%A8%EC%9D%B4?srsltid=AfmBOorG07iisT9xBPTgDyGv-AjL7T3Bq0zBQul49Ce5FrfV4SCHobY0)
- [https://velog.io/@mooh2jj/WAS는-서블릿-컨테이너-파헤치기](https://velog.io/@mooh2jj/WAS%EB%8A%94-%EC%84%9C%EB%B8%94%EB%A6%BF-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0)
- https://bny9164.tistory.com/56
- https://mangkyu.tistory.com/18
- https://riimy.tistory.com/87
