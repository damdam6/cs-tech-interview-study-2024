# JPA & DDD Entity 차이

태그: DDD, JPA
날짜: 2024년 6월 14일

# JPA Entity & DDD Entity 차이

## DDD 란 무엇인가? (간결히만 설명)

해당 부분 정리해서 공부할 예정입니다.

[https://learn.microsoft.com/ko-kr/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/microservice-domain-model#the-domain-entity-pattern](https://learn.microsoft.com/ko-kr/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/microservice-domain-model#the-domain-entity-pattern)

### [김영한 강사님 설명](https://www.inflearn.com/questions/148726/인터페이스와-구현체의-패키지-관련-질문입니다)

- hello.web
- hello.domain
- hello.infrastructure

Domain에 Repository의 인터페이스를 두고, 그 구현은 infrastructure라는 전혀 다른 패키지에 두는 것이지요.

### 참조

[https://learn.microsoft.com/ko-kr/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/ddd-oriented-microservice](https://learn.microsoft.com/ko-kr/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/ddd-oriented-microservice)

## JPA & DDD Entity는 서로 같은걸까? (부제 : Entity와 VO, DTO와의 차이)

**결론부터 말하면 다르다.**

[이 블로그 글을 기준으로 공부하였습니다.](https://namocom.tistory.com/980)

<만들면서 배우는 클린 아키텍처 2장>

![Untitled](JPA%20&%20DDD%20Entity%20차이/Untitled.png)

도메인에서 Entity와 JPA 같은 ORM-Managed Entity 가 명확하게 분리되어 있음을 알 수 있다.

- DDD에서의 Entity:
  - 비즈니스 도메인 관점에서 문제를 해결하기 위한 객체
  - 엔티티의 정체성(identity)는 ID로 표현
- JPA에서의 Entity:
  - javax.persistence 패키지의 @Entity라는 애너테이션이 붙어있음
  - Persistence, 즉 데이터베이스와 매핑이 되는 객체
  - 식별이 가능한 키, 즉 @Id 필드가 필요 (여기서 ID는 PK)
- **여기서 DDD 엔티티의 ID가 반드시 PK가 되어야 하는가? 아니다**

### <로버트 C. 마틴의 클린 아키텍처>

> From an architectural point of view, the
>
> **database is a non-entity**
>
> —it is a detail that does not rise to the level of an architectural element. Its relationship to the architecture of a software system is rather like the relationship of a doorknob to the architecture of your home.
>
> **아키텍처 관점에서 볼 때 데이터베이스는 엔티티가 아니다.**
>
> 즉, 데이터베이스는 세부사항이라서 아키텍처의 구성요소 수준으로 끌어올릴 수 없다. 소프트웨어 시스템의 아키텍처와 데이터베이스의 관계를 건물로 비교하면 건물의 아키텍처와 문 손잡이의 관계와 같다.

### 실전에서는?

**보통 JPA 엔티티를 Repository와 맞닿아 있는 Service까지로 하기로 타협을 한다.**

ID라는 비슷한 개념을 가지고 있기에 JPA의 엔티티가 비즈니스 로직에 침범하는 경우가 많다.

심지어 Repository를 넘어, Service 심지어 Controller까지 JPA의 엔티티가 와서 사용되는 경우를 보았다.

**그러나 Service 경계를 넘어서는 JPA의 엔티티가 아닌 일반 POJO 객체가 움직여야 한다.** POJO를 세부적으로 나누어보면 구분할 수 있는 ID가 있었으므로 (DB의 PK랑은 같은 경우도 다른 경우도 있었다) VO보다는 DDD의 Entity라고 할 수 있다.

### 참고 자료 : [스택 오버플로우](https://stackoverflow.com/questions/51564704/ddd-entity-in-orm-and-java)

질문 1: Hibernate의 OrderLine은 Entity이지만, OrderLine이 여전히 DDD 엔티티인가요?
답변 1: 아니요, Hibernate 엔티티는 절대 DDD 엔티티가 되어서는 안 됩니다.

질문 2: 일반적으로 JPA/Hibernate의 @Entity가 DDD 엔티티라고 말할 수 있나요, 아니면 그렇지 않나요?
답변 2: 아니요, JPA/Hibernate 엔티티는 절대 DDD 엔티티가 되어서는 안 됩니다.

질문 3: OrderLine은 JPA/Hibernate 엔티티이지만 DDD 엔티티는 아닌 것 같습니다. 맞나요?
답변 3: 맞습니다.

질문 4: 관계형 데이터베이스 관점에서 보면, JPA @Entity 중 데이터베이스에서 OnDeleteCascade로 매핑된 것은 DDD 엔티티가 아니라 VO일까요?
답변 4: 아니요, JPA 엔티티/값 객체와 DDD 객체 사이에는 직접적인 관련이 없습니다.

질문 5: Hibernate @Embedded는 항상 DDD 값 객체인가요? (아이덴티티가 없어 보이므로 그렇게 보입니다)
답변 5: 아니요, 관련이 없습니다.

질문 6: BillingCustomer와 MarketingCustomer가 동일한 DDD 엔티티(John Doe, 1980년 1월 1일 출생, SSN 12-34-56)를 나타낸다고 말할 수 있나요? 이는 자바에서 부모 클래스(Object 제외)가 공통되지 않는 두 개의 다른 클래스가 동일한 DDD 엔티티를 나타낼 수 있다는 것을 의미합니다. 그렇다면 위 클래스에서 equals를 어떻게 구현해야 할까요?
답변 6: 아니요, 객체의 ID는 순수한 개념적인 것을 참조할 수 있습니다. BillingCustomer와 MarketingCustomer는 개념적으로 다른 것이므로, 실제 인간이 동일하더라도 절대 같지 않을 것입니다. 대신 matches(), sameCustomer() 등의 새로운 관계를 정의해보세요.

질문 7: 서로 다른 자바 클래스가 동일한 DDD 엔티티를 나타내는 경우, Java의 equals()가 true를 반환해야 할까요?
답변 7: 아니요, equals()도 일반적인 자바 규칙을 따라야 합니다. 다른 클래스의 객체는 절대 같지 않아야 합니다.

## +a) [Entity와 VO의 차이](https://jaeyeolshin.github.io/2016-02-06/difference-between-entity-and-value-object/) (여기서 엔티티는 데이터베이스 엔티티가 아닌 DDD 엔티티)

- 엔티티(Entity)의 정체성은 id로 “ unique하게 식별”되고 속성이 변할 수 있는 “가변객체”이다.
  - 일반적으로 사람, 제품, 파일, 판매 등이다.
- 값 객체(VO)는 “불변 객체”이다.

  - 예시로 어떠한 사람이 위치가 변한다고 위치 객체를 변경하지 않는다. 새로운 위치 객체를 만든다.
  - 일반적으로 위치, 날짜, 숫자, 금액 등이다.
  - 그런데 만약 위치 자체가 중요해져 많은 사람들이 같은 위치에서 체크인을 하게 되는 서비스라면? 위치가 더이상 VO가 아닌 Entity가 된다.

- 왜 구분해야 하는가?
  - 두 개의 속성이 같은 엔티티는 ID가 다름으로 다른 객체이다. 하지만 같은 속성을 가지는 VO는 같은 객체이다. (equals() 판단시 중요)
    - 같은 객체라면 자유롭게 서로 대체할 수 있으나 엔티티는 그렇게 하면 안될것이다.
  - 엔티티는 속성이 변해도 같은 엔티티이다. 하지만 VO는 속성이 변한다면 항상 다른 VO 이다.
  - 이는 도메인 주고 설계 (DDD)에서 중요한 개념이다.
  - 언뜻 보았을 때 어떤 객체가 식별자를 가져야 한다고 무작정 그걸 엔티티로 만들면 안된다. VO와 엔티티를 구분하는 데 있어서는 어플리케이션에 대한 완벽한 이해를 바탕으로 한다.

아래는 해당 블로그에 달린 댓글이다. 여기서 댓글을 단 사람은 JPA 엔티티와 블로그 필자가 쓴 DDD 엔티티를 혼동하여 아래와 같은 질문을 한 것으로 보인다.

![Untitled](JPA%20&%20DDD%20Entity%20차이/Untitled%201.png)

### 참조

[https://namocom.tistory.com/980](https://namocom.tistory.com/980)

사례 : [https://applepick.tistory.com/153](https://applepick.tistory.com/153)

[https://wlswoo.tistory.com/67](https://wlswoo.tistory.com/67)

### [ORM Entity vs POJO Entity](https://www.inflearn.com/questions/458968/jpa와-ddd에-대하여-질문-드립니다)

질문

안녕하세요. 공부하는 개발자입니다.
도메인 주도 설계를 바탕으로 설계를 해보려고 하는 도중에 궁금한것이 있어 질문 드립니다.
데이터를 영속화하기 위해 사용하는 ORM Entity를 하나의 도메인으로 바라보고 설계를하니 너무 관계형 데이터베이스 모델링에 의존적인 설계가 되어버리는것 같습니다. 마치 도메인 주도 설계가 아니라 ERD 주도 설계를 하는 느낌이 듭니다.
도메인 주도 설계에서 말하는 Domain과 데이터를 영속화하기 위해 사용하는 ORM(JPA)Entity를 분리하여 ORM Entity는 RDB의 모델링에 맞게 설계하여 데이터베이스와의 작업을 수행하고, 실제 핵심 비즈니스 로직들은 ORM Entity가 아니라 POJO로 만들어진 Domain이 수행하는게 좀 더 DDD에 맞는 개발이 아닌가 하는 생각이 듭니다.
영한님의 생각은 어떠한지, 실무에서는 어떤식으로 설계를 하는지 궁금해서 질문 남깁니다.
감사합니다.

김영한 강사님 답변

안녕하세요. 이선제님

이 부분은 정답이 있다기 보다는 각각 트레이드 오프가 있기 때문에 프로젝트에 맞는 선택을 하는 것이 필요합니다.

둘을 나누면서 발생하는 이점도 있지만, 막상 개발을 해보면 비슷한 엔티티를 둘로 똑같이 개발하고 있을 수도 있습니다. 그래서 프로젝트 상황에 따라서 각각의 트레이드 오프를 고민하고 선택해야 합니다.

## 그러면 DTO와 Entity, VO의 차이는?

### DTO (Data Transfer Object)

- Entity : DB 테이블과 1:1로 매핑되는 클래스
  - 상속 받는 구현체거나 테이블 내에 존재하지 않는 칼럼을 가져서는 안됨
  - 테이블과 매핑되어있기 때문에 변경되어서는 안되며, 이런 이유로 DTO와 구분되어야 함
- DTO
  - Layer간의 데이터 교환을 위한 자바 객체 (여러 Layer간 이동이지만, 특히 DB에서 View)
  - **비즈니스 로직이 없는 순수한 데이터 객체로, Getter, Setter 메서드만 가짐 (데이터 “전달”만이 목적)**
  - **가변**의 성격을 가짐
  - 필드값이 같아도 객체는 동일하지 않다

### VO (Value Object)

- 값 오브젝트로서 **불변!**
- **객체의 불변성을 보장하여 Read만 가능 (Setter X)**
- **equals, hashcode 재정의하여 객체의 동일성 보장해야**
- 고정된 값에 VO 사용하면 됨

### [만약 Repository, DAO의 정확한 의미적 차이를 알고싶다면?](https://bperhaps.tistory.com/entry/Repository%EC%99%80-Dao%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90)

요역하면.

**현재는 DAO나 Repository나 같다고 봐도 무방하다.**

현업의 코드들도 어쩌면 JpaRepository를 상속했다는 이유로 xxxRepository라고 명명한 것일 수도 있다.

필자는 DAO를 사용하더라도 도메인 레이어에 속하는 Repository와 같은 interface를 이용하여 구현하면(데코레이팅) 충분히 Repository처럼 사용할 수 있다고 생각한다고 한다.

### 그러나 설계적인 의미의 차이를 알고싶다면 아래를 보자.

Presentation - Application - Domain - Infrastructure

- Domain은 결국 **인터페이스이다.**
- Infrastructure는 보통 영속성 레이어이며 DB, 외부 라이브러리 들과 연결되는 계층을 말한다.

### DAO

DAO 는 기존에 어플리케이션이 영구저장소에 접근하는 방법의 문제를 해결하기 위해 나왔다.

기존에는 각 영구 저장소 밴더에서 제공하는 API를 통해서 접근했었다. 그러나 이 방법은 구현체롸 로직이 너무 강한 결합을 갖게 하고, 인프라와 응용계층 (Application) 이 섞여버리는 문제를 가지게 한다.

DAO는 Infrastructure 밴더와 API 로직 사이의 어댑터 역할을 한다. 벤더 구현체를 그대로 사용하는 것이 아니라 DAO라는 객체로 한번 더 감쌈으로서 내가 사용하는 데이터 소스가 변경되더라도 로직에 변화가 없도록 함.

DAO는 자신이 영속성 객체라는 것을 숨기지 않는다. 이름에서 그 의도를 바로 나타낸다. 자신이 인프라 스트럭쳐 레이어에 있다는 걸 숨기지 않는다. 또한 계층(모듈)의 의존성을 봤을 때 도메인이 인프라 스트럭쳐에 의존한다. 따라서 DAO를 이용해 바로 entity를 컨트롤 하는것은 계층을 무너트리고 잘못된 사용이다. 항상 DTO를 사용하여 데이터를 주고받아야 한다(같은 이유로 DTO에 Entity를 넣어서 보내는 행위 또한 안된다.)

### Repository

Repository는 DDD에서 처음 등장한 개념으로 **Domain 계층에 속해있다.** Repository는 객체의 상태를 관리하는 저장소임으로 엔티티 그 자체를 저장하고 불러오는 역할을 한다. 당연하게 Repository는 도메인 레이어에 대한 지식을 알고 있어야 한다.

여기서 혼동하지 말아야 할 것은 **Repository는 영구저장소를 의미하는게 아니다 (Infrastructure 단이 아니다).** 말 그대로, **객체의 상태를 관리하는 저장소**일 뿐이다. 즉, Repository의 구현이 파일시스템으로 되든, 아니면 HashMap으로 구현됐든지 상관없다. 그냥 객체(entity)에 대한 CRUD를 수행할 수 있으면 된다.

클라이언트는 Repository의 인터페이스만을 사용한다. 따라서 그 구현을 모른다. DAO와 같다. 구현을 숨김으로써 로직을 한곳으로 응집시켰다(캡슐화). 따라서 서비스 로직에서는 "도메인"의 레포지토리(컬렉션)을 사용한 것 뿐이다. 영속성과 관련된 로직은 전혀 모른다. 또한 인프라 스트럭처와 관련된 모듈 또한 전혀 import되지 않는다. 따라서 Repository가 실제로 바인딩 되어있는 콘크리트 객체는 인프라 스트럭쳐 레이어에 속해있고, Repository는 도메인 계층에 속해있음으로 계층이 깨지는 일은 없다.

**Repository의 Interface는 도메인 레이어,  Repository의 구현체는 영속성 레이어(Infrastructure)에 속한다. 이를 통해 도메인 레이어와 인프라스트럭처 레이어의 의존성이 뒤집힌다(DIP). 따라서 이젠 인프라스트럭처 레이어에서 도메인의 지식을 알아도 괜찮으며 이를 통해 Entity를 영속성 로직에 포함시킬 수 있게 되는 것 이다.**

Repository는 자신이 영속성 객체임을 숨긴다. 자신이 인프라 스트럭처 레이어에 있다는 것을 숨긴다. 또한 의존성이 역전되어 있기 때문에, entity를 그대로 가져와 영속성 로직을 수행하는 것을 가능하게 한다.
