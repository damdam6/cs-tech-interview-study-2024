## 1. 읽기락과 쓰기락

이 주제는 [Select 쿼리는 S락이 아니다. (X락과 S락의 차이)](https://velog.io/@soongjamm/Select-%EC%BF%BC%EB%A6%AC%EB%8A%94-S%EB%9D%BD%EC%9D%B4-%EC%95%84%EB%8B%88%EB%8B%A4.-X%EB%9D%BD%EA%B3%BC-S%EB%9D%BD%EC%9D%98-%EC%B0%A8%EC%9D%B4) 글을 읽고서 공부해 본 내용이다.

_S-Lock : Shared-Lock, 읽기 락_

_X-Lock : Exclusive-Lock, 쓰기 락, 배타적 락_

### 문제 상황

> 1번 트랜잭션에서 A레코드를 SELECT FOR UPDATE 사용해서 X-Lock 획득
>
> 2번 트랜잭션은 A레코드를 SELECT 하려하지만 락으로 인해 실패하고, 1번 트랜잭션이 끝날 때까지 대기할것으로 예상하였으나, 조회가 되어버림

### 이유

이유는 SELECT 라는 행위가 곧 S-Lock을 가지는 것이 아니기 때문이다.
“MySQL InnoDB는 SELECT시 S락을 걸지 않고 조회”한다!

### 읽기락(공유락) vs 쓰기락(배타락)

쓰기 락을 걸면 다른 트랜잭션에서 읽기락을 얻지 못한다. 즉, S락 끼리는 호환되고, X락이 끼면 호환되지 않는다.

- 읽기 락 : 내가 이 데이터의 정합성이 매우 중요하니까, 읽는 동안 데이터가 바뀌면 안돼. 아무도 나 끝날 때까지 쓰지마!

  - **다른 트랜잭션 읽기락 얻을 수 있음, 쓰기락은 얻지 못함**

- 쓰기 락 : 내가 이 데이터를 쓸 꺼니까 다른 사람들 변경하지 말고, 만약 변동없이 정합성 보장해서 읽고 싶으면 나 다 쓰고 읽어!
  - **다른 트랜잭션 쓰기락, 읽기락 얻지 못함**

### 결론

따라서 의도된 대로 동작하게 하려면

**2번 트랜잭션이 `SELECT ~ LOCK IN SHARE MODE`이나 `SELECT FOR SHARE`를 사용해서 읽기락을 얻도록 요청하게 해야 SELECT 하지 못하고 대기하는 상태가 될 것이다.**

이러한 경우는 2번 트랜잭션이 데이터 정합성이 매우 중요한 상태여서 무조건 최신 데이터를 보장 받아야 하는 경우에 사용하는 경우일 것이다.

아래는 해당 블로그에 댓글로 달려있던 타 DBMS와 다른 innodb의 동작 설명이다.
![](https://velog.velcdn.com/images/damongsanga/post/04f55b5d-4793-4fd6-8aa0-2d2498587f05/image.png)

**`SELECT ... FOR SHARE` 는 `SELECT ... FOR UPDATE`로 잠긴 행의 현재 값을 읽을 수 없다.
대신, innoDB의 보통 `SELECT`는 트랜잭션 격리 수준에 따라 일관된 비잠금 읽기를 통해 잠금을 피하고 이전 스냅샷(언두 로그에서)을 읽게 된다.**
이는 일반적인 읽기 작업이 잠금으로 인해 차단되지 않도록 하기 위함이다.

참고로 여기서 말하는 snapshot은 트랜잭션 내에서 읽기 작업에 일관성을 제공하기 위한 "논리적 개념"으로, undo 로그라는 트랜잭션의 롤백 및 일관된 읽기 작업을 지원하기 위한 물리적 로그 파일에 기록된 이전 상태를 참조하는 것이라고 말할 수 있다.

## 2. REPETABLE READ vs READ COMMITTED (최신 데이터 보장받기)

### 다른 코드 공부하면서 생긴 의문

[다른 프로젝트 코드](https://github.com/C4-ComeTrue/c4-cometrue-assignment/tree/feature/soo_step2)를 공부하면서 계좌 관련 서비스 로직에서 **왜 Service 계층에서 일부 트랜잭션을 `@Transactional(isolation = Isolation.READ_COMMITTED)` 으로 설정했던 걸지 궁금했었다.**

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public void saveMainAccount(Long memberId) {
 // ...
}
```

> **결론 부터 말하면 읽기 락 사용하지 않고 최신 데이터를 확인하기 위함이었다.**
>
> **`READ_COMMITTED`으로 트랜잭션을 사용했음에도 불구하고 쓰기 락을 사용한 이유는, 최신 데이터 확인 외에도 데이터 정합성을 위해 동시 데이터 수정을 방지하기 위함일 것이다.**

원본 코드는 [youngsoosoo](https://github.com/youngsoosoo) 님의 https://github.com/C4-ComeTrue/c4-cometrue-assignment/tree/feature/soo_step2 에서 AccountService를 참조하면 된다. (코드 소개를 허락 맡은 부분이 아니라 언급만 해두고자 한다!)

### How to see the “freshest” state of the database

> If you want to see the “freshest” state of the database, use either the [`READ COMMITTED`](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-isolation-levels.html#isolevel_read-committed) isolation level or a [locking read](https://dev.mysql.com/doc/refman/8.4/en/glossary.html#glos_locking_read):
>
> `SELECT * FROM t FOR SHARE;`
>
> With [`READ COMMITTED`](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-isolation-levels.html#isolevel_read-committed) isolation level, each consistent read within a transaction sets and reads its own fresh snapshot. With `FOR SHARE`, a locking read occurs instead: A `SELECT` blocks until the transaction containing the freshest rows ends (see [Section 17.7.2.4, “Locking Reads”](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking-reads.html)).

원본 : https://dev.mysql.com/doc/refman/8.4/en/innodb-consistent-read.html

요약하면, `READ_COMITTED` 로 트랜잭션 격리 레벨을 지정하면 각 시점의 최신 데이터를 읽게 해줄 수 있다. 그러나 이는 같은 트랜잭션에서도 읽기 작업마다 다른 결과를 볼 수도 있다는 뜻이다.

- `READ_COMITTED`

  - 쿼리가 시작되기 전 다른 트랜잭션에서 commit 된 결과를 참조할 수 있으며 동일 트랜잭션 상에서 동일한 쿼리문에 대해 서로 다른 결과를 조회(Phantom-Read)할 수 있다.

- `REPETABLE_READ`
  - 트랜잭션이 시작되기 전(쿼리가 시작되기 전이 아닌) commit 된 결과를 참조한다. phantom-read 를 방지(**mysql의 경우, SELECT는 방지, UPDATE는 방지하지 않는다.**)하므로 **트랜잭션 내 같은 쿼리문은 트랜잭션이 시작된 시점의 데이터 스냅샷 정보를 조회한다.** 따라서 다른 트랜잭션의 commit 여부와 무관하게 항상 같은 결과가 출력된다.

참조 : https://corekms.tistory.com/entry/Read-committed-VS-Repeatable-read

여기서 phantom read를 SELECT는 방지하지만 UPDATE는 그렇지 않다는 내용이 잘 이해가 가지 않을 수 있다. 글이 너무 길어지는 관계로, [별도의 글](<./PHANTOM%20READ%20방지%20(SELECT%20vs%20UPDATE).md>)에 예시를 들어두었으니 관심있다면 참고하도록 하자.

### 결론

결론적으로 최신 데이터를 보장하는 방법은 아래와 같다

1. **`READ_COMMITTED` 를 사용하면 최신 데이터는 보장하지만, 데이터 일관성은 떨어진다.**

2. **읽기/쓰기 락(`FOR SHARE` 또는 `FOR UPDATE`)을 사용하여 다른 트랜잭션의 종료를 기다리며 커밋된 최신 데이터를 읽기 위해 기다릴 수 있다.**

`READ_COMMITTED`는 InnoDB의 기본 트랜잭션 레벨인 `REPETABLE_READ` 보다 더 낮은 트랜잭션 레벨을 사용하는 것이다. **이는 높은 격리수준인 `REPEATABLE READ`가 트랜잭션 동안 일관된 읽기 일관성을 제공하지만, 역으로 최신 데이터를 보장하지는 않는다는 것을 의미한다.**

참조
https://velog.io/@soongjamm/Select-%EC%BF%BC%EB%A6%AC%EB%8A%94-S%EB%9D%BD%EC%9D%B4-%EC%95%84%EB%8B%88%EB%8B%A4.-X%EB%9D%BD%EA%B3%BC-S%EB%9D%BD%EC%9D%98-%EC%B0%A8%EC%9D%B4
https://joojimin.tistory.com/71
https://devhyogeon.tistory.com/4
https://devhyogeon.tistory.com/5
