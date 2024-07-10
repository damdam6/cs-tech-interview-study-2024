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
        
        - 모든 스트림의 요소를 처리하지 않고도 결과 반환 가능
            - limit() : 숫자 제한 이후에 불필요한 연산 하지 않음.
            - findAny() : 한 가지 값을 찾으면 종료됨.
    
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

    *지연연산(중간연산/최종연산)의 이해를 돕기 위한 테스트 코드*

    ```
    public class LazyTest {

    public static void main(String[] args) {

        List<String> list = Arrays.asList("test", "lazy", "loading", "show", "result", "light");

        Stream<String> stream = list.stream().filter(
                text -> {
                    System.out.println("filter: " + text);
                    return text.startsWith("l");
                }
        );

        System.out.println("stream work?");

        stream = stream.sorted(
                (text, text2) -> {
                    System.out.println("sorted: " + text + " " + text2);
                    return text.compareTo(text2);
                }
        );

        System.out.println("stream work? 2");

        List<String> newList = stream.collect(Collectors.toList());
        }
    }

    ```



    결과값
    ```
    > Task :LazyTest.main()
    stream work?
    stream work? 2
    filter: test
    filter: lazy
    filter: loading
    filter: show
    filter: result
    filter: light
    sorted: loading lazy
    sorted: light loading
    sorted: light loading
    sorted: light lazy

    ```


    
    32
    
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
    위의 코드는 sorted가 우선하기 때문에 작업이 멈추지 않음. 아래는 limit(5)를 통해 갯수를 제한하여 작업이 종료됨.


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

### Stream vs ParallelStream

* Stream연산은 기본적으로 순차처리를 진행합니다. 그러나 병렬처리를 원할 경우 ParellelStream을 사용할 수 있습니다.

```
        List<Integer> listOfNumbers = Arrays.asList(1, 2, 3, 4);
        System.out.println("Stream");
        listOfNumbers.stream().forEach(number ->
                System.out.println(number + " " + Thread.currentThread().getName())
        );
        System.out.println("ParallelStream");
        listOfNumbers.parallelStream().forEach(
                number ->
                        System.out.println(number + " " + Thread.currentThread().getName())
        );

```

 아래 결과와 같이 여러개의 스레드를 사용하는 것을 확인할 수 있습니다.
```
 > Task :ParellelStream.main()
Stream
1 main
2 main
3 main
4 main
ParallelStream
3 main
4 main
2 ForkJoinPool.commonPool-worker-1
1 ForkJoinPool.commonPool-worker-2

```

---

### Primitive type vs Reference type

- 자바 Stream은 참조타입 만을 다룬다.
    - Stream<T>
    - **기본 타입 스트림**: `IntStream`, `LongStream`, `DoubleStream`
    - 박싱/언박싱 발생으로 인한 오버헤드
        
        ```java
        List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
        int sum = integers.stream() // Stream<Integer>
                          .mapToInt(Integer::intValue) // 언박싱 발생
                          .sum();
        System.out.println("Sum: " + sum);
        
        ```
        
    - IntStream으로 작업하면
        
        ```java
        int[] intArray = {1, 2, 3, 4, 5};
        // IntStream intStream = Arrays.stream(intArray);
        int sum = Arrays.stream(intArray) // IntStream
                        .sum();
        System.out.println("Sum: " + sum);
        
        ```
        

---

### 사실 스트림 타입은 느립니다

for-loop

Stream

Parallel stream

비교 테스트

10,000,000

![Untitled](Java Stream API/Untitled.png)

```java
package testForStream;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.stream.Collectors;

public class Big_Size_TEST {

    public static void main(String[] args) {
        // 데이터 준비
        List<value> values = generateRandomValues(1_000_000_000); // 데이터 크기 증가

        // for 루프를 사용한 최대 값 찾기
        long startTimeForLoop = System.nanoTime();
        int maxForLoop = findMaxUsingForLoop(values);
        long endTimeForLoop = System.nanoTime();
        long durationForLoop = (endTimeForLoop - startTimeForLoop) / 1_000_000; // ms

        // 스트림을 사용한 최대 값 찾기
        long startTimeStream = System.nanoTime();
        int maxStream = findMaxUsingStream(values);
        long endTimeStream = System.nanoTime();
        long durationStream = (endTimeStream - startTimeStream) / 1_000_000; // ms

        // 병렬 스트림을 사용한 최대 값 찾기
        long startTimeParallelStream = System.nanoTime();
        int maxParallelStream = findMaxUsingParallelStream(values);
        long endTimeParallelStream = System.nanoTime();
        long durationParallelStream = (endTimeParallelStream - startTimeParallelStream) / 1_000_000; // ms

        // 결과 출력
        System.out.println("For loop max value: " + maxForLoop + ", duration: " + durationForLoop + " ms");
        System.out.println("Stream max value: " + maxStream + ", duration: " + durationStream + " ms");
        System.out.println("Parallel stream max value: " + maxParallelStream + ", duration: " + durationParallelStream + " ms");
    }

    private static List<value> generateRandomValues(int count) {
        Random random = new Random();
        List<value> values = new ArrayList<>(count);
        for (int i = 0; i < count; i++) {
            values.add(new value(random.nextInt()));
        }
        return values;
    }

    private static int findMaxUsingForLoop(List<value> values) {
        int max = Integer.MIN_VALUE;
        for (value v : values) {
            if (v.getV() > max) {
                max = v.getV();
            }
        }
        return max;
    }

    private static int findMaxUsingStream(List<value> values) {
        return values.stream()
                .map(value::getV)
                .reduce(Integer.MIN_VALUE, Math::max);
    }

    private static int findMaxUsingParallelStream(List<value> values) {
        return values.parallelStream()
                .map(value::getV)
                .reduce(Integer.MIN_VALUE, Math::max);
    }

    static class value {
        int v;

        value(int v) {
            this.v = v;
        }

        public int getV() {
            return v;
        }
    }

}

```

- 복잡도와 숫자가 증가했을 때는 이점이 보입니다.

![Untitled](Java Stream API/Untitled%201.png)

```java
package testForStream;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class Compli_Loop {

    public static void main(String[] args) {
        // 데이터 준비
        List<value> values = generateRandomValues(200_000_000); // 데이터 크기 증가

        // for 루프를 사용한 최대 값 찾기
        long startTimeForLoop = System.nanoTime();
        int maxForLoop = findMaxUsingForLoop(values);
        long endTimeForLoop = System.nanoTime();
        long durationForLoop = (endTimeForLoop - startTimeForLoop) / 1_000_000; // ms

        // 스트림을 사용한 최대 값 찾기
        long startTimeStream = System.nanoTime();
        int maxStream = findMaxUsingStream(values);
        long endTimeStream = System.nanoTime();
        long durationStream = (endTimeStream - startTimeStream) / 1_000_000; // ms

        // 병렬 스트림을 사용한 최대 값 찾기
        long startTimeParallelStream = System.nanoTime();
        int maxParallelStream = findMaxUsingParallelStream(values);
        long endTimeParallelStream = System.nanoTime();
        long durationParallelStream = (endTimeParallelStream - startTimeParallelStream) / 1_000_000; // ms

        // 결과 출력
        System.out.println("For loop max value: " + maxForLoop + ", duration: " + durationForLoop + " ms");
        System.out.println("Stream max value: " + maxStream + ", duration: " + durationStream + " ms");
        System.out.println("Parallel stream max value: " + maxParallelStream + ", duration: " + durationParallelStream + " ms");
    }

    private static List<value> generateRandomValues(int count) {
        Random random = new Random();
        List<value> values = new ArrayList<>(count);
        for (int i = 0; i < count; i++) {
            values.add(new value(random.nextInt()));
        }
        return values;
    }

    private static int findMaxUsingForLoop(List<value> values) {
        int max = Integer.MIN_VALUE;
        for (value v : values) {
            int processedValue = complexProcessing(v.getV());
            if (processedValue > max) {
                max = processedValue;
            }
        }
        return max;
    }

    private static int findMaxUsingStream(List<value> values) {
        return values.stream()
                .map(value::getV)
                .map(Compli_Loop::complexProcessing)
                .reduce(Integer.MIN_VALUE, Math::max);
    }

    private static int findMaxUsingParallelStream(List<value> values) {
        return values.parallelStream()
                .map(value::getV)
                .map(Compli_Loop::complexProcessing)
                .reduce(Integer.MIN_VALUE, Math::max);
    }

    // 복잡한 연산을 시뮬레이션하는 메소드
    private static int complexProcessing(int value) {
        // 예시: CPU 집약적 연산 시뮬레이션 (단순히 반복 연산)
        int result = value;
        for (int i = 0; i < 1000; i++) { // 복잡도 증가
            result = result ^ (result << 1);
        }
        return result;
    }

    static class value {
        int v;

        value(int v) {
            this.v = v;
        }

        public int getV() {
            return v;
        }
    }
}

```

---

## 어떤 개발자의 테스트…

![Untitled](Java Stream API/Untitled%202.png)

---

**참고 자료**


### 메서드 참조 종류

메서드 참조는 네 가지 주요 유형이 있습니다:

1. **정적 메서드 참조**: `ClassName::methodName`
2. **인스턴스 메서드 참조 (특정 객체)**: `instance::methodName`
3. **인스턴스 메서드 참조 (임의의 객체)**: `ClassName::methodName`
4. **생성자 참조**: `ClassName::new`

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

https://teachingdev.com/for-vs-streams/

https://x.com/xpvit/status/1626899895499063297?s=20

https://devm.io/java/java-performance-tutorial-how-fast-are-the-java-8-streams-118830