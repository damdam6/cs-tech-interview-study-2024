# MongoDB 자세히 알아보기 : 초안

Date: June 14, 2024 12:00 AM (GMT+9) → 8:00 PM

![Untitled](MongoDB%20%E1%84%8C%E1%85%A1%E1%84%89%E1%85%A6%E1%84%92%E1%85%B5%20%E1%84%8B%E1%85%A1%E1%86%AF%E1%84%8B%E1%85%A1%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%20%E1%84%8E%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20945355ad11444cc58599f0a23e5fbc25/Untitled.png)

### MongoDB는 무엇인가?

![Untitled](MongoDB%20%E1%84%8C%E1%85%A1%E1%84%89%E1%85%A6%E1%84%92%E1%85%B5%20%E1%84%8B%E1%85%A1%E1%86%AF%E1%84%8B%E1%85%A1%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%20%E1%84%8E%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20945355ad11444cc58599f0a23e5fbc25/Untitled%201.png)

- MongoDB
  - 대표적인 NoSql 데이터 베이스
  - JSON 형식 (저장 방식은 Binary JSON)
  - 컬렉션(테이블) / 도큐먼트

---

### Scalability : Sharding

![Untitled](MongoDB%20%E1%84%8C%E1%85%A1%E1%84%89%E1%85%A6%E1%84%92%E1%85%B5%20%E1%84%8B%E1%85%A1%E1%86%AF%E1%84%8B%E1%85%A1%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%20%E1%84%8E%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20945355ad11444cc58599f0a23e5fbc25/Untitled%202.png)

- 수평 확장
  - 하나의 컬렉션을 여러 개의 서버에 나누어 저장 가능
  - 응용 프로그램에 변경 없음
  - 실시간 확장 및 rebalance 작업
  - 다양한 옵션 제공 (shard 분류를 위한 옵션)

_— 기존 DB :수직 확장 ⇒ 더 강력한 CPU를 사용하거나, RAM을 추가하거나, 스토리지 공간을 늘리는 등 단일 서버의 용량을 늘리는 것이 포함 ⇒ 실질적인 최대치 존재_

— 수평확장 ⇒ 시스템 데이터 세트와 로드를 여러 서버로 나누고 필요에 따라 서버를 추가하여 용량을 늘리는 것

- Shard

  - 데이터를 여러 머신에 분산하는 방법
  - 샤딩을 사용하여 대규모 데이터 세트와 높은 처리량 작업의 배포를 지원

- [`mongos`](https://www.mongodb.com/ko-kr/docs/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 는 특정 샤드 또는 샤드 집합을 쿼리 대상으로 지정 가능
  ⇒ 읽기/쓰기 효율화
- 샤딩 사용 예시

  - Nike와 같은 브랜드
    - 온라인/오프라인 데이터의 통합 컬렉션 사용
    - 기존의 RDBMS
      - 하나의 데이터로 통합 어려움
    - Shard를 무엇으로 기준점을 정하느냐에 사용 방식은 무궁무진
  - 서버의 확장이 쉬움
    - ex) 새로운 지역에 서비스 시작 - 신규 판매처 관리
    - mongo : shard를 통해 해결 가능
    - DB 서버를 물리적으로 늘릴 필요가 없다

- 단계

  - mongos : 클라이언트와 연결된 인스턴스
    - 클라이언트의 요청을 받음
    - 샤딩 키를 기반으로 샤드에 요청을 보냄
    - 쿼리 라우터 역
  - config servers : 샤드 클러스터에 대한 메타데이터 구성 및 설정 저장
  - shard :
    ⇒

- 추가 용어 및 구조
  - 샤드 키
    - 데이터의 분포를 결정하는 기준
    - 데이터의 논리적 구조
  - 청크
    - 실질적으로 그룹화 된 데이터
    - 샤드 키 값의 범위에 따라 묶음으로 지정
    - 청크를 기준으로 데이터를 샤드에 분배 - 128MB가 기준
    - 데이터 파티셔닝 개념

![Untitled](MongoDB%20%E1%84%8C%E1%85%A1%E1%84%89%E1%85%A6%E1%84%92%E1%85%B5%20%E1%84%8B%E1%85%A1%E1%86%AF%E1%84%8B%E1%85%A1%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%20%E1%84%8E%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20945355ad11444cc58599f0a23e5fbc25/Untitled%203.png)

---

### Replica Sets

- Replica Sets란?

  - 동일한 데이터 세트를 유지하는 mongodb 프로세스 그룹
  - 중복성 - 고가용성
  - primary - secondary 구조
  - 예외 처리시 읽기/ 쓰기에 대한 재시도
  - 조정 가능한 Durability & Consistency
    - Read/Write 옵션 설정 가능
  - 데이터 센터 간 Replica Set 구성을 통한 데이터 안정성 확보

- 구조
  ![Untitled](MongoDB%20%E1%84%8C%E1%85%A1%E1%84%89%E1%85%A6%E1%84%92%E1%85%B5%20%E1%84%8B%E1%85%A1%E1%86%AF%E1%84%8B%E1%85%A1%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%20%E1%84%8E%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20945355ad11444cc58599f0a23e5fbc25/Untitled%204.png)
  ![Untitled](MongoDB%20%E1%84%8C%E1%85%A1%E1%84%89%E1%85%A6%E1%84%92%E1%85%B5%20%E1%84%8B%E1%85%A1%E1%86%AF%E1%84%8B%E1%85%A1%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%20%E1%84%8E%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20945355ad11444cc58599f0a23e5fbc25/Untitled%205.png)
  - Heartbeat : 현재 상태에 대한 공유 - 장애 있없 여부 판단
  ![Untitled](MongoDB%20%E1%84%8C%E1%85%A1%E1%84%89%E1%85%A6%E1%84%92%E1%85%B5%20%E1%84%8B%E1%85%A1%E1%86%AF%E1%84%8B%E1%85%A1%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%20%E1%84%8E%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20945355ad11444cc58599f0a23e5fbc25/Untitled%206.png)
  - 장애 감지 시 → Secondary 중 하나가 Primary로 지정 → 무중단 서비스

![Untitled](MongoDB%20%E1%84%8C%E1%85%A1%E1%84%89%E1%85%A6%E1%84%92%E1%85%B5%20%E1%84%8B%E1%85%A1%E1%86%AF%E1%84%8B%E1%85%A1%E1%84%87%E1%85%A9%E1%84%80%E1%85%B5%20%E1%84%8E%E1%85%A9%E1%84%8B%E1%85%A1%E1%86%AB%20945355ad11444cc58599f0a23e5fbc25/Untitled%207.png)

- Hedged Read
  - 읽기 성능 최적화 기능
  - 단순히 Primary하나에서 읽는게 아닌 옵션 설정 기능
  - Secondary에 동시 접근 → 가장 빠른 응답을 반환

---

### 참고자료

https://www.mongodb.com/ko-kr/docs/manual/sharding/

https://www.mongodb.com/ko-kr/docs/manual/core/transactions/

https://www.mongodb.com/ko-kr/docs/manual/core/sharding-data-partitioning/

https://www.youtube.com/watch?v=qiElyjkUrKY

https://jh2021.tistory.com/24

https://steady-coding.tistory.com/610

https://hongong.hanbit.co.kr/데이터베이스-이해하기-databasedb-dbms-sql의-개념/
