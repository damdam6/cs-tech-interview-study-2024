[앞선 REPETABLE READ vs READ COMMITTED 관련 글](./읽기락과%20쓰기락,%20REPETABLE%20READ와%20READ%20COMMITTED.md)에서 MySQL의 `REPEATABLE READ` 격리 수준에서는 SELECT 쿼리에 대해 Phantom Read를 방지하지만, UPDATE 쿼리의 경우에는 그렇지 않는다고 했었다. 이 부분에 대해 더 자세히 알아보자.

- `REPETABLE_READ`
  - 트랜잭션이 시작되기 전(쿼리가 시작되기 전이 아닌) commit 된 결과를 참조한다. phantom-read 를 불허(**mysql의 경우, SELECT는 안되는데 UPDATE는 된다.**)하므로 **트랜잭션 내 같은 쿼리문은 트랜잭션이 시작된 시점의 데이터 스냅샷 정보를 조회한다.** 따라서 다른 트랜잭션의 commit 여부와 무관하게 항상 같은 결과가 출력된다.

참조 : https://corekms.tistory.com/entry/Read-committed-VS-Repeatable-read

<hr>

### SELECT vs UPDATE (PHANTOM READ)

- **SELECT** 쿼리는 **“트랜잭션이 시작된 시점(정확히는 트랜잭션에서 첫 SELECT가 발생했을시의 시점)의 데이터 스냅샷을 참조”** 함으로 트랜잭션 내에서 같은 SELECT 쿼리를 여러 번 실행하면 항상 같은 결과를 반환한다. 이는 **Phantom Read**를 방지한다.

- 그러나 **UPDATE** 쿼리는 트랜잭션이 진행되는 동안 **"다른 트랜잭션이 커밋한 데이터를 반영" 할 수 있다. **즉, UPDATE 쿼리는 최신 데이터를 기준으로 업데이트를 수행할 수 있다.

### 예시를 들어보자

```sql
CREATE TABLE records (
    id INT PRIMARY KEY,
    count INT
);

INSERT INTO records (id, count) VALUES (1, 1), (2, 2), (3, 3);
```

| id  | count |
| --- | ----- |
| 1   | 1     |
| 2   | 2     |
| 3   | 3     |

### 예시 1

순서를 잘 따라와보자

```sql
# 트랜잭션 A
START TRANSACTION; #1
SET SQL_SAFE_UPDATES = 0; #3
SELECT * FROM records; #6
UPDATE records SET count = 300 WHERE count = 2; #7
COMMIT; #8

# 트랜잭션 B
START TRANSACTION; #2
UPDATE records SET count = 2 WHERE id = 1; #4
COMMIT; #5
```

1, 2 : 트랜잭션 A → B 시작

3 : key가 아닌 칼럼 (count)에 대해서 update를 하기 때문에 SQL_SAFE_UPDATES 모드를 OFF

4 : 트랜잭션 B에서 업데이트

| id  | count      |
| --- | ---------- |
| 1   | 2 (update) |
| 2   | 2          |
| 3   | 3          |

6: 트랜잭션 B 커밋 후 A에서 조회하였음, 첫 조회임으로 update 내용이 반영되어 SELECT됨

트랜잭션 B 커밋 후 SELECT

| id  | count                        |
| --- | ---------------------------- |
| 1   | 2 (트랜잭션 B의 update 반영) |
| 2   | 2                            |
| 3   | 3                            |

7 : 트랜잭션 B에서 업데이트 내용을 커밋 완료 후 트랜잭션 A에서 count = 2인 레코드를 300으로 업데이트를 한다면 id=1인 레코드도 업데이트가 될까?

| id  | count                          |
| --- | ------------------------------ |
| 1   | 300 (트랜잭션 A의 update 반영) |
| 2   | 300                            |
| 3   | 3                              |

된다!

### 예시 2

그렇다면 트랜잭션 B가 업데이트 하기 전에 트랜잭션 A에서 SELECT을 미리 했다면 어떻게 될까? 트랜잭션 A에서 추가된 SELECT문에 집중해보자.

```sql
# 트랜잭션 A
START TRANSACTION; #1
SET SQL_SAFE_UPDATES = 0; #3

SELECT * FROM records; #4 (추가된 SELECT문, 아이솔레이션 시작)

SELECT * FROM records; #7
UPDATE records SET count = 300 WHERE count = 2; #8
COMMIT; #9

# 트랜잭션 B
START TRANSACTION; #2
UPDATE records SET count = 2 WHERE id = 1; #5
COMMIT; #6
```

4: 트랜잭션 B가 UPDATE하기 전에 SELECT를 하면서 아이솔레이션이 시작된다.

7 : 트랜잭션 B가 업데이트 후 커밋까지 해도 이보다 먼저 조회하였음으로, 트랜잭션 A가 첫 SELECT를 하는 시점 (4번 시점)부터 REPETABLE READ를 보장하기 위해 아이솔레이션이 시작된다. 따라서 7번 시점은 실제 데이터베이스가 아닌 스냅샷에서 데이터를 조회해온다.

**트랜잭션 A 첫 SELECT (아이솔레이션 시작)**

| id  | count |
| --- | ----- |
| 1   | 1     |
| 2   | 2     |
| 3   | 3     |

**트랜잭션 A 두번째 SELECT (스냅샷 참조)**

| id  | count                       |
| --- | --------------------------- |
| 1   | 1 (트랜잭션 B UPDATE 반영X) |
| 2   | 2                           |
| 3   | 3                           |

즉, 트랜잭션 A에서 SELECT를 하면 이미 id가 1인 칼럼은 1로 보이게 된다. 그렇다면, 이 경우에서도 `UPDATE records SET count = 300 WHERE count = 2` 는 동일하게 동작할까?

| id  | count                          |
| --- | ------------------------------ |
| 1   | 300 (트랜잭션 A UPDATE 반영!!) |
| 2   | 300                            |
| 3   | 3                              |

놀랍게도 동일하게 동작한다!

**즉, 트랜잭션 A가 특정 시점에 SELECT 쿼리를 실행하여 `id = 1`인 행의 `count` 값이 1로 보일지라도, UPDATE 쿼리는 현재 데이터베이스 상태를 기준으로 작동하게 된다.**
