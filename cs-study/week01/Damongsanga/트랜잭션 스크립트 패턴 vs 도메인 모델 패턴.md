# 트랜잭션 스크립트 패턴 vs 도메인 모델 패턴 (CQRS & 이벤트 소싱)

태그: DDD
날짜: 2024년 6월 10일

## 트랜잭션 스크립트 패턴 vs 도메인 모델 패턴 (CQRS & 이벤트 소싱)

비즈니스 로직을 당연히 서비스 계층에 주로 작성하고 있었는데, 도메인 객체 (DDD에서의 엔티티)가 모두 비즈니스 로직을 가지고 있고,

서비스 계층은 트랜잭션 관리만 맡고 적절한 도메인 객체를 선택하여 비즈니스 책임을 위임해야 한다는 말을 어렴풋이 들은 적이 있었다.

그러면 실제 비스니스 로직은 어디에 들어가야 하는가?

# 1. 트랜잭션 스크립트 패턴 vs 도메인 모델 패턴

1. **트랜잭션 스크립트 패턴**
   - 특징
     - **엔티티에 비즈니스로직이 거의 없고, 서비스 계층에서 비즈니스 로직을 처리하는 방법**
     - **엔티티는 DTO의 역할 만하고 위에 해당 비즈니스 로직이 서비스 계층으로 옮겨짐**
       - 도메인 엔티티는 아무런 비즈니스 로직이 없는 POJO 클래스로 데이터를 전달하는 역할만 수행
     - 하나의 트랜잭션으로 구성된 로직을 단일 함수/스크립트에서 처리
     - JSP에서 주로 적용
   - 장점
     - 단순, 구현이 매우 쉬움
   - 단점
     - 비즈니스 로직이 복잡해질수록 난잡해짐
     - 분석, 설계에 대한 개념이 약함. 코드 중복 증가
     - 공통 모듈 분리 X
     - 서비스에 로직이 들어가면 테스트에 DB가 관여하기 때문에 테스트 하기가 POJO에 비해 어려워짐. 테스트가 DB 상태에 종속적이기 때문
2. **도메인 모델 패턴**
   - 특징
     - 객체 지향 분석 설계에 기반하여 구현하고자 하는 도메인(비즈니스) 모델을 생성하는 패턴
     - **서비스의 많은 로직이 엔티티로 이동하고, 서비스는 엔티티를 호출하는 정도의 얇은 비즈니스 로직을 가짐**
     - 데이터와 프로세스가 혼합되어있음
     - 객체간의 복잡한 연관관계
     - 도메인 모델과 데이터베이스 테이블 사이 매핑을 어떻게 처리할 것인가
   - 장점
     - 객체 지향에 기반한 재사용성, 확장성, 그리고 유지 보수의 편리함
     - 엔티티와 VO가 개념적으로 잘 정리됨
     - 비즈니스 전문가와 개발자 간의 의사소통이 원활
   - 단점
     - 도메인 모델 구축에 들어가는 노력

### 클린코드 6장

간단하게 책의 내용을 정리하자면, “모든”것을 객체지향적으로 구현해야 한다는 생각은 잘못된 것일 수 있다는 것이다. 객체 지향적인 코드가 가진 장점은 절차적인 코드가 가진 단점이지만, 이 반대 역시 성립하기 때문이다.

그 예시로 시스템을 구현할 때 새로운 자료 타입을 추가하는 유연성이 필요하다면 객체지향적인 코드가, 새로운 동작을 추가하는 유연성이 필요하다면자료 전달 객체(ex. DTO)와 절차적인 코드가 더 적합 할 수도 있다.

- 예시
  **절차적인 도형 클래스**

  ```java
  public class Square {
  	public Point topLeft;
  	public double side;
  }

  public class Rectangle {
  	public Point topLeft;
  	public double height;
  	public double width;
  }

  public class Circle {
  	public Point center;
  	public double radius;
  	public double width;
  }

  public class Geometry {
  	public final double PI = 3.141592653585793;

  	public double area(Object shape) throws NoSuchShapeException {
  		if(shape instanceOf Square) {
  			Square s = (Square)shape;
  			return s.side * s.side;
  		}
  	else if(shape instanceOf Rectangle) {
  			Rectangle r = (Rectangle)shape;
  			return r.height * r.width;
  		}
  	else if(shape instanceOf Circle) {
  			Circle c = (Circle)shape;
  			return PI * c.radius * c.radius
  		}
  	}
  }
  ```

  만약 Geomertry에 area() 함수를 변경하거나, 각 도형 객체들의 둘레 길이를 구하는 perimeter() 라는 새로운 함수를 추가해야 한다면?
  Square, Rectangle, Circle.. 도형 클래스들은 전혀 영향을 받지 않는다!
  그러나 만약 새로운 도형 클래스를 추가해야 한다면?
  Geometry 클래스의 모든 함수를 변경해야 할 것이다.
  **객체 지향적인 도형 클래스**

  ```java
  public class Square implements Shape {
  	public Point topLeft;
  	public double side;

  	public double area() {
  		return side*side;
  	}
  }

  public class Rectangle implements Shape {
  	public Point topLeft;
  	public double height;
  	public double width;

  	public double area() {
  		return height*width;
  	}
  }

  public class Circle implements Shape {
  	public Point center;
  	public double radius;
  	public double width;

  	public double area() {
  		return PI*radius*radius;
  	}
  }
  ```

  객체 지향적인 코드에서는 반대로 새로운 클래스를 추가하는 것에 다른 클래스가 영향을 받지 않는다.
  그저 부모 클래스인 Shape 클래스만 상속받으면 되기 때문이다.
  그러나 만약 Shape의 함수가 변경되거나 추가된다면?
  Shape 클래스를 상속받은 모든 자식 클래스들도 영향을 받게 된다.

### +a) [Spring에서 트랜잭션 스크립트 패턴을 써야하는 이유?](https://chem-en-9273.tistory.com/135)

**DB 커넥션을 잡고있는 시간 때문이다**

위 블로그에 따르면,

DB 커넥션 풀 자원 유한성 관점에서는 트랜잭션의 범위를 최소화해야한다

그런데 도메인 모델 패턴을 사용하는 경우 **엔티티를 로드하고, 여러 비즈니스 로직을 수행한 후, 변경된 엔티티를 저장하기까지의 시간이 길어지면 DB 커넥션을 잡고 있는 시간이 길어짐**

이와 달리 서비스단에서의 하나의 트랜잭션이 비즈니스로직을 담당하여 처리.

그 결과 **데이터베이스와의 상호작용을 빠르게 끝내고, 필요한 작업을 빠르게 처리하게 됨**

# **2. OSIV (Open-Session-In-View)**

**왼쪽은 OSIV ON, 오른쪽은 OSIV OFF**

![Untitled](./트랜잭션%20스크립트%20패턴%20vs%20도메인%20모델%20패턴/Untitled%209.jpeg)

- **HTTP 요청의 시작부터 끝까지 데이터베이스 세션을 열어두는 방식**
  - DB 커넥션 시작 시간부터 API 응답 끝날 때까지 영속성 컨텍스트와 데이터베이스 커넥션 유지
  - 영속성 컨텍스트의 연장
- **Hibernate와 같은 ORM에서 자주 사용되는 패턴**
  - **이 패턴의 주된 목적은 지연 로딩(Lazy Loading)을 적극 활용하기 위함**
- **이 방식은 뷰 렌더링 도중에도 엔티티에 접근할 수 있게 해주지만, 동시에 DB 커넥션을 더 오래 잡고 있게 만듬**
  - 실시간 트래픽이 중요한 애플리케이션에서는 커넥션이 부족해질 수 있음
- **도메인 주도 개발 방식에서는 엔티티를 뷰에서 직접 접근할 일이 잦기 때문에 OSIV를 사용하는 경우가 많다. 이는 DB 커넥션을 오래 유지하게 되는 주요 원인이 됨**

### 해결방법

OSIV를 끈 채로 개발을 한다면 컨트롤러, 뷰단에서 지연 로딩을 사용하지 못하는 문제가 있다. (이로 인한 성능 이슈 발생)

**이를 위해 CQRS 패턴을 도입하여, 쿼리 API가 들어오면 엔티티 대신 DTO로 반환하면 된다.**

장점으로 지연로딩 문제 예방, 성능 최적화, 데이터 보안, 유지보수성이 증가한다!

### 세부내용 1 (OSIV OFF)

- Spring에서는 개발 확장과 유연성을 위해 OSIV가 default로 켜져있다
- 그러나 트랜잭션 스크립트 패턴에서는 트랜잭션의 범위가 명확하게 서비스 단에서만 지속되도록 설계하는 것
- **원격 서비스를 많이 호출하거나 트랜잭션 컨텍스트 외부에서 많은 일이 발생하는 경우 OSIV를 모두 비활성화하는 것이 좋다.**
  ```yaml
  spring:
  	jpa:
  		open-in-view: false
  ```
- 그러나 OSIV를 아예 끄면 지연로딩을 할 수 없다
  - **OSIV 패턴을 비활성화하면, 트랜잭션이 종료될 때 영속성 컨텍스트도 함께 종료되므로, 트랜잭션 외부에서는 지연 로딩을 수행할 수 없게 된다.**
  - **이 경우, 지연 로딩은 트랜잭션 내부에서만 사용해야 한다**
- 따라서 특정 URL에서만 OSIV를 제외시켜주는 커스텀 필터를 적용해볼 수 있다
  - 자세한 내용은 [여기](https://velog.io/@haron/Spring-Connection-Pool-%EC%9D%B4-%EB%B6%80%EC%A1%B1%ED%95%98%EB%8B%A4%EA%B3%A0%EC%9A%94)

### 세부내용 2 (커맨드 & 쿼리 분리)

### **CQRS 패턴**

1. 커맨드 (비즈니스 로직)
   - CUD
   - 정책적인 것임으로 잘 변경되지 않음
   - 특정 수의 엔티티를 등록, 수정하는 것임으로 성능 최적화가 크게 문제되지 않음
2. 조회 API
   - R
   - 자주 변동되고 라이프사이클이 빠름
   - 성능 최적화가 중요

**⇒ 서로 라이프 사이클이 다름으로 명확히 분리**

# 3. CQRS **(Command Query Responsibility Segregation) 란?**

![Untitled](./JPA%20&%20DDD%20Entity%20차이/Untitled.png)

**데이터 저장소로부터의 읽기와 쓰기 작업을 분리하여 별도의 데이터 모델로 나누는 패턴**

### 왜 분리해야 하는가?

1. 읽기와 쓰기의 부하는 같지 않다. 각각에 다른 성능 요구
2. 읽기와 쓰기 작업에 사용되는 데이터 표현들이 일치하지 않음
   1. 읽기의 경우 다른 형태의 DTO를 반환하는 매우 다양한 쿼리들을 수행
3. 보안 관리를 위한 데이터 모델 분리 필요 (예시로 사용자 데이터를 조회할 때 비밀번호 노출 불가)
4. 동일 데이터 세트에 따른 병렬 작업 수행 시 데이터 경합 발생 문제

### 어떻게?

!https://velog.velcdn.com/images/everyhannn/post/5b6c3ed6-2db9-4266-8f4d-61a8e99c03ad/image.png

1. 읽기와 쓰기 컴포넌트를 분리한다
   - Aggregate 패턴을 도입하여 쓰기 도메인 모델을 그룹화, 트랜잭션 일관성을 제공한다.
2. 읽기와 쓰기 저장소를 분리한다.
   - 왼쪽 그림과 같이 하나의 read와 write 저장소를 사용하여도 되지만 오른쪽 그림과 같이 물리적으로 두 개의 저장소로 나눈 뒤 동기화를 유지하는 매커니즘을 도입할 수 있다.
   - **이론적으로 CQRS와 Event Sourcing은 종속되지 않지만 현실에서는 이 두개는 필수적으로 종속된다. (후술)**
3. 명령(Command)을 통해 데이터를 쓰고,
   - 데이터 중심적이 아니라 수행할 작업 중심이 되어야 한다. 예를 들면 '호텔룸의 상태를 예약됨으로 변경한다'가 아니라 '호텔 룸 예약'과 같이 생성되어야 한다.
   - 보통 동기적으로 처리되기보단, 비동기적으로 큐에 쌓인 후 수행된다
   - Validation을 통한 검증 대상이다
4. 쿼리(Query)를 통해 데이터를 읽는다.
   - 쿼리(Query)는 데이터 베이스를 결코 수정하지 않는다. 쿼리(Query)는 **어떠한 도메인 로직도 캡슐화하지 않은 DTO만을 반환한다.**
   - 검증 대상이 아니다.
5. 읽기와 쓰기 데이터를 동기화한다. (Projector)
   - 이는 물리적으로 저장소를 분리 시킬 때만 필요하다.
   - write 도메인 모델을 read 도메인 모델로 projection(투영)한다

### 장점

1. 높은 쓰기 처리량, 낮은 읽기 대기 시간 등 읽기 및 쓰기 작업의 복잡성을 처리하는 데 **개별적으로 적합한 리포지토리를 선택가능**
2. 관심사 분리와 더 간단한 도메인 모델을 제공 ⇒ 분산 아키텍처에서 **이벤트 기반 프로그래밍 모델을 자연스럽게 보완**
3. 읽기, 쓰기를 모두 지원하는 복잡한 도메인 모델을 보다 간결하게 나눌 수 있음
4. 도메인별로 로깅을 남길 수 있음 (Write Repository)

### 단점

1. 복잡성 : CQRS의 기본 아이디어는 간단하지만, 만약 이벤트 소싱 패턴을 포함할 경우, 더 복잡해질 수 있다.
2. 메세징 : 메세징이 CQRS의 필수요소는 아니지만, 명령(Command)을 수행하고 업데이트 이벤트를 발행하는 것이 보편적인 사용법이다. 이 경우 어플리케이션은 반드시 메세지 실패나 중복 메세지와 같은 것들에 대한 처리를 해야 한다.
3. **데이터 일관성 : 실시간 일관성 보장 불가, 최종 일관성은 보장**
   1. 데이터는 언젠가는 다 맞춰진다!
   2. 만약 읽기/쓰기 저장소가 분리된다면, 읽기 데이터가 최신의 데이터가 아닐 수도 있다. 읽기 저장소에는 쓰기 저장소의 변경 사항들이 반영되어야 하는데, 이에는 딜레이가 생기기 때문이다.
4. ORM 툴을 통해 DB 스키마로부터 자동으로 생성되도록 할 수는 없다.
5. 코드중복 발생

### 언제 사용해야 하는가?

- 프로세스나 도메인 모델이 복잡한 경우
- 읽기의 성능이 쓰기의 성능과 별도로 조정이 가능해야 할 때.
  - 특히 읽기의 수가 쓰기의 수보다 훨씬 많을 때.
  - 이 경우, 읽기 모델은 스케일 아웃을 하고, 쓰기 모델은 적은 수로 유지할 수 있으며, 적은 수의 쓰기 모델은 병향 충돌의 가능성을 최소화해준다.
- 시스템 계속 변화, 다양한 버전 존재 가능
- 다른 시스템과의 통합 (특히 이벤트 소싱)할 때, 서브 시스템의 장애가 다른 시스템에게 영향을 줘서는 안될 때
- 팀별로 쓰기 / 읽기 모델에 따로 집중해야 할 때

# 4. 이벤트 소싱

![Untitled](./트랜잭션%20스크립트%20패턴%20vs%20도메인%20모델%20패턴/Untitled%208.jpeg)

**상태를 업데이트하는 대신 이벤트를 생성한다**

**도메인에서 발생하는 “모든 사실”들을 “순차적”으로 기록하는 패턴이다.**

### 특징

도메인 객체에 대한 모든 변경을 이벤트로 관리

이 이벤트들은 시간적 순서를 가지며, 변경할 수 없음

저장소를 정렬된 도메인 이벤트 목록을 저장하도록 변경

여기서 이벤트가 얼마나 세분화되는지는 도메인 설계의 영역

**CQRS는 읽기에 최적화된 패턴이라면, 이벤트소싱은 쓰기에 적합한 패턴이라고 할 수 있다.**

### Projection 방법

1. Aggregate 에서 업데이트할 이벤트들을 대응할 Query Entity를 만든다
2. Aggregate 내에서 이벤트가 발생할 때마다 Query Entity를 업데이트 하여 데이터의 최종상태 유지

이 과정에서 동시성 문제 발생 가능하며 , Disk I/O로 인한 성능 저하가 발생할 수 있다. (캐싱 처리를 통해 개선해야 함, 예를 들어 write back & scheduling이면 쓰기 쿼리 회수 비용 부하를 줄일 수 있을 것이다! )

### 주의할 점

1. 상태를 저장한 후에 로그를 기록하는 것이 아니고 어떠한 변화를 나타내는 이벤트를 저장소에 기록하고 이 일련의 이벤트를 재생해서 상태를 만들어 내는 것이다.
2. 도메인이 Command를 받으면 그 결과로 이벤트를 발생시키고 그것을 이벤트 저장소에 저장, 이벤트 핸들러를 통해 상태로 변환
3. 영속대상은 이벤트이며 이벤트는 절대 삭제되고 수정되지 않고 오직 추가 되기만 한다.
4. 도메인 오브젝트 하나당 도메인 이벤트 스트림이 존재한다.
5. CQRS에서 이벤트 소싱을 적용하면서 Command가 Event로 바뀌었다. 여기서 Command와 Event는 같은 메시지이지만 성격이 다르다. **Command는 검증의 대상이지만 Event는 그 자체로 이미 벌어진 사실이며 검증 대상이 아니다.**
6. 이벤트 조회 과정에서의 성능 이슈를 해결해야 한다.

   백 만개의 이벤트를 가지는 도메인 이벤트라면? 전통적인 시스템과는 다르게 성능 이슈가 발생할 수 있다.(백 만개의 데이터를 조회해야하기 때문에) 그래서 스냅샷이라는 버저닝을 진행한다. 스냅샷은 계속 추가되는 것이 아니고 갱신 되는 것이다. 이것을 롤링 스냅샷이라고 부른다. ⇒ 이후 추가 공부 예정

7. 이벤트 방식은 순서가 보장되지 않을 수 있다. 이를 해결할 방법을 고민해야 한다.

### 방법

```java
public abstract class Event {
    public final UUID id = UUID.randomUUID();
    public final Date created = new Date();
}

public class UserCreatedEvent extends Event {
    private String userId;
    private String firstName;
    private String lastName;
}

public class UserContactAddedEvent extends Event {
    private String contactType;
    private String contactDetails;
}

public class UserContactRemovedEvent extends Event {
    private String contactType;
    private String contactDetails;
}

// ...
```

### 저장소 사용

보통 이벤트 소싱을 처리하기 위한 범용 분산 데이터 저장소를 사용 (Kafka, Cassandra, …)

### 장점

1. 쓰기는 이벤트를 저장하는 단계밖에 없어 작업 속도가 빠르다
2. 컴포넌트간 결합도를 낮추어 MSA 구조에 적합하다

### 단점

1. 학습곡선 & 직관적이지 않음
2. **상태를 로컬캐시에 저장해두어야 할 수 있음**
3. 이벤트 중심 아키텍처에 적합, 모든 도메일 모델에 적용할 수는 없음

### 예시 코드

https://www.baeldung.com/cqrs-event-sourcing-java

# 5. 참고하면 좋을 영상

## [**[우아콘2020] 배달의민족 마이크로서비스 여행기**](https://www.youtube.com/watch?v=BnS6343GTkY&t=2007s)

배달의 민족 시스템은 거대한 CQRS

성능이 중요한 외부 시스템 (Query)과 비즈니스 명령이 많은 내부 시스템 (Command)으로 분리

이벤트 발행을 통한 최종적 일관성 보장

각 시스템은 API 혹은 이벤트 방식으로 연동한다

### 이벤트 전파 아키텍처

![Untitled](./트랜잭션%20스크립트%20패턴%20vs%20도메인%20모델%20패턴/Untitled%203.png)

[여기서 나오는 SNS, SQS란?](https://btcd.tistory.com/98)

AWS Managed Kafka가 없을 때 사용했던 아키텍처

| SNS                                                                                                                                                              | SQS                                                                                                                                 |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Simple Notification Service                                                                                                                                      | Simple Queue Service                                                                                                                |
| Publisher(게시자)가 Subscriber(구독자)에게 메세지를 전송하는 관리형 서비스                                                                                       | 마이크로서비스, 분산 시스템 및 서버리스 애플리케이션을 쉽게 분리하고 확장할 수 있도록 지원하는 완전관리형 메세지 대기열 서비스      |
| Publisher는 Topic(주제)에 메세지를 발행한다. Topic은 수많은 Subscribers(구독자들)에게 전달될 수 있다.(fan out)이때 전달 방식은 다양하다.(Lambda, SQS, Email ...) | 시스템은 Queue로부터 새로운 이벤트를 실시할 수 있다.Queue에 있는 메세지들은 한 명의 consumer(고객) 또는 하나의 서비스에서 실행된다. |
| 다른 시스템들이 이벤트에 신경쓰는가?Topic에 메세지를 publish(발행)하고 싶어하고, 사람들에게 발행되었다고 알리고 싶을 때                                          | 이 시스템이 이벤트에 신경쓰는가?내가 이벤트의 수신자일 때                                                                           |

### ZERO-PAYLOAD

![Untitled](./트랜잭션%20스크립트%20패턴%20vs%20도메인%20모델%20패턴/Untitled%204.png)

데이터 동기화를 위해 메세지 전파 방식으로 ZERO-PAYLOAD 방식을 사용

**방식**

1. 이벤트에 식별자와 최소한의 정보만 발행
2. 이벤트를 받은 시점에 두 시스템간에 미리 협의한 조회 API로 필요한 데이터를 조회해서 저장

**이유 : 이벤트 순서 보장 문제를 해결**

1. 변경된 데이터만 보내게 되면 순서를 맞추기 위한 많은 고민 필요
2. 전체 데이터를 모두 보내버리면 너무 페이로드가 무거워짐

### CQRS QueryModel

![Untitled](./트랜잭션%20스크립트%20패턴%20vs%20도메인%20모델%20패턴/Untitled%205.png)

KEY : VALUE 형태

key를 가게 아이디, 값을 가게 노출 시에 보여줘야 하는 모든 데이터들을 미리 만들어둠

가게 아이디만 넣으면 가게 상세화면을 그리는데 필요한 데이터를 한번에 보내줄 수 있도록 해둠

![Untitled](./트랜잭션%20스크립트%20패턴%20vs%20도메인%20모델%20패턴/Untitled%206.png)

![Untitled](./트랜잭션%20스크립트%20패턴%20vs%20도메인%20모델%20패턴/Untitled%207.png)

### 기타

- 적극적인 캐시 사용
- 서킷 브레이커
- 비동기 Non-Blocking 시스템 적용 (Spring Webflux) : 가게노출, 광고리스팅, 검색

- API 활용
  - 주문 완료 직전에 메뉴 시스템에 API로 메뉴에 대한 Validation을 한다. (모든걸 이벤트로만 해결할 수는 없다)
  - 그러나 만약 이 API가 Timeout이 나거나 메뉴 시스템이 다운된다면 어떻게 할 것인가?
  - 개발과 기획등이 같이 고민. 그런 경우는 거의 없고 고객에게 치명적인 피해가 가지 않음으로 이런 경우는 회사 차원에서 보상을 하는 시스템으로 구성하였다.

### 참조

**클린코드 6. 객체와 자료구조**

[두 패턴의 차이](https://velog.io/@hellojihyoung/Design-Pattern-Domain-Model-Pattern-vs-Transaction-Script-Pattern)

[트랜잭션스크립트패턴을써야하는이유](https://chem-en-9273.tistory.com/135)

[김영한강사님답변](https://www.inflearn.com/questions/117315/%EB%B9%84%EC%A7%80%EB%8B%88%EC%8A%A4-%EB%A1%9C%EC%A7%81%EA%B5%AC%ED%98%84-entity-vs-service)

[OSIV관련](https://velog.io/@haron/Spring-Connection-Pool-%EC%9D%B4-%EB%B6%80%EC%A1%B1%ED%95%98%EB%8B%A4%EA%B3%A0%EC%9A%94)

[OSIV관련2](https://f-lab.kr/insight/understanding-osiv-in-spring-boot)

[OSIV관련3](https://dodeon.gitbook.io/study/kimyounghan-spring-boot-and-jpa-optimization/04-osiv)

[CQRS](https://mslim8803.tistory.com/73)

[CQRS2](https://www.baeldung.com/cqrs-event-sourcing-java)

[CQRS3](https://velog.io/@everyhannn/CQRS-Event-Sourcing)
