# [스터디] POST 멱등성 확보 방법 (멱등키, e-tag, UPSERT 등)

날짜: 2024년 7월 12일

## 주제 선정 이유

오픈카톡방에서 나온 XX 기업 질문

질문 : “A은행에서 B은행에 입금했는데 응답이 안오면 어떻게 처리해야되나요?

타임아웃나서 재시도했는데 5000원 입금되야할게 10000원 입금 되어버리면 어떡하나요?“

답 : “결제마다 키를 생성해서 같이 전달하면 멱등하게 처리합니다.”

## 멱등성이란?

첫 번재 작업을 수행한 뒤에 추가적으로 여러번 수행해도 첫 작업의 결과를 변경시키지 않는 것. (이후 설명하겠지만 응답코드등의 결과 반환값은 중요치 않다)

예시 : \* 1, 절대값, 등등

## HTTP 메서드 에서의 멱등성

**HTTP에서는 POST, PATCH, CONNECT 메서드는 멱등적이지 않다**

- **PATCH : 리소스의 일부를 수정**
  - 대부분의 PATCH는 생각해보면 멱등성이 보장된다. 그러면 언제 멱등성이 보장이 안되는걸까? : 아래처럼 기ㅗㄴ 리소스에 응답을 추가, 증가하도록 구현하는 경우. (실제로 이렇게 하는 경우는 거의 없으나 PATCH 메서드의 제한이 없음으로 표준을 어기는게 아니긴 하다.
  ```json
  PATCH users/1
  {
      $increase: 'age',
      value: 10,
  }
  ```
- **CONNECT : 대상 자원으로 식별되는 서버에 대한 터널을 설정**
  - 동일한 요청을 보내면 각 요청마다 새로운 자원이 소모되어 새로운 세션이 설정되면서 기존 연결상태에 영향을 미치기 때문
- **DELETE는 멱등한가?**
  - 답부터 말하면 멱등하다. DELETE를 사용해서 어떠한 계좌를 삭제하고자 한다. 첫번째 요청에서 200 OK를 받고 다시 삭제하였을 때 404 NOT FOUND를 반환했다고 하자. 이럼에도 불구하고 멱등한데, 멱등성의 기준은 “상태코드”가 아닌 “서버에 미치는 의도된 영향”이기 때문이다. 첫번째 요청을 보낸 후와 추가 요청을 보낸 후의 서버 상태가 동일하기 때문에 멱등하다고 할 수 있다.

### 멱등성 vs 안정성

- **참고로 GET은 멱등성과 안전성을 보장하지만 PUT, DELETE는 안전하지는 않다. 안정한 메서드는 리소스의 변화를 아예 일으키지 않는 메서드이다. 안전성은 멱등성 보장을 포함하는 개념이다.**

## 토스 페이먼츠에서 사용하는 멱등키

![image](./[스터디]%20POST%20멱등성%20확보%20방법/Untitled.png)

**위 그림은 이해를 돕기 위해 가져온 것으로, 토스 페이먼츠에서 제작한 그림이 아닙니다!**

### **멱등키 사용하기** (토스페이먼츠 개발자 센터에서 발췌)

요청 헤더에 `Idempotency-Key`를 추가하면 멱등한 요청을 보낼 수 있습니다.

`Idempotency-Key: {IDEMPOTENCY_KEY}`

- 멱등키는 [**UUID**](https://docs.tosspayments.com/resources/glossary/uuid)와 같이 충분히 무작위적인 고유 값으로 생성해주세요. 최대 길이는 300자입니다.
- **멱등키는 처음 요청에 사용한 날부터 15일 간 유효**합니다. 처음 요청한 날부터 15일이 지났다면 새로운 멱등키로 요청하세요.

아래와 같이 API 요청에 멱등키 헤더를 사용하면 같은 요청이 두 번 일어나도 실제로 요청이 이루어지지 않고 첫 번째 요청 응답과 같은 응답을 보내줍니다.

```bash
curl --request POST \
  --url https://api.tosspayments.com/v1/payments/confirm \
  --header 'Authorization: Basic dGVzdF9za196WExrS0V5cE5BcldtbzUwblgzbG1lYXhZRzVSOg==' \
  --header 'Content-Type: application/json' \
  --header 'TossPayments-Test-Code: REJECT_CARD_PAYMENT' \
  --data '{"paymentKey":"5zJ4xY7m0kODnyRpQWGrN2xqGlNvLrKwv1M9ENjbeoPaZdL6","orderId":"a4CWyWY5m89PNh7xJwhk1","amount":15000}'
```

- 토스페이먼츠 서버는 상점에서 API 요청 헤더로 보낸 **멱등키와 API 키, API 주소, HTTP 메서드 조합이 같은 요청이 있는지 확인**해서 멱등성을 보장합니다. 따라서 API 키, API 주소, HTTP 메서드가 다르다면 같은 멱등키를 사용해도 괜찮습니다.
- 멱등키 관리를 위한 별도의 데이터베이스나 테이블을 사용하면 서버 성능 및 보안 및 유지보수 측면에서 장점이 있습니다. 단순한 데이터 구조라면 Redis와 같은 **키-값(K-V) 저장소**를 사용해도 괜찮습니다. 멱등키와 관련된 데이터의 복잡도, 유효 기간 관리, 확장성, 성능 등을 고래해서 적절한 데이터베이스 유형을 선택하세요.

### **에러 처리하기** (토스페이먼츠 개발자 센터에서 발췌)

멱등키 헤더를 사용할 때 발생할 수 있는 에러는 두 가지가 있습니다. 멱등키 길이가 300자보다 길면 HTTP `400 - INVALID_IDEMPOTENCY_KEY`가 돌아옵니다. 300자 이하로 멱등키를 다시 만들어주세요. 첫 번째 요청이 처리중일 때 같은 요청을 다시 보내면 HTTP `409 - IDEMPOTENT_REQUEST_PROCESSING` 에러가 돌아옵니다. 이 에러가 돌아오면 다시 한 번 요청해서 응답을 확인하세요.

| HTTP Status Code | 에러 코드                     | 메시지                          |
| ---------------- | ----------------------------- | ------------------------------- |
| 400              | INVALID_IDEMPOTENCY_KEY       | 멱등키는 300자 이하여야 합니다. |
| 409              | IDEMPOTENT_REQUEST_PROCESSING | 이전 멱등 요청이 처리중입니다.  |

### 클라이언트에서 멱등 키를 만드는 기준은 어떤게 있을까?

아래 방법 외에도 생각해볼 수 있다.

1. **요청마다 고정된 Idempotent-Key를 생성**
   - 결제 요청을 한다고 했을 때, 결제 프로세스가 시작될 때 기준으로 멱등키를 생성
   - 만약 결제 도중에 튕겨서 다시 요청한다면
     1. 기존 멱등키가 남아있다면 중복 처리가 방지된다.
     2. 기존 멱등키 없이 요청한다면 정상적으로 결제 프로세스 시작부터 진행된 것이 아님으로 멱등키가 없어서 BAD_REQUEST를 반환하는 방법이 있을 것이다. 새로운 멱등키는 만들어지지 않는다.
2. **특정 페이지에 들어올 때마다 고정된 Idempotent-Key를 사용**
   - 사용자가 특정 페이지에 들어올 때마다 고정된 Idempotent-Key를 생성하고, 해당 페이지에서 이루어지는 모든 POST 요청에 동일한 키를 사용하는 방식
   - 이 방법은 페이지 단위로 멱등성을 보장. 즉, 사용자가 페이지를 다시 로드하지 않는 한 동일한 키가 사용.
   - 구현이 간단하나 페이지를 새로고침하면 다른 요청으로 간주될 수 있음
3. **요청하는 정보의 조합에 따라 Idempotent-Key를 생성**
   - 요청하는 정보(예: 폼 데이터, API 호출 파라미터 등)를 기반으로 Idempotent-Key를 생성.
   - 요청의 내용에 따라 멱등성을 보장하므로, 동일한 데이터로 요청할 경우 같은 키가 사용됨.
   - 요청 데이터를 해시 값으로 계산해야함으로 약간의 성능 오버헤드가 있을 수 있음.
   - 그리고 흔히 “딸깍”이라고 부르는 중복 요청과 나중에 같은 정보 조합으로 보내는 요청에 대해 중복 구분이 불가능하다.
   - 예시로 유저 1이 물건 1을 사는 요청이라면 나중에 유저 1이 물건 1을 사는 경우에도 중복 요청으로 인식할 수 있다.
   - 따라서 이를 개선하기 위해서는 요청 시간을 고려하거나 추가적인 방법으로 고려해야 할 것이다.

### 토스페이먼츠에서는? (내 추측)

개인적으로는 멱등키를 1번 방법으로 사용하는 것 같다. **“UUID와 같이 충분히 무작위적인 고유 값으로 생성해달라”** 고 하는 것에서 추측이 가능하다. 그러나 멱등키만으로 멱등성을 판단하는 것은 아닌데, 이는 **“API 키, API 주소, HTTP 메서드가 다르다면 같은 멱등키를 사용해도 괜찮습니다.”** 라는 구문을 보면 된다.

즉, 아래처럼 작동하는 것 아닐까?

1. **클라이언트가 결제 프로세스가 시작될 때 멱등키를 생성한다. 즉, 결제 요청마다 멱등키는 고유하다고 가정된다.**
2. **Idempotency-Key: {IDEMPOTENCY_KEY} 헤더에 멱등키를 넣고, 주문 ID와 가격을 포함하여 요청하면 멱등키만 사용하는 것이 아닌, (멱등키-API키-API주소-HTTP메서드) 조합을 기준으로 멱등성을 판단하는 것 같다. 이 과정에서 혹시나 모를 멱등키 중복으로 인한 오작동을 방지할 수 있다.**
   1. 정확히 알 수는 없지만 paymentKey는 결제 기준? 같은 거로 멱등키와는 무관한 것 같고, orderId는 주문이라는 DB 엔티티의 식별자라고 판단된다.
3. **따라서 첫 요청이라면 이 값을 키로, 응답값을 밸류로 redis에 저장하고, TTL을 15일로 설정한다.**
4. **다른 요청이 온다면 (멱등키-API키-API주소-HTTP메서드) 조합이 redis에 저장되어있는지 확인하고, 동일한 조합이라면 redis에 저장된 값을 반환한다.**

**(그런데 원문에서는 Idempotency-Key: {IDEMPOTENCY_KEY} 헤더를 추가하여 멱등키를 첨부한다고 되어있는데, 예시에는 Idempotency-Key 헤더가 없었다. 생략된 것이라고 판단하고 과정을 작성해봤다.)**

## 이외의 멱등성 보장 방법

### 1. **Idempotent-Key**

### **2. Entity Tag**

1. 멱등성 키 헤더 사용과 거의 유사하나 **“서버에서 생성”**해서 응답으로 보내줌
2. 특정 URL의 리소스가 변경되면 새로운 Etag를 생성하고, 이들의 비교를 통해 자원이 동일한지 여부를 확인

**요청**

```
GET /users/123
```

응답

```
HTTP/1.1 200 OK
Etag: "33a64df551425fcc55e4d42a148795d9f25f89d4"

{
"name": "aaaa",
"email": "aaaa@abc.com"
}
```

**If-Match를 통한 Etag 확인**

```
POST /users/123
If-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"

{
"name": "aaaa"
}
```

**스프링 내에서 구현 방법**

```java
public String getEtag() {
    return DigestUtils.md5Hex(title + content + lastModified.getTime());
}
```

### **3. 분산 락 사용**

1. 일명 “따닥” 요청 방지 방법이다
2. redis 분산락 등을 사용하여 요청이 시작될 때 Lock을 걸고, 요청이 끝나면 락을 푼다.
3. 동시에 2개 들어온 요청 중 나중에 들어온 요청은 Lock으로 인해 무시되게 된다.

### **4. 추상화된 멱등성 구현 라이브러리 사용**

AWS Lambda Powertools : 멱등성의 구현 추상화

파이썬 사용 시 **`*@idempotent` 어노테이션으로 간단하게 멱등성 작업 구현\***

DynamoDB만 지원하고 있으며, Redis나 다른 Database로 지정하려면 커스텀하게 로직 구현해야

### 5. UPSERT로 멱등성 확보하기

Upsert란? : 특정 Index 기준 (unique key, 보통은 PK이나 꼭 아니어도 됨)으로 이미 값이 있으면 Update, 없으면 Insert

**이미 있는 값이면 같은 데이터로 UPDATE하게 됨으로 중복으로 요청해도 서버의 상태를 변경시키지 않는다.**

MySQL에서의 쿼리 :

```sql
INSERT INTO 테이블 VALUES ( , , ) ON DUPLICATE KEY UPDATE 중복_시_업데이트할_내용
```

SQL 문 예시 :

```sql
# 1.
INSERT INTO person (id, name, address, inserted_cnt) VALUES (NULL, 'Cynthia', 'Seoul', 1)
	ON DUPLICATE KEY UPDATE inserted_cnt = inserted_cnt + 1;

# 2.
INSERT INTO user VALUES (email, name) VALUES ("email@naver.com", "damongsanga")
	ON DUPLICATE KEY UPDATE name = "damongsanga";
```

- JPA 사용해서 UPSERT 방식으로 멱등성 부여하기 예시

  ```java
  import javax.persistence.*;

  @Entity
  @Table(name = "users")
  public class User {
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private Long id;

      @Column(unique = true)
      private String email;

      private String name;

      // 생성자, 게터/세터 메서드 생략
  }

  ```

  여기서 `email` 필드를 유일한 값으로 설정했습니다. 이 필드를 기준으로 UPSERT 작업을 수행할 것입니다.

  다음으로 Repository 인터페이스를 정의합니다:

  ```java
  import org.springframework.data.jpa.repository.JpaRepository;
  import org.springframework.data.jpa.repository.Query;
  import org.springframework.data.repository.query.Param;

  public interface UserRepository extends JpaRepository<User, Long> {

      @Query(value = "INSERT INTO users (email, name) VALUES (:email, :name) " +"ON DUPLICATE KEY UPDATE name = :name", nativeQuery = true)
      void upsert(@Param("email") String email, @Param("name") String name);
  }

  ```

  여기서 `upsert` 메서드는 MySQL의 `ON DUPLICATE KEY UPDATE` 문법을 사용하여 UPSERT 작업을 수행합니다. 이메일 값이 중복되면 기존 데이터를 업데이트하고, 중복되지 않으면 새로운 데이터를 생성합니다.

  마지막으로 Service 클래스에서 UPSERT 메서드를 호출합니다:

  ```java
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.stereotype.Service;

  @Service
  public class UserService {
      @Autowired
      private UserRepository userRepository;

      public void createOrUpdateUser(String email, String name) {
          userRepository.upsert(email, name);
      }
  }

  ```

  이렇게 구현된 `createOrUpdateUser` 메서드를 통해 클라이언트는 PK(id)를 모르더라도 이메일 값을 사용하여 사용자 정보를 생성하거나 업데이트할 수 있습니다. 중복된 이메일로 요청이 들어오더라도 기존 데이터가 업데이트되므로 멱등성이 보장됩니다.

  이와 같은 UPSERT 방식을 사용하면 클라이언트가 PK를 모르더라도 멱등성을 구현할 수 있습니다. 이는 REST API 설계 시 유용하게 사용될 수 있습니다.

- 주의점

  - UPSERT문 같은 경우는 DB 벤더에 따라 다른 형태를 가지고 있어서 ORM에서 BULK 구문 자동생성이 없다면, 특정 DB에 종속적인 코드를 작성해야
  - 또, BULK쿼리가 실행되다가 쿼리 실행에 실패한 경우, DB 벤더와 DB 설정에 따라 해당 BULK쿼리의 모든 내용이 ROLLBACK될 수 있음
  - 성능상의 장점이 있지만, 운영상의 단점이 존재하기 때문에 면밀히 검토하고 사용하시는 것을 추천

- PK만 사용 가능하다고 생각해서 들었던 의문 :

  DUPLICATE KEY를 PK로 하게 된다면 PK로 먼저 있는지 조회하고 없으면 INSERT하는 방식 아닌가? 그러면 결국 PK를 POST에 넣어주어야 한다는 것 아닌가?

- **답 : `UNIQUE INDEX`면 됨**

  **UPSERT 방식에서는 중복 레코드 관리를 위해선 테이블에 `PRIMARY KEY` 혹은 `UNIQUE INDEX`가 필요하다. 즉, 반드시 PK(기본 키)일 필요는 없고, 유일한 값을 가지는 필드만 있으면 된다.**

  **UPSERT는 "Update if exists, Insert if not exists"라는 의미를 가지고 있습니다. 이때 "exists"의 기준이 되는 것은 유일한 값을 가지는 필드, 즉 Unique Key이다.**

- **참고로 MySQL에서는 UPDATE가 동작하면 2개의 row가 영향받았다고 인식한다.**
  ```sql
  # 고유키 기준으로 이미 있는 값이라면
  mysql> INSERT INTO person VALUES (NULL, 'James', 'Seongnam')
             ON DUPLICATE KEY UPDATE address = VALUES(address);
  Query OK, 2 rows affected, 1 warning (0.01 sec) #
  ```
- 이유는 새로운 행 삽입 시도 (`INSERT`)와 기존 행 업데이트 (`UPDATE`)를 각각 요청으로 인식하기 때문이다. 따라서 고유키 충돌에 의해 `INSERT` 가 동작하지 않더라도 MySQL은 한 행을 삽입 시도 했다고 인식, 1개의 row가 affected 되었다고 반환한다.

### 참조

토스페이먼츠

[https://velog.io/@tosspayments/멱등성이-뭔가요](https://velog.io/@tosspayments/%EB%A9%B1%EB%93%B1%EC%84%B1%EC%9D%B4-%EB%AD%94%EA%B0%80%EC%9A%94)

[https://docs.tosspayments.com/reference/using-api/authorization?utm_source=tosspayments&utm_medium=velog#멱등키-헤더](https://docs.tosspayments.com/reference/using-api/authorization?utm_source=tosspayments&utm_medium=velog#%EB%A9%B1%EB%93%B1%ED%82%A4-%ED%97%A4%EB%8D%94)

블로그

[https://www.inflearn.com/questions/110644/patch-메서드가-멱등이-아닌-이유](https://www.inflearn.com/questions/110644/patch-%EB%A9%94%EC%84%9C%EB%93%9C%EA%B0%80-%EB%A9%B1%EB%93%B1%EC%9D%B4-%EC%95%84%EB%8B%8C-%EC%9D%B4%EC%9C%A0)

[https://mangkyu.tistory.com/251](https://mangkyu.tistory.com/251)

[https://cobinding.tistory.com/279](https://cobinding.tistory.com/279)

[https://jason-heo.github.io/mysql/2014/03/05/manage-dup-key2.html](https://jason-heo.github.io/mysql/2014/03/05/manage-dup-key2.html)
