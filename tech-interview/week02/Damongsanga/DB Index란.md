# [스터디 CS] DB Index에 대해 설명해주세요

태그: DB, 스터디
날짜: 2024년 6월 20일

# DB 인덱스

## 왜 인덱스를 사용하나요?

- 인덱스가 없다면 full scan이 필요함 ( O(N) )
- 인덱스가 있다면 B-Tree 기준 ( O(logN) )
- 빠르게 조회, 정렬, 그룹하기 위해 사용

## **언제, 어디에 인덱스를 적용하면 좋은가요? (인덱스 적용 기준)**

**인덱스를 적용하면 좋은 경우**

1. 카디널리티가 높은 (중복도 낮은) 컬럼
2. WHERE, JOIN, ORDER BY 절에 자주 사용되는 컬럼 (조건절이 없으면 인덱스는 사용 안됨)
3. CUD가 자주 발생하지 않는 컬럼
4. 규모가 큰 테이블

**오히려 Full Scan 이 좋은 경우**

1. WHERE 절이나 ON 절에 인덱스를 이용할 수 있는 적절한 조건이 없는 경우
2. 테이블에 데이터가 몇십, 몇백건으로 적은 경우 (테이블이 1개의 페이지만으로 구성될정도로 작은 경우)
3. 조회하려는 데이터가 테이블의 상당 부분을 차지할 때
   - 예시로 조건이 column 값이 a,b,c,d로 4개밖에 없는데 `SELECT * FROM table WHERE column = “a”`인 경우

주의할 점

- order by, group by에도 index 사용될 수 있음
- foreign key에는 index가 자동으로 생성되지 않을 수 있음 (MYSQL은 자동생성되나 DBMS에 따라 다르다, join 관련하여 영향을 주기 때문)
- 이미 데이터가 몇백만건 이상 있는 테이블에 index를 새로 생성하는 것은 매우 부담이 큰 작업이다
- **`LIKE`와 사용할 경우에는 %가 뒤에 사용되도록 하기. (앞에 사용되면 `Full Scan`)**

## 왜 무조건 Index를 사용하는 것이 좋은게 아닌가요?

- 인덱스를 통해 데이터를 조회하는 것은 아래의 2가지 작업이 수행
  1. **인덱스를 통해 PK를 찾음**
  2. **PK를 통해 레코드를 찾음**
- 이러한 이유로 옵티마이저는 인덱스를 통해 레코드 1건을 읽는 것이 테이블을 통해 직접 읽는 것 보다 4~5배 정도 비용이 더 많이 드는 것으로 예측
  - 하지만 DBMS는 우리가 원하는 레코드가 어디있는지 모르기에 모든 테이블을 뒤져서 레코드를 찾아야하고, 이는 엄청난 디스크 읽기 작업이 필요하므로 상당히 느림
- 하지만 인덱스를 사용한다면 인덱스를 통해 PK를 찾고, PK를 통해 레코드를 저장된 위치에서 바로 가져올 수 있으므로 디스크 읽기가 줄어들게 된다. 그렇기 때문에 레코드를 찾는 속도가 훨씬 빠르며, 이것이 인덱스를 사용하는 이유
- 반면에 인덱스를 타지 않는 것이 효율적일 수도 있음. **인덱스를 통해 레코드 1건을 읽는 것이 4~5배 정도 비싸기 때문에, 읽어야 할 레코드의 건수가 전체 테이블 레코드의 20~25%를 넘어서면 인덱스를 이용하지 않는 것이 효율적**이다. 이런 경우 옵티마이저는 인덱스를 이용하지 않고 테이블 전체를 읽어서 처리한다.

## 인덱스의 종류는 무엇이 있나요? (클러스터링 / 논 클러스터링)

- **인덱스 성능에서** 페이지 분할이 가장 큰 비용이다
- **클러스터링 인덱스**
  - **실제 데이터 자체가 정렬**
  - **테이블당 1개만 존재 가능**
  - 리프 페이지가 데이터 페이지
  - PK > UNIQUE + NOT NULL 조건으로 클러스터링 인덱스에 적합
- **논 클러스터링 인덱스**
  - **실제 페이지는 그대로, 별도의 인덱스 페이지 생성 (column, pointer)**
  - **약 10%의 추가 공간 필요하며 테이블당 여러개 존재 가능하다**
  - 리프 페이지에 실제 데이터의 페이지 주소를 담고 있음
  - **UNIQUE 제약조건 적용시 자동 생성**
  - **직접 index 생성**하면 논 클러스터링 인덱스를 만든다
  - 클러스터링 인덱스보다 R는 조금 느리지만 CUD가 빠르다. (인덱스보다 페이지 분할이 적게 일어남)
- **클러스터링 with 논클러스터링 인덱스 (같이 사용하면?)**
  - 논 클러스터링 인덱스에 페이지 주소가 아닌 클러스터링 키를 가지게 된다
  - 이유 : CUD에 의한 클러스터링 인덱스의 페이지 분할이 발생하면 논클러스터링 인덱스의 페이지 주소변경이 많이 일어나기 때문이다. 이를 방지하기 위함.

## 사용 예시?

**[클러스터링 인덱스]**

경우에 따라서는 기본키가 아닌 다른 컬럼을 클러스터링 인덱스로 지정할 수도 있습니다. 예를 들어, 기본키가 자동 증가하는 숫자 ID인 경우, 실제 검색에 많이 사용되는 다른 컬럼(예: 주문일자)을 클러스터링 인덱스로 지정하는 것이 더 효율적일 수 있습니다.

**[논 클러스터링 인덱스]**

1. **고유성이 높은 컬럼(Cardinality가 높은 컬럼)**
   - 예: 사용자 ID, 주문번호, 제품코드 등
   - 고유성이 높은 컬럼은 중복된 값이 적어 인덱스 효율이 좋습니다.
2. **자주 검색/조회되는 컬럼**
   - 예: 고객명, 주문일자, 배송상태 등
   - 자주 검색/조회되는 컬럼은 인덱스를 통해 빠른 검색이 가능합니다.
3. **범위 검색에 사용되는 컬럼**
   - 예: 주문일자, 가입일자, 생년월일 등
   - 범위 검색 시 인덱스를 활용하면 성능 향상을 기대할 수 있습니다.
4. **조인에 사용되는 컬럼**
   - 예: 고객ID, 상품ID, 부서ID 등
   - 조인 시 인덱스를 활용하면 조인 성능을 크게 향상시킬 수 있습니다.
5. **데이터 변경이 적은 컬럼**
   - 예: 주소, 성별, 직책 등
   - 데이터 변경이 적은 컬럼은 인덱스 관리가 용이합니다.

## Covering Index란 무엇인가요?

- 조회하려는 column이 index 내의 데이터만으로 모두 cover가 가능할 때 이 index를 covering index라고 한다
  - **쿼리에 사용되는 모든 열(컬럼)을 인덱스에 포함시킵니다.**
- 일반적으로는 인덱스 기반으로 실제 테이블을 조회하여 찾으나, 쿼리에 필요한 모든 열이 인덱싱 되어있다면, 인덱스만으로도 쿼리 결과를 얻을 수 있다.
- 본 테이블까지 찾을 필요가 없기 때문에 조회 성능이 매우 빠르다
- 따라서 성능 개선을 위해 의도적으로 만들기도 한다.

## DB 인덱스 구현을 위해서는 어떤 자료구조를 사용하나요?

- 구현 자료구조
  - Binary Search Tree : 이진 탐색 트리!
  - B Tree : 한 노드에 한개 이상의 값을 가지게 하기 (한번의 스캔으로 탐색할 수 있는 범위 증가)
  - B+Tree : 리프노드에만 값을 가지게 하고 나머지 노드들에는 가이드라인만 가지게 하기 + 리프노드 간의 탐색 추가 (범위 검색에 매우 유리)

### 0. **Hash Index (Hash Table)**

- hash table을 사용하여 index를 구현
- 조회, 삽입, 삭제에 대한 시간복잡도가 O(1)로 매우 성능이 좋다
- 단점
  - rehashing에 대한 부담 (크기가 가득차서 동적으로 늘려줄 때의 부담)
  - 범위 비교가 불가능하다 (equality 비교만 가능)
  - multicolumn index의 경우 전체 attributes에 대한 조회만 가능
    - (A, B) 인덱스인 경우 A만으로 검색할 때 hash Index는 안됨

### 1. 이진 탐색 트리

- 이진 탐색의 장점과 연결 리스트의 장점을 결합
- **균형 문제 발생시 O(N)으로 수렴**

### 2. B-Tree

- 자식 노드가 2개 이상인 트리
- 이진탐색트리처럼 왼쪽이 작은 값, 오른쪽이 큰 값이지만, balance가 맞춰져있음
- 최악의 경우에도 O(logN)
- **중위순회**를 하여 모든 데이터 순회시 **트리의 모든 노드를 방문해야 하는 문제**가 있음

[Visualization](https://www.cs.usfca.edu/~galles/visualization/BTree.html)

### 3. B+Tree ⇒ Inno DB

- 특징
  - **leaf node에만 데이터를 저장**하고 나머지 노드는 자식 포인터만 저장하도록 한다
  - 그리고 **leaf node까리 linkedList로 연결되어있다 (InnoDB는 doublely linked list)**
- 장점
  - leaf node에만 데이터를 저장함으로 메모리 확보
  - 하나의 node에 더 많은 포인터를 가질 수 있어 트리의 높이가 낮아져 검색속도 증가
  - **full scan의 경우 linkedLsit를 사용하여 선형 시간 소모**
- 단점
  - **b-tree는 최악의 경우가 O(logN)인데에 비해 B+Tree는 항상 O(logN) 시간**
    - **특정 key에 접근하려면 반드시 leaf-node로 가야함**으로
    - b-tree는 자주 접근하는 데이터를 위쪽 노드에 두면 더 빨리 찾지만 B+Tree는 불가능

[Visualization](https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html)

### 4. Red Black Tree ⇒ 왜 DB 인덱스로 사용하지 않는가!!

- **각 노드는 하나의 데이터만 가짐**
- 특징
  - 좌, 우 자식 노드의 개수 밸런스를 맞추고 검은색, 빨간색 노드는 재 정렬을 위해 사용
  - 데이터 접근 시 포인터 필요
  - 루트 노드와 리프노드(NIL : null leaf, 자료를 갖지 않고 트리의 끝을 나타내는 노드)는 항상 검은색
  - 새로운 노드는 항상 빨간색
  - 빨간 노드의 자식은 모두 검은색 (No Double Red)
    - **Restructuring (삼촌 노드가 검은색이거나 없는 경우)**
    - **Recoloring (삼촌 노드가 빨간색인 경우)**
    - [**관련 글**](https://code-lab1.tistory.com/62)
  - 모든 리프 노드에서 Black Depth(리프~루트에서의 검은색 노드 갯수)는 같다.
- B-Tree와의 공통점
  - 밸런스 유지. O(logN)
- B-Tree와의 차이점
  - **참조 포인터 접근 수로 인하여 Index로는 사용되지 않는다**
  - 하나의 노드가 가지는 데이터 수가 다름
  - B-Tree는 배열로 실제 메모리 상에도 차례대로 저장 되어있음 ⇒ 실제 메모리 디스크에서 다음 인덱스에 접근 하는 식으로 빠름
  - 그러나 Red-Black Tree는 각 노드가 개별 참조 포인터로 접근하기 때문에 **속도가 느림**
    - 주소를 알아내기 이해 내부적으로 추가 연산이 필요
      ![Untitled](./DB%20Index란/Untitled.png)
      - 첫 바이트 블럭 주소 이동 연산
      - 그 주소에서 x Byte 만큼 주소를 더하는 덧셈 연산
      - 실제 배열에 넣을 값을 대입하는 연산 수행
  - 사용 예시
    - Java8 이후 TreeMap
    - Java HashMap 8개 키-값 쌍 이상인 경우 링크드리스트 ⇒ RBT로 변경
      - 6개 이하면 링크드리스트로 원복
    - 리눅스 커널에서 많이 사용된다고 함

[Visualiztaion](https://www.cs.usfca.edu/~galles/visualization/RedBlack.html)

## 왜 다른 자료구조가 아닌 B-Tree (B+Tree) 자료구조를 사용하나요?

- 왜 배열 X?
  - 조회는 O(1)로 빠르지만 **저장, 수정, 삭제시 월등히 느림**
- 왜 해시테이블 X?
  - 데이터 내부 정렬이 되어있지 않아 등호 연산만 가능하고 **부등호 연산이 불가능** ⇒ 인덱스 특성상 크기 비교가 필요하기 때문
- 왜 RBT X?
  - 시간 복잡도는 O(logN)으로 동일하지만 **노드에 1개의 값만 존재하여 참조 포인터 접근수가 더 많아** 탐색에 불리하다
  - RedBlack-Tree는 무조건 하나의 노드에 하나의 데이터 요소만을 저장하므로 어떠한 요소를 탐색하든 참조 포인터 접근이 필수적이다. 반면, B-Tree는 하나의 노드에 여러 개의 데이터를 저장하므로 각 노드의 데이터 요소를 탐색할 때 참조 포인터 접근 없이 배열의 성질을 이용하여 빠르게 탐색이 가능하다. 결론적으로 참조 포인터의 접근 수가 B-Tree가 훨씬 적으므로 B-Tree를 인덱스의 자료 구조로 사용한다.
  - 여기서 배열의 성질이란 **B+Tree의 리프 노드가 연결 & 한 노드에 여러 값을 저장, 이 값들은 실제 메모리에도 순차적으로 저장되어있음으로 참조 없이 순차 탐색 가능**

## 공간복잡도 적으로 가장 유리한 자료구조는?

B-Tree, B+Tree, Red-Black Tree 중에서 공간복잡도 측면에서 가장 효율적인 것은 B+Tree입니다. B+Tree는 내부 노드에 데이터를 저장하지 않고 포인터만 저장하므로, 동일한 데이터를 저장할 때 B-Tree나 Red-Black Tree보다 공간을 더 효율적으로 사용할 수 있습니다.

1. **B-Tree**:
   - B-Tree는 각 노드에 데이터와 포인터가 모두 저장
   - 따라서 노드 하나당 데이터와 포인터를 모두 저장해야 하므로 공간복잡도가 상대적으로 높습니다.
2. **B+Tree**:
   - **B+Tree는 리프 노드에만 데이터가 저장되고, 내부 노드에는 포인터만 저장**
   - 따라서 내부 노드에는 데이터를 저장할 필요가 없어 공간복잡도가 B-Tree에 비해 효율적입니다.
3. **Red-Black Tree**:
   - Red-Black Tree는 자가 균형 이진 탐색 트리의 일종입니다.
   - **각 노드에는 데이터와 2개의 포인터(left, right)가 저장**됩니다.
   - B+Tree에 비해 공간복잡도가 다소 높으나 B-Tree보다는 효율적

## +a) 인덱스는 어떻게 만드나요?

1. **기존 테이블에서 인덱스 만들기**
   - Not unique 한 인덱스 만들기
     - `CREATE INDEX player_name_idx ON player (name);`
   - Unique한 복합 인덱스 만들기
     - `CREATE UNIQUE INDEX team_id_backnumber_idx ON player (team_id, backnumber);`
     - **복합 인덱스인 경우 왼쪽 column이 우선순위를 가진다**
       - 따라서 team_id 를 기준으로 찾을 때 이 인덱스를 사용할 수 있다
       - 하지만 backnumber 만 기준으로 찾을 때는 위 인덱스를 활용할 수 없다
2. **테이블 생성 시 인덱스 만들기**

   - PK에는 index가 자동으로 생성된다

   ```sql
   - CREATE TABLE player (
       …
       …
       INDEX player_name_idx (name),  // 여기서 인덱스 이름 생략 가능
       UNIQUE INDEX (team_id, backnumber)
       );
   ```

   - Index 보기
     ```sql
     SHOW INDEX FROM player;
     ```
   - 특정 Index 를 사용하도록 요청 (원래는 optimizer가 적절한 index를 찾는다)
     ```sql
     SELECT * FROM player USE INDEX (backnumber_idx) // 권고
     SELECT * FROM player FORCE INDEX (backnumber_idx) // 해당 index를 사용하도록 강제. 그러나 optimizer가 이 인덱스를 사용하는게 더 나쁘다고 판단하면 full scan
     ```

### 참조

https://www.youtube.com/watch?v=IMDH4iAQ6zM&t=510s

https://steady-coding.tistory.com/558

[https://velog.io/@jewelrykim/Binary-Search-Tree에서-BTree까지Database-Index-추가](https://velog.io/@jewelrykim/Binary-Search-Tree%EC%97%90%EC%84%9C-BTree%EA%B9%8C%EC%A7%80Database-Index-%EC%B6%94%EA%B0%80)

[https://velog.io/@semi-cloud/MySQL-B-Tree-인덱스-구조와-인덱스-스캔](https://velog.io/@semi-cloud/MySQL-B-Tree-%EC%9D%B8%EB%8D%B1%EC%8A%A4-%EA%B5%AC%EC%A1%B0%EC%99%80-%EC%9D%B8%EB%8D%B1%EC%8A%A4-%EC%8A%A4%EC%BA%94)

https://steady-coding.tistory.com/558

[https://velog.io/@chosj1526/DB-Index-개념-장단점-자료구조](https://velog.io/@chosj1526/DB-Index-%EA%B0%9C%EB%85%90-%EC%9E%A5%EB%8B%A8%EC%A0%90-%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0)

[https://velog.io/@sweet_sumin/클러스터드-인덱스-Clustered-Index-넌-클러스터드-인덱스-Non-Clustered-Index](https://velog.io/@sweet_sumin/%ED%81%B4%EB%9F%AC%EC%8A%A4%ED%84%B0%EB%93%9C-%EC%9D%B8%EB%8D%B1%EC%8A%A4-Clustered-Index-%EB%84%8C-%ED%81%B4%EB%9F%AC%EC%8A%A4%ED%84%B0%EB%93%9C-%EC%9D%B8%EB%8D%B1%EC%8A%A4-Non-Clustered-Index)

[https://mangkyu.tistory.com/286](https://mangkyu.tistory.com/286) ⇒ 매우 자세히 설명되어있음

https://bo5mi.tistory.com/212
