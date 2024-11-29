# HashTable 해시테이블

## Hash

- Hash란?
    - 해시는 데이터를 다루는 기법이다.
    - Hash 값이란?
        - 다양한 길이를 가진 데이터를 고정된 길이를 가진 데이터로 매핑한 값이다.
    - 데이터를 검색할 때 사용하는 key와 실제 데이터 값 value가 한 쌍으로 존재함.
        - key값이 배열의 인덱스로 변환되어 검색과 저장에 평균 시간 복잡도가 O(1)에 수렴
    - 특징
        - 암호화 : 입력 받은 데이터를 전혀 다른 모습의 데이터로 변환함.
        - 데이터 축약 : 길이가 다른 입력에 대해 일정한 길이의 출력을 만들 수 있음.
- Hash function, Hash Alogrithm, Hash code
    - Hash function(algorighm)
        - key에 대한 해시값을 만드는 함수(알고리즘)
- Hash Table
    - 데이터를 담을 테이블을 미리 크게 확보해 놓은 후 입력받은 데이터를 해시하여 테이블 내의 주소를 계산하고 이 주소에 데이터를 담는 것
    - 해시를 통해 데이터가 담길 주소값을 얻는 것이 상수 시간 내에 결정됨.
        - 따라서 데이터 액세스(삽입, 삭제, 탐색)시 계산복잡성을 O(1)을 지향함.
    
    ↔ 이진탐색 트리와 그 변형
    
    - 메모리를 효율적으로 사용함
    - 계산복잡도는 O(logn)
    
    ↔ 배열
    
    - 계산 복잡도는 O(1)
    - 메모리를 미리 할당해야 해서 공간이 비효율적

### Hash Collision 해시 충돌

![./image/image.png](image.png)

- 해시 충돌이란?
    - 해시함수는 해시값의 갯수보다 많은 키값을 해시값으로 변환하기 때문에 해시함수가 서로 다른 두 개의 키에 대해 동일한 해시값을 내는 해시충돌이 발생한다.
        - many-to-one 대응

- Direct-address table
    
    ![image.png](./image/image%201.png)
    
    - 키의 전제 개수와 동일한 크기의 버킷을 가진 해시 테이블
    - 전체 키가 실제 사용하는 키보다 훨씬 많은 경우 사용하지 않는 키들을 위한 공간까지 마련해야함. ⇒ 메모리 효율성이 떨어짐.
    
    ⇒ 보통 해시테이블의 크기(m)가 실제 사용하는 키 개수(n) 보다 적은 해시테이블을 운용함.
    
    - load factor = n/m
    - load factor가 1보다 크면 해시 충돌이 발생

### 해시 충돌 해결법 - Chaining

- Chaining
    - 한 버킷당 들어갈 수 있는 엔트리의 수에 제한을 두지 않음.
        - 해당 버킷에 데이터가 이미 있으면 체인처럼 노드를 추가하여 다음 노드를 가리키는 방식으로 구현(Linked List)
    
    ![image.png](./image/image%202.png)
    
- 시간 복잡도 따지기 조건
    - 해시 테이블 크기 : m
    - 실제 사용하는 키 갯수 : n
    - 버킷 당 균일 분배되었다 가정 시 키 수 : n/m = a
    
- 삽입(inserrtion)
    - O(1) : head에 삽입할 시
- 탐색(search)
    - unsuccessful search : O(1+a)
    - succesful search
        - O(1+a)
- 삭제(delete)
    - 탐색과 유사. O(1+a)

### 해시 충돌 해결법 - open addressing

- open addressing
    - 한 버킷 당 들어갈 수 있는 엔트리가 하나
    - 비어 있는 해시 테이블의 공간을 활용하는 방법

- 대표적인 방식
    
    ![image.png](./image/image%203.png)
    
    - 선형 탐사 Linear Probing
        - 현재의 버킷 index로부터 고정폭 만큼씩 이동하여 차례대로 검색해 비어 있는 버킷에 데이터 저장
        - 탐사 이동폭이 고정돼 있는 선형탐사는 특정 해시값 주변 버킷이 모두 채워져 있는 primary clustring 문제에 취약함.
    - 제곱 탐사 Quadratic Probing
        - 해시의 저장순서 폭을 제곱으로 저장하는 방식.
        - 여러 개의 서로 다른 키들이 동일한 초기 해시값(아래에서 *initial probe*)을 갖는 secondary clustering에 취약함.
    
    - 이중 해싱 double hashing
        - 탐사할 해시값의 규칙성을 없애버려서 clustering을 방지하는 기법
        - 2개의 해시 함수를 준비
            - 최초의 해시값
            - 해시 충돌이 일어났을 때 탐사 이동폭
            
- 시간 복잡도 따지기 조건
    - 해시 테이블 크기 : m
    - 실제 사용하는 키 갯수 : n
    - 버킷 당 균일 분배되었다 가정 시 키(key) 수 : n/m = a
        - 단, 여기서는 a가 1과 같거나 작다고 가정함. (테이블이 꽉 차지 않는다는 걸 전제로 함.)

- 시간 복잡도 따지기
    - 탐사 횟수에 비례함. ⇒ 탐사 횟수의 기대값
    - successful search
        
        $$
        E[successful search] = \frac{1}{α} ln\frac{1}{(1−α)}
        $$
        
    - unsuccessful search
        
        $$
        E[unsuccessful search]=\frac{1}{α}
        $$
        
    
    ⇒ 수학적 확률 계산에 가까움…
    
    ⇒ a에 영향을 많이 받는다!
    

---

https://velog.io/@colorful8315/Hash%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80

https://dbehdrhs.tistory.com/70

https://baeharam.netlify.app/posts/data%20structure/hash-table

https://ratsgo.github.io/data%20structure&algorithm/2017/10/25/hash/

https://mangkyu.tistory.com/102