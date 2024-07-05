# Java Stream API


## JAVA Stream API란?

<aside>
💡 Classes to support functional-style operations on streams of elements, such as map-reduce transformations on collections.

</aside>

```java
     int sum = widgets.stream()
                      .filter(b -> b.getColor() == RED)
                      .mapToInt(b -> b.getWeight())
                      .sum();
```

- Stream API란
    - I/O Stream과는 별도
    - 데이터 소스를 스트림으로 만들어 작업하기 위한 기능 제공 (JAVA 8부터 지원)
    - 동일한 형싱르 지닌 복수의 데이터를 다룬다는 개념
    - 순차적으로 하나씩 사용되고 사라지게 된다. (메모리의 낭비를 줄인다.)

- Stream의 특징
    - 저장소가 없다
        - 스트림은 요소를 저장하는 데이터구조가 아니라 데이터 구조, 배열, 생성기 함수 또는 I/O 채널과 같은 소스로부터 요소를 전달하여 계산 작업의 파이프라인을 통해 전달한다.
    - 함수적 특성
        - 스트림에 대한 작업은 결과를 생성하지만 소스를 수정하지 않는다. filter와 같은 메소드를 이용할 경우 필터링 된 요소를 뺀 새로운 스트림을 생성한다.
    - 지연 연산 추구
        - 지연연산 : 결과값이 필요할 때까지 계산을 늦추는 기법
        - 스트림 작업의 분류
            - 중간 작업 : 항상 지연
            - 최종 작업
    - 무한 스트림
        
        <aside>
        💡  Possibly unbounded. While collections have a finite size, streams need not. Short-circuiting operations such as `limit(n)` or `findFirst()` can allow computations on infinite streams to complete in finite time.
        
        </aside>
        
        - 기본적으로 데이터가 무한히 들어갈  수 있다 (사이즈가 정해져 있지 않다)
    - Consumable
        
        <aside>
        💡  Consumable. The elements of a stream are only visited once during the life of a stream. Like an [`Iterator`](https://docs.oracle.com/javase/8/docs/api/java/util/Iterator.html), a new stream must be generated to revisit the same elements of the source.
        
        </aside>
        
        - 한 번 소비된 스트림은 다시 접근이 불가능합니다.
    
- 지연 연산(Lazy)
    
     `filtering, mapping, or duplicate removal`
    
    - 스트림 파이프라인이 어떠한 중간연산과 최종연산으로 구성되어 있는지에 대한 검사를 진행한다.
    - 최적화를 통해 개별 요소에 대한 스트림 연산 수행
    
    - 루프 퓨전 (loop fusion)
        - 연속적으로 체이닝된 복수의 스트림 연산을 하나의 연산 과정으로 병합
    - 쇼트서킷
        - 불필요한 연산을 의도적으로 수행하지 않음
        - 무한 스트림을 다루는데 적합
        
    
    ```java
    List<String> result = Stream.of("java", "streams", "are", "great", "stuff")
        .filter(s -> {
                      System.out.println("filtering " + s);
                      return s.length()>=4;
                     })
        .map(s -> {
                   System.out.println("mapping " + s);
                   return s.toUpperCase();
                  })
        .limit(3)
        .collect(Collectors.toList());
    System.out.println("Result:");
    result.forEach(System.out::println);
    ```
    
    ```java
    filtering java
    mapping java
    filtering streams
    mapping streams
    filtering are
    filtering great
    mapping great
    Result:
    JAVA
    STREAMS
    GREAT
    ```
    
    *주의해야할 것 : 위와 아래의 다른 점은?*
    
    ```java
    Stream.generate(() -> new RandomInt()) 
    .sorted(Comparator.comparingInt(Data::getValue)) 
    .limit(5) 
    .collect(Collectors.toList());
    ```
    
    ```java
    Stream.generate(() -> new RandomInt())
    .limit(5) 
    .sorted(Comparator.comparingInt(Data::getValue))
    .collect(Collectors.toList());
    ```
    

- stateless operation(상태가 없는 연산) vs stateful operation(상태가 있는 연산)  [ Stream]
    - Stateless operation (상태가 없는 연산)
        - `filter(), map(), flatMap()`
        - 현재 스트림 요소를 처리하기 위해 이전에 스트림을 지나간 요소에 대한 데이터가 필요 없는 것
        - process each element one by one
        
    - Stateful operation (상태가 있는 연산)
        - `distinct(), limit(), sorted(), collect(), reduce()`
            
            ```java
            // reduce
            
            List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
            int sum = numbers.stream()
                             .reduce(0, (subtotal, element) -> subtotal + element);
            System.out.println(sum);  // 출력: 15
            ```
            
        - 이전 스트림 요소의 계산 상태를 기억해야 다음 계산이 가능한 연산.
    
    - 일반적으로 상태가 없는 연산은 병렬처리에 문제가 발생하지 않는다. (parallel)
        - 작은 작업으로 쪼개 병렬 처리가 되므로 이득임!
    
    |  | Stateful Streaming | Stateless Streaming |
    | --- | --- | --- |
    | Order -  | Dependency | Independence |
    | Buffering | O | X |
    | Parallel- | ism Challenges | Friendly |

---

### Stream Api의 메서드들을 조사할 예정입니다

iterator

generator… 등..

각 메서드의 작동 원리 (int, class 등 데이터 종류에 따른 계산 방식 차이도 점검..)

* 빠른가? 빠르다면 왜 빠른가? 느린가?
* 언제 사용해야 하는가?

* 아래는 같이 사용하면 좋은 Lamda 함수 / 함수형 인터페이스(저번에 답변 못한 부분..)을 추가적으로 가볍게 다룰 예정입니다!!!

---

### 함수형 인터페이스

- 람다식 → 순수 함수 선언 가능

- 함수형 인터페이스
    - 함수를 1급 객체처럼 다룰 수 있게 해주는 어노테이션
    - 인터페이스에 선언하여 단 하나의 추상 메소드만을 갖도록 제한하는 역할
    
    ```java
    @FunctionalInterface
    interface MyLambdaFunction {
        int max(int a, int b);
    }
    
    public class Lambda {
    
        public static void main(String[] args) {
    
            // 람다식을 이용한 익명함수
            MyLambdaFunction lambdaFunction = (int a, int b) -> a > b ? a : b;
            System.out.println(lambdaFunction.max(3, 5));
        }
    
    }
    ```
    

---

https://mangkyu.tistory.com/112

https://bugoverdose.github.io/development/stream-lazy-evaluation/

https://bugoverdose.github.io/development/stream-operations/

https://stackoverflow.com/questions/42804226/loop-fusion-of-stream-in-java-8-how-it-works-internally

https://javatute.com/core-java/stateful-vs-stateless-streaming-java/