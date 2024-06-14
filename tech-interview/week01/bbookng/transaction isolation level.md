# 📌 Transaction Isolation Level

## ✏️ Transaction 들이 동시에 실행될 때 발생 가능한 이상 현상들

### 1) Dirty Read
- **commit 되지 않은 변화를 읽음.**
- 예시
	- **x = 10, y = 20 일 경우**
	- **transaction1 : x += y** 
	- **transaction2 : y = 70**
	- transaction1에서 read(x) = 10 을 하고, transaction2에서 write(y) = 70을 동시에 했을 경우 transaction1 에서 read(y) = 70, write(x) = 80이 되는데, 이 때 commit 되지 않은 transaction2를 읽어 값이 이상해진 경우 dirty read 발생

### 2) Non-Repeatable Read (=Fuzzy Read)
- **같은 데이터의 값이 달라짐.**
- 예시
	- **x = 10 일 경우**
	- **transaction1 : x를 두 번 읽는다.**
	- **transaction2 : x += 40**
	- transaction1이 먼저 실행되어 read(x) = 10 이 되고, transaction2가 실행되어 read(x) = 10, write(x) = 50이 실행된 후 커밋을 한다. 다시 transaction1에서 read(x)를 했을 때 read(x) = 50이 된다. 같은 데이터를 한 transaction 내에서 두 번 읽었음에도 불구하고 서로 다른 값을 읽음. 
		- transaction isolation 속성에서 여러 transaction이 동시에 실행되어도 각각의 transaction이 혼자 실행되는 것 처럼 동작해야 한다는 것을 위배함.

### 3) Phantom Read 
- **없던 데이터가 생김.**
- 예시
	- **t1(..., v = 10), t2(..., v = 50) 일 경우**
	- **transaction1 : v가 10인 데이터를 두 번 읽는다.**
	- **transaction2 : t2의 v를 10으로 바꾼다.**
	- transaction1이 실행되어 read(v = 10)을 했을 경우 t1을 읽는다. 이후 transaction2가 실행되어 write(t2.v = 10)을 하고 커밋을 한다. 다시 transaction1이 실행되어 read(v = 10)을 하게 될 경우 t1, t2를 읽게 된다. 위와 마찬가지로 하나의 transaction에서 다른 값을 읽어옴.
		- transaction isolation 속성에서 여러 transaction이 동시에 실행되어도 각각의 transaction이 혼자 실행되는 것 처럼 동작해야 한다는 것을 위배함.
		
		

## ✏️ Transaction Isolation Level 
### - Isolation Level 이 필요한 이유
- 이런 이상한 현상들이 모두 발생하지 않게 만들 수 있지만 그러면 제약사항이 많아져서 동시 처리 가능한 트랜잭션 수가 줄어들어 결국 DB의 전체 처리량(throughput)이 하락하게 됨.
- 일부 이상한 현상은 허용하는 몇 가지 level을 만들어서 사용자가 필요에 따라서 적절하게 선택할 수 있도록 하자!

- 세 가지 이상 현상을 정의하고 어떤 현상을 허용하는지에 따라서 각각의 isolation level이 구분됨.
- 어플리케이션 설계자는 isolation level을 통해 전체 처리량(throughput)과 데이터 일관성 사이에서 어느 정도 trade를 할 수 있다. 

| Isolation level  | Dirty read | Non-repeatable read | Phantom read |
| ---------------- | :--------: | :-----------------: | :----------: |
| Read uncommitted |     O      |          O          |      O       |
| Read commited    |     X      |          O          |      O       |
| Repeatable read  |     X      |          X          |      O       |
| Serializable     |     X      |          X          |      X       |

### 1) Read uncommitted
- Dirty read, Non-repeatable read, Phantom read 허용.
- 가장 자유롭지만 이상 현상에 가장 취약함.
- 동시성은 높아져서 전체 처리량은 높음.

### 2) Read committed
- Non-repeatable read, Phantom read 허용.
- 커밋된 데이터만 읽기 때문에 Dirty read 는 허용하지 않음.

### 3) Repeatable read
- Phantom read 허용.

### 4) Serializable
- Dirty read, Non-repeatable read, Phantom read 모두 허용하지 않음.
- 아예 이상한 현상 자체가 발생하지 않는 level을 의미. 





## ✏️ 면접 질문

- 트랜잭션 격리 수준이란 무엇인가요 ?
  - 트랜잭션 격리 수준이 왜 필요한가요 ? 

- 트랜잭션 격리 수준의 종류를 설명해보세요.
  - 트랜잭션 격리 수준에 따라 발생할 수 있는 문제점에 대해 말해주세요.
    - Dirty Read, Non-repeatable Read, Phantom Read가 각각 어떤 상황에서 발생하는지 예를 들어 설명해보세요.
  - MySQL의 트랜잭션 격리 수준에 대해 설명해주세요. (Inno DB, gap lock)
- 트랜잭션에 대해서 설명해주세요. 
  - 트랜잭션의 특징(ACID)에 대해서 설명해주세요. 
    - 데이터베이스가 Durability를 보장하는 방식을 설명해주세요
  - Spring에서 `@Transactional` 의 동작 방식에 대해 설명해주세요.