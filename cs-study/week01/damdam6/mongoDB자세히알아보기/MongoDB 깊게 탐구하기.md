# MongoDB 깊게 탐구하기 : 최종

## MongoDB - NoSQL

- MongoDB
    - 대표적인 NoSql 데이터 베이스
    - JSON 형식 (저장 방식은 Binary JSON)
    - 컬렉션(테이블) / 도큐먼트

- NoSQL 개념
    - 비관계형 데이터 베이스 유형
    - 사전에 스키마 지정할 필요 없음 (유연성)
    - 신속한 수평적 확장 - 높은 트래픽 처리에 적합 (확장성)
    - 고성능
        - 분산 저장 / 샤딩 - 독립적 작동을 통한 병목 현상 해소
        - 캐싱
        - 자동 복제 저장
    
    ** 프로젝트에서 MongoDB에 저장했던 데이터 형식
    
    ![Untitled](MongoDB 깊게 탐구하기 이미지/Untitled.png)
    
    ![Untitled](MongoDB 깊게 탐구하기 이미지/Untitled%201.png)
    
    ![Untitled](MongoDB 깊게 탐구하기 이미지/Untitled%202.png)
    
- NoSQL 대표 : REDIS
    - 데이터를 RAM에 저장 - 메모리에서 직접 액세스
    - 캐싱의 개념으로 많이 사용
    - 키-값 페어로 저장 : 다양한 데이터 유형 제공
    - 키 길이의 한계 있음(512MB)
    
    ![Untitled](MongoDB 깊게 탐구하기 이미지/Untitled%203.png)
    
    ![Untitled](MongoDB 깊게 탐구하기 이미지/Untitled%204.png)
    
- NoSQL 대표:  MongoDB
    - 외부 메모리 스토리지에 데이터 저장
    - 최대 문서 크기 16MB

- Redis vs MongoDB
    - 기본적으로 Redis는 확장성 지원 X
    - mongoDB - 복제의 강력한 기능 (사본-분산-장애조치 등) Redis엔 없다
    - Redis는 내장 ACID 지원 기능X
    - Redis - 키:값 빠른 탐색 / MongoDB는 비교적 복잡한 쿼리도 가능

---

### MongoDB  - 장점을 기능하게 하는 요소

- 대표적인 장점
    - 유연성
    - 확장성
    - 성능

### Sharding

![Untitled](MongoDB 깊게 탐구하기 이미지/Untitled%205.png)

- 수평 확장
    - 하나의 컬렉션을 여러 개의 서버에 나누어 저장 가능
    - 응용 프로그램에 변경 없음
    - 실시간 확장 및 rebalance 작업
    - 다양한 옵션 제공 (shard 분류를 위한 옵션)
    - [`mongos`](https://www.mongodb.com/ko-kr/docs/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 는 특정 샤드 또는 샤드 집합을 쿼리 대상으로 지정 가능
        
        ⇒ 읽기/쓰기 효율화
        
    
- 단계
    - mongos : 클라이언트와 연결된 인스턴스
        - 클라이언트의 요청을 받음
        - 샤딩 키를 기반으로 샤드에 요청을 보냄
        - 쿼리 라우터 역
    - config servers : 샤드 클러스터에 대한 메타데이터 구성 및 설정 저장
    - shard :  데이터 탐색 위치
    - chunk : 파티션 개념 - 샤드 간의 데이터 불균형 해소 목적

![Untitled](MongoDB 깊게 탐구하기 이미지/Untitled%206.png)

*— 기존 DB :수직 확장 ⇒ 더 강력한 CPU를 사용하거나, RAM을 추가하거나, 스토리지 공간을 늘리는 등 단일 서버의 용량을 늘리는 것이 포함 ⇒ 실질적인 최대치 존재*

— 수평확장 ⇒ 시스템 데이터 세트와 로드를 여러 서버로 나누고 필요에 따라 서버를 추가하여 용량을 늘리는 것

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

---

### Replica Sets

- Replica Sets란?
    - 동일한 데이터 세트를 유지하는 mongodb 프로세스 그룹
    - 중복성 - 고가용성
    - primary - secondary 구조
    - 예외 처리 시 읽기/ 쓰기에 대한 재시도
    - 조정 가능한 Durability & Consistency
        - Read/Write 옵션 설정 가능
    - 데이터 센터 간 Replica Set 구성을 통한 데이터 안정성 확보

- 구조
    
    ![Untitled](MongoDB 깊게 탐구하기 이미지/Untitled%207.png)
    
    ![Untitled](MongoDB 깊게 탐구하기 이미지/Untitled%208.png)
    
    - Heartbeat : 현재 상태에 대한 공유 - 장애 있없 여부 판단
    
    ![Untitled](MongoDB 깊게 탐구하기 이미지/Untitled%209.png)
    
    - 장애 감지 시 → Secondary 중 하나가 Primary로 지정 → 무중단 서비스

![Untitled](MongoDB 깊게 탐구하기 이미지/Untitled%2010.png)

- Hedged Read
    - 읽기 성능 최적화 기능
    - 단순히 Primary하나에서 읽는 게 아닌 옵션 설정 기능
    - Secondary에 동시 접근 → 가장 빠른 응답을 반환

- ACID - 트랜잭션 서비스 제공 (replica set 기반)
    - 4.0에 서비스 업뎃
    
    하단부 모두 지원 - 단, 성능적 문제는 발생할 수 있음.
    
    - 원자성
    - 일관성
    - 격리성
    - 지속성

---

### 참고 자료

https://aws.amazon.com/ko/compare/the-difference-between-redis-and-mongodb/