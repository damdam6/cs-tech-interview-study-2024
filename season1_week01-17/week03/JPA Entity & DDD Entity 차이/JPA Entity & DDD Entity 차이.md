# [스터디 발표] JPA Entity & DDD Entity 차이

태그: DDD, JPA, 스터디
날짜: 2024년 6월 14일

# JPA Entity & DDD Entity 차이

# 1. DDD (Domain-Driven Development) 이란

- 도메인 그 자체와 도메인 내 로직에 집중하는 개발 방식
  - 혹은 비즈니스 Domain별로 나누어 설계하는 방법
  - **여기서 Domain은 유사한 업무의 집합으로 “비즈니스” Domain을 의미한다!**
- **보편적인 언어 추구**
  - 비즈니스 용어(유비쿼터스 언어)를 사용하는 것이 중요
  - 도메인 전문가와 소프트웨어 개발자 간의 커뮤니케이션 문제를 없애는 것이 중요
  - 상호가 이해할 수 있고 모든 문서와 코드가 동일한 표현과 단어로 구성된 언어체계 구축이 목적
- 소프트웨어 엔티티와 도메인 컨셉트를 가장 가까이 일치 시킨다
- 목표
  - Loosly coupling
    - ~~Object Reference~~, ID Reference
  - High cohesion
- 레이어드 아키텍처 패턴 & DDD

![Untitled](./JPA%20Entity%20&%20DDD%20Entity%20차이/Untitled.png)

- 도메인 레이어가 repository, Infrastructure가 repositoryImpl?
  - **아니다. Infrastructure는 JDBC 등 실제로 DB와 접근하는 구현체 기술을 의미한다.**
  - 예시
    ```java
    public class JdbcUserRepositoryImpl implements UserRepository {
        private final DataSource dataSource;

        public JdbcUserRepositoryImpl(DataSource dataSource) {
            this.dataSource = dataSource;
        }

        @Override
        public User findById(Long id) {
            String sql = "SELECT * FROM users WHERE id = ?";
            try (Connection connection = dataSource.getConnection();
                 PreparedStatement statement = connection.prepareStatement(sql)) {
                statement.setLong(1, id);
                ResultSet resultSet = statement.executeQuery();
                if (resultSet.next()) {
                    return mapResultSetToUser(resultSet);
                } else {
                    return null;
                }
            } catch (SQLException e) {
                throw new RuntimeException("Error finding user by id", e);
            }
        }

        @Override
        public void save(User user) {
            String sql = "INSERT INTO users (name, email) VALUES (?, ?)";
            try (Connection connection = dataSource.getConnection();
                 PreparedStatement statement = connection.prepareStatement(sql)) {
                statement.setString(1, user.getName());
                statement.setString(2, user.getEmail());
                statement.executeUpdate();
            } catch (SQLException e) {
                throw new RuntimeException("Error saving user", e);
            }
        }

        private User mapResultSetToUser(ResultSet resultSet) throws SQLException {
            Long id = resultSet.getLong("id");
            String name = resultSet.getString("name");
            String email = resultSet.getString("email");
            return new User(id, name, email);
        }
    }

    ```

### 도메인 엔티티란?

1. 비즈니스 로직을 들고 있고
2. 식별 가능하며
3. 일반적으로 생명주기를 가짐

- 예시
  ```java
  @Getter
  public class User {

      private final Long id;
      private final String email;
      private final String nickname;
      private final String address;
      private final String certificationCode;
      private final UserStatus status;
      private final Long lastLoginAt;

      @Builder
      public User(Long id, String email, String nickname, String address, String certificationCode, UserStatus status, Long lastLoginAt) {
          this.id = id;
          this.email = email;
          this.nickname = nickname;
          this.address = address;
          this.certificationCode = certificationCode;
          this.status = status;
          this.lastLoginAt = lastLoginAt;
      }

      public static User from(UserCreate userCreate, UuidHolder uuidHolder) {
          return User.builder()
              .email(userCreate.getEmail())
              .nickname(userCreate.getNickname())
              .address(userCreate.getAddress())
              .status(UserStatus.PENDING)
              .certificationCode(uuidHolder.random())
              .build();
      }

      public User update(UserUpdate userUpdate) {
          return User.builder()
              .id(id)
              .email(email)
              .nickname(userUpdate.getNickname())
              .address(userUpdate.getAddress())
              .certificationCode(certificationCode)
              .status(status)
              .lastLoginAt(lastLoginAt)
              .build();
      }

      public User login(ClockHolder clockHolder) {
          return User.builder()
              .id(id)
              .email(email)
              .nickname(nickname)
              .address(address)
              .certificationCode(certificationCode)
              .status(status)
              .lastLoginAt(clockHolder.millis())
              .build();
      }

      public User certificate(String certificationCode) {
          if (!this.certificationCode.equals(certificationCode)) {
              throw new CertificationCodeNotMatchedException();
          }
          return User.builder()
              .id(id)
              .email(email)
              .nickname(nickname)
              .address(address)
              .certificationCode(certificationCode)
              .status(UserStatus.ACTIVE)
              .lastLoginAt(lastLoginAt)
              .build();
      }
  }
  ```

### 영속성 엔티티란?

1. (영속성) Entity : DB 테이블과 1:1로 매핑되는 클래스
2. 상속 받는 구현체거나 테이블 내에 존재하지 않는 칼럼을 가져서는 안됨
3. JPA @Entity

- 예시
  **UserEntity → User : `public User toModel()`**
  **User → UserEntity : `public static UserEntity fromModel(User user)`**
  ```java
  @Getter
  @Setter
  @Entity
  @Table(name = "users")
  public class UserEntity {

      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private Long id;

      @Column(name = "email")
      private String email;

      @Column(name = "nickname")
      private String nickname;

      @Column(name = "address")
      private String address;

      @Column(name = "certification_code")
      private String certificationCode;

      @Column(name = "status")
      @Enumerated(EnumType.STRING)
      private UserStatus status;

      @Column(name = "last_login_at")
      private Long lastLoginAt;

      public static UserEntity from(User user) {
          UserEntity userEntity = new UserEntity();
          userEntity.id = user.getId();
          userEntity.email = user.getEmail();
          userEntity.nickname = user.getNickname();
          userEntity.address = user.getAddress();
          userEntity.certificationCode = user.getCertificationCode();
          userEntity.status = user.getStatus();
          userEntity.lastLoginAt = user.getLastLoginAt();
          return userEntity;
      }

      public User toModel() {
          return User.builder()
              .id(id)
              .email(email)
              .nickname(nickname)
              .address(address)
              .certificationCode(certificationCode)
              .status(status)
              .lastLoginAt(lastLoginAt)
              .build();
      }
  }
  ```

### 도메인 객체란?

**DDD의 보편적인 용어로, 도메인 모델을 구성하는 모든 객체를 의미**

**Entity, VO, Aggregate Root, Service 객체 (→ 이건 Application Layer에 속하는것 아닌가..??) 등이 포함된다고 한다.**

**DDD 엔티티는 도메인 객체의 한 종류라고 볼 수 있다.** DDD 엔티티는 고유한 식별자를 가지고, 시간에 따라 변화하는 도메인 객체를 나타낸다.

**다만, 모든 도메인 객체가 엔티티는 아니다.** 벨류 오브젝트나 서비스 객체와 같은 다른 종류의 도메인 객체도 있다. **하지만 일반적으로 DDD 엔티티와 도메인 객체는 뭉뚱그려서 동의어로 사용된다.**

![Untitled](./JPA%20Entity%20&%20DDD%20Entity%20차이/Untitled%201.png)

### Aggregate Pattern이란?

엔티티를 Aggregate root에 바인딩하여 다양한 엔티티를 논리적으로 그룹화하는 것

- 연관된 Domain Entity와 VO의 묶음 단위 ⇒ 일관성과 트랜잭션, 분산의 단위
- **한 개의 루트 엔티티와 기타 엔터티+VO 로 구성된다**
- [실제로는 매우 작게, 1개의 애그리게이트당 1개의 엔티티 정도가 들어가도록 작게 구성해야 한다고 한다.](https://www.inflearn.com/questions/27918/%EB%8F%84%EB%A9%94%EC%9D%B8%EC%84%A4%EA%B3%84-%EA%B4%80%EB%A0%A8-%EC%A7%88%EB%AC%B8%EB%93%9C%EB%A6%BD%EB%8B%88%EB%8B%A4)
- 애그리게이트 루트 : 에그리게이트가 제공해야할 핵심 도메인 기능을 보유한 모델
- Entity
  - 테이블 모델, 고유 식별자가 있다
  - 독립적으로 식별될 수 있다
- VO

  - 데이터 표현 모델 식별자가 없고 **불변 객체**이다
  - 값으로만 표현되는 값으로 값이 같다면 같은 객체로 간주된다
  - 식별자가 아닌 속성에 따라 비교된다

- **예시를 들어보자**

  **Aggregate (주문)**

  - **Entity (주문): ⇒ ROOT**
    - ID: 12345
    - 날짜: 2024년 3월 1일
  - **Entity (상품):**
    - ID: 67890
    - 이름: "책"
    - 가격: $20
    - 주소 : `배송 주소`
    - **VO (배송 주소):**
      - 도시: "서울"
      - 우편번호: "12345"

- DDD 설계과정에 대해 더 자세한 내용은 [여기](https://www.notion.so/Domain-Driven-Design-068b1a9ebb1d4e5ba3543b39efbebbbe?pvs=21)와 [여기](https://velog.io/@injoon2019/CS-Domain-Driven-Design)를 참고해주세요

### 이에 반하는 **데이터 주도 설계**란?

- 객체가 가져야 할 데이터에 초점을 두고 설계
- 따라서 **설계 시 협력에 고민을 하지 않아 객체가 어디서 쓰일지 모름 ⇒ 과도한 수정자와 접근자들이 탄생하게 된다.**
- 캡슐화 원칙 위반
- 내부 구현이 퍼블릭 인터페이스에 노출, 결합도 높아짐

### 참조

[https://learn.microsoft.com/ko-kr/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/ddd-oriented-microservice](https://learn.microsoft.com/ko-kr/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/ddd-oriented-microservice)

[https://incheol-jung.gitbook.io/docs/q-and-a/architecture/ddd](https://incheol-jung.gitbook.io/docs/q-and-a/architecture/ddd)

[\*\*https://happycloud-lee.tistory.com/94](https://happycloud-lee.tistory.com/94) ⇒ 글이 매우 좋음!\*\*

[**https://learn.microsoft.com/ko-kr/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/microservice-domain-model#the-domain-entity-pattern**](https://learn.microsoft.com/ko-kr/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/microservice-domain-model#the-domain-entity-pattern)

# 2. JPA & DDD Entity는 서로 같은걸까?

**결론부터 말하면 다르다. 그러나 이 둘을 크게 구분하지 않고 쓸 것인지에 여부는 프로젝트의 규모와 성격 등을 고려해야 한다.**

[이 블로그 글을 기준으로 공부하였습니다.](https://namocom.tistory.com/980)

<만들면서 배우는 클린 아키텍처 2장>

![Untitled](./JPA%20Entity%20&%20DDD%20Entity%20차이/Untitled%202.png)

도메인에서 Entity와 JPA 같은 ORM-Managed Entity 가 명확하게 분리되어 있음을 알 수 있다.

### 둘의 차이는?

- DDD에서의 Entity:
  - 비즈니스 도메인 관점에서 문제를 해결하기 위한 객체
  - 고유한 식별자인 ID를 가지고 있다.
  - 비즈니스 로직을 가지고 있다.
  - **도메인 모델에 집중하며, 데이터 저장소와의 관계는 고려하지 않는다.**
- JPA에서의 Entity:
  - javax.persistence 패키지의 @Entity라는 애너테이션이 붙어있음
  - Persistence, 즉 데이터베이스와 매핑이 되는 객체. 데이터 영속성을 위한 구현 기술
  - 식별이 가능한 키, 즉 @Id 필드가 필요 (여기서 ID는 PK)
  - **데이터베이스 테이블과의 매핑에 집중하며, 도메인 모델과의 관계는 고려하지 않는다.**
- **여기서 DDD 엔티티의 ID가 반드시 PK가 되어야 하는가? 아니다**

### 예시

```java
// DDD 엔티티
public class Order {
    private OrderId orderId;
    private Customer customer;
    private List<OrderItem> orderItems;

    public void addItem(OrderItem item) {
        orderItems.add(item);
        calculateTotalAmount();
    }

    private void calculateTotalAmount() {
        // 비즈니스 로직 구현
    }
}

// JPA 엔티티
@Entity
@Table(name = "orders")
public class OrderEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "customer_id")
    private CustomerEntity customer;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItemEntity> orderItems = new ArrayList<>();

    public void addItem(OrderItemEntity item) {
        orderItems.add(item);
        item.setOrder(this);
    }
}

```

위의 예시에서 `Order`는 DDD 엔티티로, 비즈니스 로직을 포함하고 있다. 여기서 Order라는 “보편적인” 이름을 가지고 있는 것을 알 수 있다.

반면 `OrderEntity`는 JPA 엔티티로, 데이터베이스 테이블과의 매핑을 위한 구현체다. 이 두 엔티티는 서로 다른 목적으로 사용되며, 필요에 따라 서로 변환되어 사용될 수 있다. 예를 들어, 애플리케이션 계층에서는 DDD 엔티티를 사용하고, 데이터 액세스 계층에서는 JPA 엔티티를 사용할 수 있다. 이를 통해 도메인 모델과 데이터 저장소 간의 관심사를 분리할 수 있다.

그러나 실제로는 두 엔티티가 동일한 형태로 사용되는 경우도 있으며, 굳이 둘을 분리하여 구현할 필요가 없는 경우도 많다. 비즈니스 상황에 따라서 잘 확인해야 할 것이다. 다른 글들도 많이 읽어봐야겠지만, **김영한 강사님은 실무에서는 둘을 분리하여 사용하지 않는 것을 추천한다고 한다. 자세한 이야기는 후술하겠다.**

- [이 질문을 참고해보자](https://www.inflearn.com/questions/90087/domain%EA%B3%BC-repository-%EA%B5%AC%ED%98%84-%EC%83%81%EC%84%B8%EB%A5%BC-%EB%B6%84%EB%A6%AC%ED%95%98%EA%B3%A0%EC%9E%90-%ED%95%A0-%EB%95%8C-entity-%EB%94%94%EC%9E%90%EC%9D%B8)
  이런 관점에서 질문 전체를 통과하는 답을 드리자면, **엔티티와 도메인 객체를 따로 분리하지 말고, 엔티티 자체를 도메인 객체로 사용하시는 것이 좋습니다.** xml로 매핑하면 정말 겉으로 보기에는 순수한, 의존성 없는 코드를 얻으실 수 있겠지만, 애노테이션 정도는 실용적인 관점에서 가지고 가는 것이지요.

### 그러면 DDD Entity 이면서 JPA Entity는 아닌 예시는 무엇이 있는가?

개인적으로 생각한 예시는, 비즈니스에서 중요한 엔티티이나 Database에 저장되지 않는 중간 객체라면, 이는 DDD 엔티티이나 JPA 엔티티는 아니라고 말할 수 있을 것 같다. (틀렸다면 지적 부탁!)

**가장 대표적으로는 Domain Event 객체가 있다. Factory 객체도 복잡한 객체 생성 로직을 캡슐화하면서 도메인 모델에 속하지만, 데이터베이스와 매핑되지 않는다.**

아래 코드를 예시로 보자, ShoppingCart와 Product는 DB에 저장되지만, CartItem이라는 엔티티는 DB에 저장되지 않으면서 장바구니 관련 비즈니스 로직을 가지고 있다.

_아래 CartItem이 왜 JPA 엔티티가 아니냐고 의문을 가진다면, 영속성 엔티티는 테이블과 완전히 1대1로 매핑되는 객체이다. 따라서 product_id와 quantity를 필드로 가지고 있는 테이블을 저장할 것인지를 생각해보면 답이 될 것이다._

```java
// DDD 엔티티
public class ShoppingCart {
    private CartId cartId;
    private Customer customer;
    private List<CartItem> items;

    public void addItem(CartItem item) {
        items.add(item);
        calculateTotalAmount();
    }

    private void calculateTotalAmount() {
        // 장바구니 총 금액 계산 로직
    }

    // 기타 장바구니 관련 비즈니스 로직
}

// 데이터베이스에 저장되지 않는 엔티티
public class CartItem {
    private Product product; // Product는 DB에 저장됨
    private int quantity;

    // 장바구니 항목 관련 비즈니스 로직
}

```

### 참고 자료 : [스택 오버플로우](https://stackoverflow.com/questions/51564704/ddd-entity-in-orm-and-java)

- 펼치기
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

위의 글에서도 볼 수 있듯이,

# 3. 실전에서는?

### 구분이 필요한가?

위에서 언급했듯, 김영한 강사님은 실무에서는 굳이 도메인 객체 (DDD 엔티티)와 JPA 엔티티를 구분할 필요가 없다고 주장했다. 그러나 아무래도 한국에서 JPA의 아버지이고 성능 최적화와 관련한 관점임을 유념해야 한다.

1. 만약 JPA 영속성 객체를 도메인 엔티티로써 사용한다면 더티 체킹, 양방향 편의 메서드, 쿼리 DTO 반환, fetch join 등의 성능 최적화의 장점을 사용할 수 있다.

다른 사람들의 의견은 다를 수 있다. 최근에 들은 **[Java/Spring 테스트를 추가하고 싶은 개발자들의 오답노트]** 강의에서 김우근 강사님은 도메인 엔티티와 JPA 엔티티를 분리해야한다고 주장했다.

1. 만약 DB를 RDBMS에서 DocumentDB(MongoDB)로 바꾼다면? 도메인 엔티티와 JPA 엔티티를 동일하게 사용했다면 불가능하다. 필드 및 용어도 다르거니와 모든 관련 코드를 수정해주어야 한다.
2. 테스트를 Mock, DB에 의존적이지 않게 만들 수 있다는 장점이 있다.
3. 그러나 이런 경우 JPA 더티체킹이 작동하지 않음으로 추가적으로 save 를 해주는 등을 고려해야 할 것이다.

결국 중요한 것은, 프로젝트의 성격과 성능, 아키텍처젹인 관점, 무엇보다 **“사내에서 합의된 방향”**이 제일 중요하다.

### [ORM Entity (JPA) vs POJO Entity (DDD)](https://www.inflearn.com/questions/458968/jpa와-ddd에-대하여-질문-드립니다)

질문

(중략)

데이터를 영속화하기 위해 사용하는 ORM Entity를 하나의 도메인으로 바라보고 설계를하니 너무 관계형 데이터베이스 모델링에 의존적인 설계가 되어버리는것 같습니다. 마치 도메인 주도 설계가 아니라 ERD 주도 설계를 하는 느낌이 듭니다.
**도메인 주도 설계에서 말하는 Domain과 데이터를 영속화하기 위해 사용하는 ORM(JPA)Entity를 분리하여 ORM Entity는 RDB의 모델링에 맞게 설계하여 데이터베이스와의 작업을 수행하고, 실제 핵심 비즈니스 로직들은 ORM Entity가 아니라 POJO로 만들어진 Domain이 수행하는게 좀 더 DDD에 맞는 개발이 아닌가 하는 생각이 듭니다.**
영한님의 생각은 어떠한지, 실무에서는 어떤식으로 설계를 하는지 궁금해서 질문 남깁니다.
감사합니다.

김영한 강사님 답변

**이 부분은 정답이 있다기 보다는 각각 트레이드 오프가 있기 때문에 프로젝트에 맞는 선택을 하는 것이 필요합니다.**

둘을 나누면서 발생하는 이점도 있지만, 막상 개발을 해보면 비슷한 엔티티를 둘로 똑같이 개발하고 있을 수도 있습니다. 그래서 프로젝트 상황에 따라서 각각의 트레이드 오프를 고민하고 선택해야 합니다.

### [김영한 강사님 JPA 강의를 보면 DDD 개념을 적용하지 않는다?](https://www.inflearn.com/questions/452168/ddd%EC%99%80-jpa)

예시로 Member 엔티티와 Team 엔티티를 아래처럼 서로 직접 연관관계로 묶는다면 애그리게이트간의 경계가 허물어지는 것 아닌가? 이렇게 **Team 엔티티에 대한 참조를 직접 가지는 것 보다 식별자인 String teamId을 가지는 것이 더 권장된다는 내용을 본 것 같았는데** 이와 관련하여 비슷한 질문을 하신 분이 있었다.

```java
@Entity
class Member{
	...
	@ManyToOne
	Team team; // String teamId; ???
}
```

**결론부터 이야기하면 김영한 JPA 강의는 DDD에 의거하여 설명한 것이 아니었다. 아래는 [김영한 강사님의 답변](https://www.inflearn.com/questions/27918/%EB%8F%84%EB%A9%94%EC%9D%B8%EC%84%A4%EA%B3%84-%EA%B4%80%EB%A0%A8-%EC%A7%88%EB%AC%B8%EB%93%9C%EB%A6%BD%EB%8B%88%EB%8B%A4)이다.**

**Q) 애그리거트 루트를 참조할때는 객체를 통해 접근하면 각 애그리거트간의 결합도가 상승할거같은데 어떤식으로 처리하시나요 ??**

A) DDD에서는 주로 이런 경우 연관관계를 맺지 말고, 식별자로 분리하라고 설명합니다. 이렇게 하면 다른 프로젝트여도 해당 식별자(String id, Long Id 등등)만 가지고 있으면 되기 때문에, 문제가 없습니다.

그런데, **이 모든 것은 프로젝트의 규모에 따른 선택이 필요합니다. 애그러거트 개념을 통해서 애그리거트 단위로 트랜잭션을 만들고 전파하고, 이런 부분은 다른 방면의 복잡성을 매우 높입니다. 추가로 연관관계를 매핑할 때 참조값 대신에 식별자를 사용하는 방법 등은, 모두 좋아보이지만, 실무에서는 이렇게 하면 fetch join을 통한 성능 최적화가 어렵습니다.**

DDD를 적용할 때는 모든 것을 적용한다기 보다는, DDD를 재대로 공부하고, 필요한 부분을 우리 프로젝트에 맞게 선택해서 가져오는 것이 맞는 방향이라 생각합니다.

JPA나 ORM을 처음 사용할 때는, 단순히 애그리거트 개념도 빼고, 도메인 이벤트 개념도 빼고, 연관관계는 모두 설정하는 방향으로 기본을 가져가는 것을 저는 권장합니다. 이후에 프로젝트가 커지면 점진적으로 이러한 개념을 도입하고 적용하는게 저는 더 맞다고 생각합니다^^

### 실전에서는 엔티티를 어디까지 사용하는가?

**보통 (JPA) 엔티티를 Repository와 맞닿아 있는 Service까지는 사용하기로 타협을 하는 것 같다.**

ID라는 비슷한 개념을 가지고 있기에 JPA의 엔티티가 비즈니스 로직에 침범하는 경우가 많다.

심지어 Repository를 넘어, Service 심지어 Controller까지 JPA의 엔티티가 와서 사용되는 경우를 보았다. 그러나 JPA 엔티티는 Controller 까지 넘어가면 안된다. (OSIV 트랜잭션 범위 문제, 레이어드 아키텍처 경계 무너짐 등)

**그러나 Service 경계를 넘어서는 JPA에 종속적인 JPA 엔티티가 아닌 일반 POJO 객체가 움직여야 한다.** 여기서 POJO를 세부적으로 나누어보면 구분할 수 있는 ID가 있었으므로 (DB의 PK랑은 같은 경우도 다른 경우도 있었다) VO보다는 DDD의 Entity라고 할 수 있다.

# 4. 용어 정리 (Entity, VO, DTO, Repository, DAO)

### DTO (Data Transfer Object)

- DTO
  - Layer간의 데이터 교환을 위한 자바 객체 (여러 Layer간 이동이지만, 특히 DB에서 View)
  - **비즈니스 로직이 없는 순수한 데이터 객체로, Getter, Setter 메서드만 가짐 (데이터 “전달”만이 목적)**
  - **가변**의 성격을 가짐
  - 필드값이 같아도 객체는 동일하지 않다

### VO (Value Object)

- 값 오브젝트로서 **불변!**
- **객체의 불변성을 보장하여 Read만 가능 (Setter X)**
  - 값이 변경되게 되면, 새로운 VO를 만들어서 반환해야 한다.
- **equals, hashcode 재정의하여 객체의 동일성 보장해야한다**
- 고정된 값에 VO 사용하면 됨

```java
// Entity
public class User {
    private Long id;
    private String name;
    private String email;
    private Address address;

    // Getter, Setter, 생성자 등
}

// Value Object
public class Address {
    private String street;
    private String city;
    private String state;
    private String zipCode;

    public Address(String street, String city, String state, String zipCode) {
        this.street = street;
        this.city = city;
        this.state = state;
        this.zipCode = zipCode;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Address address = (Address) o;
        return Objects.equals(street, address.street) &&
               Objects.equals(city, address.city) &&
               Objects.equals(state, address.state) &&
               Objects.equals(zipCode, address.zipCode);
    }

    @Override
    public int hashCode() {
        return Objects.hash(street, city, state, zipCode);
    }
}

// 사용 예시
Address address1 = new Address("123 Main St", "Anytown", "CA", "12345");
Address address2 = new Address("123 Main St", "Anytown", "CA", "12345");

// VO의 비교
System.out.println(address1.equals(address2)); // true (모든 속성이 같음)
```

## +a) [Entity와 VO의 차이](https://jaeyeolshin.github.io/2016-02-06/difference-between-entity-and-value-object/) (여기서 엔티티는 데이터베이스 엔티티가 아닌 DDD 엔티티)

- **엔티티(Entity)의 정체성은 id로 “ unique하게 식별”되고 속성이 변할 수 있는 “가변객체”이다.**

  - 일반적으로 사람, 제품, 파일, 판매 등이다.

- **값 객체(VO)는 “불변 객체”이다.**

  - 예시로 어떠한 사람이 위치가 변한다고 위치 객체를 변경하지 않는다. 새로운 위치 객체를 만든다.
  - 일반적으로 위치, 날짜, 숫자, 금액 등이다.
  - 그런데 만약 위치 자체가 중요해져 많은 사람들이 같은 위치에서 체크인을 하게 되는 서비스라면? 위치가 더이상 VO가 아닌 Entity가 된다.

- 왜 구분해야 하는가?
  1. 동일 여부 판단 기준이 다르다.
     - 두 개의 속성이 같은 엔티티는 ID가 다름으로 다른 객체이다. 하지만 같은 속성을 가지는 VO는 같은 객체이다. (equals() 판단시 중요)
       - _여기서 혼동하면 안되는게, ID도 필드임으로 “속성”이라고 생각하면 안된다. 어떠한 객체가 가지는 “속성”과 이를 바탕으로 어떻게 두 객체가 다른지 식별하는 방법에 대해 이야기하고 있는 것이다. 예시로, 가진 모든 필드값 아예 똑같더라도 두 객체 주소값이 다르면 같은 객체라고 말할 수 있는가? 이는 개발자가 equals() 메서드를 어떻게 구현하기 나름일 것이다._
       - 같은 객체라면 자유롭게 서로 대체할 수 있으나 엔티티는 그렇게 하면 안될것이다.
     - 엔티티는 속성이 변해도 같은 엔티티이다. 하지만 VO는 속성이 변한다면 항상 다른 VO이다.
     - 이는 도메인 주도 설계 (DDD)에서 중요한 개념이다.
  2. 비즈니스 로직과 밀접하게 연관되어있다.
     - 모든 객체가 논리적인 식별자를 가질 수는 없고 그럴 필요도 없다.
     - 그리고 언뜻 보았을 때 어떤 객체가 식별자를 가져야 한다고 무작정 그걸 엔티티로 만들면 안된다. VO와 엔티티를 구분하는 데 있어서는 어플리케이션에 대한 완벽한 이해를 바탕으로 한다.
     - 예시로
       **전자상거래 애플리케이션의 경우 고객 프로필에는 주소 정보가 포함될 수 있다. 이 주소는 중복되지 않으니 식별될 수 있다. 그러나 개인 또는 회사에 대한 특성 그룹을 나타내는 것이지, 독립적인 엔터티로 취급될 필요는 없다. 따라서 이 주소에는 ID를 부여할 필요가 없다.**
       **반면, 전력 회사 애플리케이션이라면, 고객 주소는 청구 시스템과 직접 연결되어야 하는 중요한 정보다. 따라서 이 주소는 독립적인 도메인 엔터티로 분류되어야 하며, ID를 가져야 한다!**

아래는 해당 블로그에 달린 댓글이다. 여기서 댓글을 단 사람은 JPA 엔티티와 블로그 필자가 쓴 DDD 엔티티를 혼동하여 아래와 같은 질문을 한 것으로 보인다.

![Untitled](./JPA%20Entity%20&%20DDD%20Entity%20차이/Untitled%203.png)

### 참조

[https://namocom.tistory.com/980](https://namocom.tistory.com/980)

사례 : [https://applepick.tistory.com/153](https://applepick.tistory.com/153)

[https://wlswoo.tistory.com/67](https://wlswoo.tistory.com/67)

### [만약 굳-이 Repository, DAO의 정확한 의미적 차이를 알고싶다면?](https://bperhaps.tistory.com/entry/Repository%EC%99%80-Dao%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90)

요약하면..

**현재는 DAO나 Repository나 같다고 봐도 무방하다.**

현업의 코드들도 어쩌면 JpaRepository를 상속했다는 이유로 xxxRepository라고 명명한 것일 수도 있다.

위 블로그 필자는 DAO를 사용하더라도 도메인 레이어에 속하는 Repository와 같은 interface를 이용하여 구현하면(데코레이팅) 충분히 Repository처럼 사용할 수 있다고 생각한다고 한다.

[김영한 강사님 답변](https://www.inflearn.com/questions/111159/domain%EA%B3%BC-repository-%EC%A7%88%EB%AC%B8)

이 둘은 거의 같다고 생각하셔도 무방합니다. 좀 더 깊이있게 차이를 설명하면, **repositroy는 엔티티 객체를 보관하고 관리하는 저장소이고, dao는 데이터에 접근하도록 DB접근 관련 로직을 모아둔 객체입니다.** 둘다 개념의 차이일뿐 실제로 개발할 때는 비슷하게 사용됩니다.

### 그러나 설계적인 의미의 차이를 알고싶다면 아래를 보자.

Presentation - Application - Domain - Infrastructure

- Domain은 비즈니스 로직을 처리하는 핵심 계층으로, **시스템의 핵심 로직**이 담겨 있다.
  - Infrastructure Layer 계층을 직접적으로 참조하면 안되며, 해당 계층은 **순수한 비즈니스 로직만을 담당해야 한다.**
  - 대표적인 예시로 Spring Data JPA
- Infrastructure는 보통 영속성 레이어이며 DB, 네트워크, 파일 시스템, 외부 라이브러리 들과 연결되는 계층을 말한다.
  - 대표적인 예시로 _Spring JDBC_

### DAO

**데이터에 접근하도록 DB접근 관련 로직을 모아둔 객체**

DAO 는 기존에 어플리케이션이 영구저장소에 접근하는 방법의 문제를 해결하기 위해 나왔다.

기존에는 각 영구 저장소 밴더에서 제공하는 API를 통해서 접근했었다. 그러나 이 방법은 구현체롸 로직이 너무 강한 결합을 갖게 하고, 인프라와 응용계층 (Application) 이 섞여버리는 문제를 가지게 한다.

DAO는 Infrastructure 밴더와 API 로직 사이의 어댑터 역할을 한다. 벤더 구현체를 그대로 사용하는 것이 아니라 DAO라는 객체로 한번 더 감쌈으로서 내가 사용하는 데이터 소스가 변경되더라도 로직에 변화가 없도록 함.

**DAO는 자신이 영속성 객체라는 것을 숨기지 않는다.** 이름에서 그 의도를 바로 나타낸다. **자신이 인프라 스트럭쳐 레이어에 있다는 걸 숨기지 않는다.** 또한 계층(모듈)의 의존성을 봤을 때 도메인이 인프라 스트럭쳐에 의존한다. 따라서 DAO를 이용해 바로 entity를 컨트롤 하는것은 계층을 무너트리고 잘못된 사용이다. 항상 DTO를 사용하여 데이터를 주고받아야 한다(같은 이유로 DTO에 Entity를 넣어서 보내는 행위 또한 안된다.)

### Repository

**엔티티 객체를 보관하고 관리하는 저장소**

Repository는 DDD에서 처음 등장한 개념으로 **Domain 계층에 속해있다.** Repository는 객체의 상태를 관리하는 저장소임으로 엔티티 그 자체를 저장하고 불러오는 역할을 한다. 당연하게 Repository는 도메인 레이어에 대한 지식을 알고 있어야 한다.

여기서 혼동하지 말아야 할 것은 **Repository는 영구저장소를 의미하는게 아니다 (Infrastructure 단이 아니다).** 말 그대로, **객체의 상태를 관리하는 저장소**일 뿐이다. **즉, Repository의 구현이 파일시스템으로 되든, 아니면 HashMap으로 구현됐든지 상관없다. 그냥 객체(entity)에 대한 CRUD를 수행할 수 있으면 된다.**

클라이언트는 Repository의 인터페이스만을 사용한다. 따라서 그 구현을 모른다. DAO와 같다. 구현을 숨김으로써 로직을 한곳으로 응집시켰다(캡슐화). 따라서 서비스 로직에서는 "도메인"의 레포지토리(컬렉션)을 사용한 것 뿐이다. 영속성과 관련된 로직은 전혀 모른다**. 또한 인프라스트럭처와 관련된 모듈 또한 전혀 import되지 않는다.** 따라서 Repository가 실제로 바인딩 되어있는 콘크리트 객체는 인프라스트럭쳐 레이어에 속해있고, Repository는 도메인 계층에 속해있음으로 계층이 깨지는 일은 없다.

**Repository의 Interface는 도메인 레이어,  Repository의 구현체는 영속성 레이어(Infrastructure)에 속한다. 이를 통해 도메인 레이어와 인프라스트럭처 레이어의 의존성이 뒤집힌다(DIP). 따라서 이젠 인프라스트럭처 레이어에서 도메인의 지식을 알아도 괜찮으며 이를 통해 Entity를 영속성 로직에 포함시킬 수 있게 되는 것이다. (도메인 계층이 인프라스트럭처 계층에 의존하지 않게 된다)**

**Repository는 자신이 영속성 객체임을 숨긴다. 자신을 구현하는 구현체가 인프라스트럭처 레이어에 있다는 것을 숨긴다.** 또한 의존성이 역전되어 있기 때문에, entity를 그대로 가져와 영속성 로직을 수행하는 것을 가능하게 한다.

### 참조

[https://bperhaps.tistory.com/entry/Repository와-Dao의-차이점](https://bperhaps.tistory.com/entry/Repository%EC%99%80-Dao%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90)
