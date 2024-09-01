# 아이솔레이션은 언제 시작되는가? (트랜잭션과 락의 상관관계, SELECT FOR UPDATE)

태그: JPA, MySQL, Transaction
날짜: 2024년 7월 7일

## 트랜잭션

스프링 오픈 카톡방에서 어떤 분이 왜 비관 쓰기락이 안걸리는지에 대한 질문함

원인은 트랜잭션 내부에서 가장 첫 쿼리가 쓰기락을 걸어둔 쿼리가 아닌 숨겨진 조회 쿼리가 우선하는 바람에 생겼던 문제였음

그런데 고수분들이 답변해주신 내용이 나도 궁금해서 실험해봄

### 1.1 문제상황

```sql
트랜 a 시작
트랜 b 시작
트랜 b 데이터 추가
트랜 b 커밋
트랜 a 조회 → b에서 수정한 데이터를 읽어올 수 있다
```

### 1.2 이유

트랜잭션 a가 먼저 시작되었는데 왜 이후에 시작한 트랜잭션 b의 수정사항을 읽을 수 있는걸까?

이는 트랜 a 에서 조회 쿼리를 안 쐈기 때문이다!

**⇒ 아이솔레이션은 첫 쿼리가 발생되고 나서부터 시작 된다.**

따라서, 만약 트랜잭션 b가 데이터를 넣기 전에 트랜잭션 a가 조회 쿼리를 날린다면 아이솔레이션이 시작되면서 트랜잭션 a가 종료될 때까지 트랜잭션 b에서 추가한 데이터를 읽어올 수 없다.

```sql
트랜 a 시작 & 데이터 조회 → 아이솔레이션 시작
트랜 b 시작
(여기서 트랜 a 데이터 조회해도 결과는 같이 나온다)
트랜 b 데이터 추가
트랜 b 커밋
트랜 a 조회 → 이미 아이솔레이션이 시작되었음으로 b에서 추가한 데이터를 읽어올 수 없다
```

아래는 [재형님 글](https://velog.io/@choicore/모르는데-어떻게-알아요.-feat.-Transactional)의 일부분 이다.

```sql
*START TRANSACTION
INSERT INTO ITEM (NAME) VALUES ('$1');
INSERT INTO ITEM (NAME) VALUES ('$1');
INSERT INTO ITEM (NAME) VALUES ('$1');
..
..
SELECT * FROM ITEM;
COMMIT*
```

```java
*@Id @GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;*
```

_위와 같은 옵션을 사용하면 채번 전략을 데이터베이스에게 위임하기 때문에 쓰기 지연이 동작하지 못하고 즉시 flush가 되어 insert 구문이 동작하는 것을 볼 수 있을 것이다._

_하지만, insert sql이 내 눈에 보인다고 해서 DB에 반영되는 것은 아니다._

> **_데이터베이스는 무조건 'COMMIT'이 있어야 데이터가 반영된다._**

_즉, 커밋이 나가기 전에는 당연히 '다른 작업'에서는 알 수가 없다. (같은 트랜잭션 안에서는 알 수 있다.)_

### **2. 락 잡히는 기준?**

_트랜잭션 시작이 아니라 조회를 했을때 기준으로 락이(?)걸린다?_

**트랜잭션을 시작하고 “첫 쿼리를 시작했을 때 부터” 락이 걸리면서 일관성을 지키게 된다. 이 락은 트랜잭션 종료될 때까지 유지된다.**

따라서 일관성이 필요하면 **락을 거는 쿼리를 트랜잭션의 최초 실행으로 둬야 한다.**

아래의 트랜잭션과 락의 획득, 반납 흐름을 보자

1. 트랜잭션 시작
2. **첫 번째 쿼리를 실행함과 동시에 해당하는 락을 획득 (아이솔레이션 시작)**
3. 쿼리 실행 및 작업
4. 커밋을 하면서 트랜잭션 종료 & 락 반납

### 3. SELECT FOR UPDATE, @Lock

해당 데이터를 수정할 예정으로, 동시성 제어를 위해 해당 데이터에 배타적 LOCK을 거는 SQL 문이다.

```sql
SELECT * FROM people WHERE id=1 FOR UPDATE;
```

그렇다면, JPQL에서는 어떻게 락을 걸까?

- **@Lock(LockModeType.PESSIMISTIC_READ):** 공유 락. 다른 트랜잭션에서 읽기는 가능하지만 쓰기는 불가능
- **@Lock(LockModeType.PESSIMISTIC_WRITE):** 배타 락. 다른 트랜잭션에서는 읽기와 쓰기 모두 불가능
- **@Lock(LockModeType.PESSIMISTIC_FORCE_INCREMENT) :** PESSIMISTIC_WRITE와 유사하게 작동하지만 추가적으로 낙관적 락처럼 버저닝을 하게 됩니다. 따라서 버전에 대한 칼럼이 필요합니다.

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@QueryHints({@QueryHint(name = "javax.persistence.lock.timeout", value = "1000")}) // 1초까지 락을 획득하도록 대기
@Query("SELECT p FROM Post p WHERE p.id = :postId")
Optional<Post> findByIdForUpdate(@Param("postId") Long postId);
```

주의할 점으로 **javax.persistence.lock.timeout 이 DB에 따라 동작하지 않을 수 있다고 한다. (MariaDB 안됨)**

### 4. [Select 쿼리는 S락이 아니다. (X락과 S락의 차이)](https://velog.io/@soongjamm/Select-%EC%BF%BC%EB%A6%AC%EB%8A%94-S%EB%9D%BD%EC%9D%B4-%EC%95%84%EB%8B%88%EB%8B%A4.-X%EB%9D%BD%EA%B3%BC-S%EB%9D%BD%EC%9D%98-%EC%B0%A8%EC%9D%B4)

_S-Lock = Shared-Lock, 읽기 락_

_X-Lock = Exclusive-Lock, 쓰기 락, 배타적 락_

### 문제 상황

> 1번 트랜잭션에서 A레코드를 SELECT FOR UPDATE 사용해서 X-Lock 획득
>
> 2번 트랜잭션은 A 레코드를 SELECT 하지 못하고 1번 트랜잭션이 끝날 때까지 대기할것으로 예상하였으나, 조회가 됨

### 이유

이유는 SELECT 라는 행위가 곧 S-Lock을 가지는 것이 아니기 때문

**MySQL InnoDB는 SELECT시 S락을 걸지 않고 조회함**

### 읽기락(공유락) vs 쓰기락(배타락)

쓰기 락을 걸면 다른 트랜잭션에서 읽기락을 얻지 못함

S락 끼리는 호환되고, X락이 끼면 호환되지 않음

읽기 락 : 내가 이 데이터의 정합성이 매우 중요하니까, 읽는 동안 데이터가 바뀌면 안돼. 아무도 나 끝날 때까지 쓰지마! **(다른 트랜잭션 읽기락 얻을 수 있음, 쓰기락은 얻지 못함)**

쓰기 락 : 내가 이 데이터를 쓸 꺼니까 다른 사람들 변경하지 말고, 만약 변동없이 정합성 보장해서 읽고 싶으면 나 다 쓰고 읽어! **(다른 트랜잭션 쓰기락, 읽기락 얻지 못함)**

### 결론

따라서 의도된 대로 동작하게 하려면

2번 트랜잭션이 `SELECT ~ LOCK IN SHARE MODE`이나 `SELECT FOR SHARE`를 사용해서 읽기락을 얻도록 요청하게 해야 SELECT 하지 못하고 대기하는 상태가 될 것이다.

### 해당 블로그에 댓글로 달려있던 타 DBMS와 다른 innodb의 동작 설명

> innodb에서 select를 사용할 경우, Consistent Nonlocking Reads로 동작해서 undo 로그에 있는 snapshot을 읽음
>
> 보통은 **select가 락이 걸린 것을 읽을 수 없어야 하는데,** innodb는 undo 로그까지 락을 걸지 않아서 **select ... for update로 락이 걸린 것을 select를 통해서 읽을 수가 있다**고 함.

1. **select가 락이 걸린 것을 읽을 수 없어야 하는데**

   이 부분 앞에서 설명한거랑 약간 혼동될 수 있는 느낌. 물론 이해는 제대로 하고 계신듯.

2. **select ... for update로 락이 걸린 것을 select를 통해서 읽을 수가 있다**

   이 부분은 조금 혼동이 있을 수 있다고 함. 뜻 자체는 맞음 (언두로그를 읽는 다는 뜻이라면)

   **일반적인 `SELECT`는 `SELECT ... FOR UPDATE`로 잠긴 행의 현재 값을 읽을 수 없음. 대신, 일반적인 `SELECT`는 트랜잭션 격리 수준에 따라 일관된 비잠금 읽기를 통해 잠금을 피하고 이전 스냅샷(언두 로그에서)을 읽게 됨.** 이는 일반적인 읽기 작업이 잠금으로 인해 차단되지 않도록 하기 위함

> If you want to see the “freshest” state of the database, use either the [`READ COMMITTED`](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-isolation-levels.html#isolevel_read-committed) isolation level or a [locking read](https://dev.mysql.com/doc/refman/8.4/en/glossary.html#glos_locking_read):
>
> `SELECT * FROM t FOR SHARE;`
>
> With [`READ COMMITTED`](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-isolation-levels.html#isolevel_read-committed) isolation level, each consistent read within a transaction sets and reads its own fresh snapshot. With `FOR SHARE`, a locking read occurs instead: A `SELECT` blocks until the transaction containing the freshest rows ends (see [Section 17.7.2.4, “Locking Reads”](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking-reads.html)).

https://dev.mysql.com/doc/refman/8.4/en/innodb-consistent-read.html

요약하면, `READ_COMITTED` 로 트랜잭션 격리 레벨을 지정하면 각 시점의 최신 데이터를 읽게 해줄 수 있다. 그러나 이는 같은 트랜잭션에서도 읽기 작업마다 다른 결과를 볼 수도 있다는 뜻이다.

따라서 `READ_COMMITTED`는 InnoDB의 기본 트랜잭션 레벨인 `REPETABLE READ` 보다 더 낮은 트랜잭션 레벨을 사용하는 것이다. **이는 높은 격리수준인 `REPEATABLE READ`가 트랜잭션 동안 일관된 읽기 일관성을 제공하지만, 역으로 최신 데이터를 보장하지는 않는다는 것을 의미한다.**

따라서 최신 데이터를 보장하는 방법은 아래와 같다

1. **`READ_COMMITTED` 를 사용하면 최신 데이터는 보장하지만, 데이터 일관성은 떨어진다.**
2. **읽기/쓰기 락(`FOR SHARE` 또는 `FOR UPDATE`)을 사용하여 다른 트랜잭션의 종료를 기다리며 커밋된 최신 데이터를 읽기 위해 기다릴 수 있다.**

### 다른 코드 공부하면서 생긴 의문

다른 코드 공부하면서 **왜 Service 계층에서 일부 트랜잭션을 `@Transactional(isolation = Isolation.READ_COMMITTED)` 으로 설정했던 걸지 궁금했었다.**

`@Transactional` 이 없는 코드는 repository 내부 트랜잭션을 사용한 것일텐데, 특정 구간에는 (아니 사실 지정한 곳에는 전부) 왜 굳이 READ_COMMITTED을 사용했는지가 궁금했었다.

> **결론은 읽기 락 사용하지 않고 최신 데이터를 확인하기 위함이었다.**
>
> **`READ_COMMITTED`으로 트랜잭션을 사용했음에도 불구하고 쓰기 락을 사용한 이유는, 최신 데이터 확인 외에도 데이터 정합성을 위해 동시 데이터 수정을 방지하기 위함일 것이다.**

AccountService

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public void saveMainAccount(Long memberId) {
    Member member = memberService.getMemberById(memberId);
    Account account = createAccount(Type.REGULAR_ACCOUNT, member);

    accountRepository.save(account);
}

 // 메인 계좌가 존재하는지 확인
public boolean isMainAccount(Long memberId) {
    return accountRepository.existsAccountByMemberIdAndType(memberId, Type.REGULAR_ACCOUNT);
}

// 계좌 객체 생성
public Account createAccount(Type type, Member member) {
    return Account.builder()
        .type(type)
        .member(member)
        .build();
}

@Transactional(isolation = Isolation.READ_COMMITTED)
public void transferFromRegularAccount(TransferToOtherAccountRequestDto transferToOtherAccountRequestDto) {

	// 내부 로직...

}
```

AccountRepository

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("""
    SELECT a FROM Account a
    WHERE a.member.id = :memberId AND a.type = :type
    """)
Optional<Account> findAccountByMemberId(Long memberId, Type type);

@Lock(LockModeType.PESSIMISTIC_WRITE)
Optional<Account> findAccountById(Long accountId);

@Modifying
@Query("""
        UPDATE Account a SET a.balance = a.balance - :balance
        WHERE a.member.id = :memberId AND a.type = :type AND a.balance >= :balance
    """)
int transferAccount(Long memberId, Long balance, Type type);
```

### 참조

[https://velog.io/@rookedsysc/비관적-락이-걸리지-않는-문제](https://velog.io/@rookedsysc/%EB%B9%84%EA%B4%80%EC%A0%81-%EB%9D%BD%EC%9D%B4-%EA%B1%B8%EB%A6%AC%EC%A7%80-%EC%95%8A%EB%8A%94-%EB%AC%B8%EC%A0%9C)

[https://innovation123.tistory.com/222#락을 고려한 트랜잭션 과정-1](https://innovation123.tistory.com/222#%EB%9D%BD%EC%9D%84%20%EA%B3%A0%EB%A0%A4%ED%95%9C%20%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98%20%EA%B3%BC%EC%A0%95-1)

[https://velog.io/@choicore/모르는데-어떻게-알아요.-feat.-Transactional](https://velog.io/@choicore/%EB%AA%A8%EB%A5%B4%EB%8A%94%EB%8D%B0-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%95%8C%EC%95%84%EC%9A%94.-feat.-Transactional)

[https://wildeveloperetrain.tistory.com/128](https://wildeveloperetrain.tistory.com/128)

lock 관련

https://joojimin.tistory.com/71

https://devhyogeon.tistory.com/4

https://devhyogeon.tistory.com/5
