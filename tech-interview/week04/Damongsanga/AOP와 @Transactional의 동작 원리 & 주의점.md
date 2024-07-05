# AOP와 @Transactional의 동작 원리 & 주의점

태그: AOP, Spring, Transaction
날짜: 2024년 7월 1일

## AOP(Aspect Oriented Programming)

관점 지향 프로그래밍

**Aspect(관점)이란 흩어진 관심사들을 하나로 모듈화 한 것**

흩어진 관심사 : 클래스 설계하다보면 로깅, 보안, 트랜잭션 등 여러 클래스에서 공통적으로 사용하는 부가 기능존재

문제 상황

- 여러 곳에서 반복적인 코드를 작성해야 한다.
- 코드가 변경될 경우 여러 곳에 가서 수정이 필요하다.
- 주요 비즈니스 로직과 부가 기능이 한 곳에 섞여 가독성이 떨어진다.

### 주요개념

**어떠한 “조건”으로 정의된 “pointcut”과 일치하는 곳(join point)을 찾아내어 특정 동작(advice)을 실행한다!**

- **Aspect**: Advice + PointCut로 AOP의 기본 모듈
  - “반복되는 기능을 하나의 모듈로 만들었다”라고 할 때 그 모듈이 Aspect
  - 미리 지정해둔 시점(pointcut과 join point가 일치하는 곳) 마다 실행
- **Advice [어떤]** : aspect가 취하는 동작
  - Target에 제공할 부가 기능을 담고 있는 모듈
  - "around", "before", "after" 등 다양한 종류의 Advice
- **Joint Point [여기서]** : Advice가 적용될 (수 있는) 위치
  - 메서드 진입 지점, 생성자 호출 시점, 필드에서 값을 꺼내올 때 등 다양한 시점에 적용 가능
  - 어플리케이션의 모든 동작(메서드 실행)은 Join Point이다
  - **Spring AOP에서 join point는 항상 메소드 실행을 나타냄**
- **PointCut [언제]**: Target을 지정하는 정규 표현식
  - join point들 중에서 **실행할 대상을 선택하는 기준**
- **Target**: Advice이 부가 기능을 제공할 대상 (Advice가 적용될 비즈니스 로직)
  - Spring AOP에서는 항상 프록시된 객체

![image](<./AOP와%20@Transactional의%20동작%20원리%20&%20주의점/aW1hZ2U(31).png>)

### Advice 동작 방법

- 각 advice는 인터셉터로 변환되어 join point 주변에 위치한다.
- join point가 실행될 때, interceptor chain은 해당 join point와 일치하는 pointcut에 대한 advice를 순차적으로 실행한다.
- advice의 실행 순서는 각 advice가 선언된 순서와 동일하다.
  - 즉, 같은 pointcut을 가진 advice의 경우에는 선언된 순서에 따라 실행되는 것이다.

## **Spring AOP**

- 프록시 패턴 기반의 AOP 구현체이다.
  - 프록시 객체를 쓰는 이유는 기존 코드 변경 없이 접근 제어 및 부가 기능을 추가하기 위함.
- 스프링 빈에만 AOP를 적용할 수 있음.
- Spring AOP는 프록시 기반의 AOP로서 스프링 프레임워크의 기능을 이용하여 비즈니스 로직과 관심사를 분리하여 처리할 수 있다.
- 지금 개발 중인 프로젝트에는 인증 기능이 필요하다. 모든 요청에 대해 사용자의 회원 가입 및 로그인 여부에 대한 인증을 추가해 주어야 한다. 인증 기능이 필요한 모든 클래스마다 인증 코드를 중복해서 구현하면 어떨까? 굉장히 비효율적 일 수 밖에 없다.
- 이 때, 인증 기능을 AOP로 분리해서 관리하면, 모든 클래스에서 인증 기능을 중복해서 구현하지 않아도 되므로 코드 양이 줄어들고 유지보수가 용이해질 것이다. 그리고 이렇게 분리된 공통 모듈들을 IoC/DI를 이용해 스프링 컨테이너에서 관리하게 위임할 수 있다.
- 즉, 스프링 컨테이너는 객체 간의 의존성을 관리하면서 동시에 공통 기능을 모듈화하여 관리할 수 있다.

![image](./AOP와%20@Transactional의%20동작%20원리%20&%20주의점/Untitled.png)

![image](./AOP와%20@Transactional의%20동작%20원리%20&%20주의점/Untitled%201.png)

## 프록시 사용

![image](./AOP와%20@Transactional의%20동작%20원리%20&%20주의점/Untitled%202.png)

- 프록시 사용하지 않으면 Aspect 클래스에 정의된 기능 사용하기 위해 클래스 직접 호출해야 → 이는 동일한 문제를 야기
- **Spring에서는 런타임 시에** **Target 클래스 혹은 그의 상위 인터페이스를 상속하는 프록시 클래스를 생성하고, 프록시 클래스에서 부가 기능에 관련된 처리**
  ### 프록시란
  프록시 객체는 실제 객체의 대리인 역할을 수행하며, 실제 객체와 동일한 인터페이스를 구현한다. 따라서, 클라이언트에서 실제 객체를 호출하는 것처럼 보일지라도 실제로는 진짜 객체의 메소드를 프록시 객체가 대신 호출한다.
  실제 객체와 같은 인터페이스를 구현하였더라도 실제 객체의 메소드를 호출할 때 추가적인 동작을 수행하도록 구현되는데, 이를 통해 타깃이 되는 실제 객체의 동작을 제어하거나 변경할 수 있다.
  장점
  - OCP 원칙
  - SRP 원칙
  ### 런타임 위빙(Runtime Weaving)
  : 타겟 객체를 새로운 프록시 객체로 적용하는 과정
  프록시 패턴은 타겟 하나 하나마다 프록시 객체를 정의해야하므로 번거롭고 코드의 중복이 생긴다는 점이 있다.
  이를 방지하기 위해 클라이언트가 메서드를 요청하면 ProxyFactoryBean에서 인터페이스 유무를 확인하여 인터페이스가 있으면 동적으로 JDK Dynamic Proxy를 생성하여 호출하고 없으면 CGLIB 방식으로 프록시를 생성하는 방법

## JDK vs CGLib Proxy (매우 중요)

### 1. JDK : Interface Implement Based

**JDK Proxy는 Target의 상위 인터페이스를 상속 받아 프록시를 만든다.**

![image](./AOP와%20@Transactional의%20동작%20원리%20&%20주의점/Untitled%203.png)

![image](./AOP와%20@Transactional의%20동작%20원리%20&%20주의점/Untitled%204.png)

- 자바 프록시 API를 사용하여 **“인터페이스”**를 기반으로 프록시 객체를 동적으로 생성
- **따라서 인터페이스를 구현한 클래스가 아니면 프록시를 생성할 수 없다.**
- 리플렉션 API를 통해서 구현하고 있어 느리다.
  **구체적인 클래스 타입을 알지 못해도 런타임에 클래스의 정보에 접근할 수 있어야 함으로 자바 API 리플렉션을 사용해야 한다. 그러다보니 동적일 때 해결되는 타입을 포함하게 으므로 JVM의 optimization이 작동하지 않아 성능상 느리다.**
- Target에서 다른 구체 클래스에 의존하고 있다면, JDK 방식에서는 그 클래스(빈)를 찾을 수 없어 런타임 에러가 발생한다.

![image](./AOP와%20@Transactional의%20동작%20원리%20&%20주의점/Untitled%205.png)

- InvocationHandler를 구현
  - invoke() 메소드 하나만 가지는 인터페이스 ⇒ 이 메서드가 실행됨
  - 이 메소드는 Proxy.newProxyInstance에 의해 동적으로 생성된 프록시 객체의 메소드가 호출되면 동작하는 메소드이다
  - 각각, 또는 전체의 메소드의 확장 기능을 구현할 수 있고, 호출된 메소드 정보와 파라미터를 파라미터로 받는다.
  - **즉, 프록시 대상이 되는 인터페이스 각각의 메소드의 사용될 확장기능 코드가 반복되지 않게 구현할 수 있다.**
  - 예시 코드
    ```java
    @Slf4j
    public class TransactionInvocationHandler implements InvocationHandler {

        private final Object target;

        public TransactionInvocationHandler(Object target) {
            this.target = target;
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            log.info("InvocationHandler parameter: proxy={}, method={}, args={}", proxy, method, args);

            try {
                log.info("--- 트랜잭션 커밋 시작 ---");

                Object result = method.invoke(target, args);

                log.info("--- 트랜잭션 커밋 완료 ---");
                return result;
            } catch (Exception e) {
                log.info("--- 트랜잭션 롤백 ---");
                throw e;
            } finally {
                log.info("--- DB 커넥션 자원 반환 ---");
            }
        }
    }
    ```

![image](./AOP와%20@Transactional의%20동작%20원리%20&%20주의점/Untitled%206.png)

- 장점
  - 개발자가 직접 프록시 객체를 만들 필요가 없다.
- 단점
  - 프록시하려는 클래스는 반드시 인터페이스의 구현체여야한다.
  - 리플렉션을 활용하므로 성능이 떨어진다.
- 주의할 점
  - [만약 인터페이스 타입으로 프록시를 구현했는데 구현체 클래스 타입으로 DI 해버리면 Runtime Exception이 발생할 수 있다!!](https://stackoverflow.com/questions/37726351/can-not-set-field-to-com-sun-proxy-proxy/37726709)

### 2. CGLib (Code Generator Library) : Subclass Extend Based

**CGLib Proxy는 Target 클래스를 상속 받아 프록시를 만든다.**

![image](./AOP와%20@Transactional의%20동작%20원리%20&%20주의점/Untitled%207.png)

- 자바 바이트 코드 조작 라이브러리를 사용하여 프록시 객체를 **“상속으로”** 생성
- **따라서 final 클래스, final 메서드에 대해서는 프록시를 생성할 수 없다.**
- JDK 방식과는 달리 인터페이스를 구현하지 않아도 되고, 구체 클래스에 의존하기 때문에 런타임 에러가 발생할 확률도 상대적으로 적다.
- 또한 JDK Proxy는 내부적으로 Reflection을 사용해서 추가적인 비용이 들지만, CGLib는 그렇지 않다고 한다.
- 외부 라이브러리인 Enhancer를 사용해서 CGLIB 생성한다. 때문에 보다 더 복잡하다.

- **MethodInterceptor를 구현**
  - 예시 코드
    ```java
    @Slf4j
    public class TransactionMethodInterceptor implements MethodInterceptor {

        private final Object target;

        public TransactionMethodInterceptor(Object target) {
            this.target = target;
        }

        @Override
        public Object intercept(Object object, Method method,
                                Object[] args, MethodProxy proxy) throws Throwable {
            try {
                log.info("--- 트랜잭션 커밋 시작 ---");

                Object result = proxy.invoke(target, args);

                log.info("--- 트랜잭션 커밋 완료 ---");
                return result;
            } catch (Exception e) {
                log.info("--- 트랜잭션 롤백 ---");
                throw e;
            } finally {
                log.info("--- DB 커넥션 자원 반환 ---");
            }
        }
    }
    ```
  ![image](./AOP와%20@Transactional의%20동작%20원리%20&%20주의점/Untitled%208.png)
- 장점
  - 인터페이스 없이 단순 클래스만으로도 프록시 객체를 동적으로 생성해 줄 수 있다.
  - 리플렉션이 아닌 바이트 조작을 사용하며, 타겟에 대한 정보를 알고 있기 때문에 JDK Dynamic Proxy에 비해 성능이 좋다.
- 단점
  - **의존성을 추가해야 한다. (Spring 3.2 이후 버전의 경우 Spring Core 패키지에 포함되어 있음)**
    ![image](./AOP와%20@Transactional의%20동작%20원리%20&%20주의점/Untitled%209.png)
  - default 생성자가 필요하다. (현재는 objenesis 라이브러리를 통해 해결)
  - 타겟의 생성자가 두 번 호출된다. (현재는 objenesis 라이브러리를 통해 해결)
  - 상속을 사용함으로 final 메소드가 들어있거나, final 클래스면 안 된다. 더불어, private 접근자로 된 메소드도 상속이 불가하므로 적용되지 않는다.

### **현재 Spring Boot는 CGLib를 사용한 방식을 기본으로 채택?**

- 예전에는 인터페이스 기준으로 프록시 생성 했음
  - 이때 만약 타깃이 하나 이상의 인터페이스를 구현하고 있는 클래스라면 JDK Dynamic Proxy의 방식으로 생성되고 인터페이스를 구현하지 않은 클래스라면 CGLIB의 방식으로 AOP 프록시를 생성해줍니다. ⇒ 라고 하네요.
- [그런데.. Spring Boot 1.4 부터는 CGLib을 방식으로 Proxy를 생성해주고 있다고 한다.](https://github.com/spring-projects/spring-boot/issues/8434)
  - 성능상의 이유와 CGLib 가 많이 개선되어 Exception 처리 및 클래스 추적에 유리하기 때문이라고 함
  - **CGLib를 기본으로 선택하기 위해서는 `proxy-target-class`** 속성값을 true로 설정하거나 configuration 파일에 [`@EnableTransactionManagement(proxyTargetClass = true)`](https://github.com/spring-projects/spring-boot/issues/5423) 어노테이션을 추가해야 한다
  - 따라서 `proxy-target-clas`는 기본적으로 true로 설정되어있고, JDK로 하려면 false로 해야 한다
  - TransactionAutoConfiguration
    ```java
    @AutoConfiguration
    @ConditionalOnClass(PlatformTransactionManager.class)
    public class TransactionAutoConfiguration {

    	// ...

    	@Configuration(proxyBeanMethods = false)
    	@ConditionalOnBean(TransactionManager.class)
    	@ConditionalOnMissingBean(AbstractTransactionManagementConfiguration.class)
    	public static class EnableTransactionManagementConfiguration {

    		@Configuration(proxyBeanMethods = false)
    		@EnableTransactionManagement(proxyTargetClass = false)
    		@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "false")
    		public static class JdkDynamicAutoProxyConfiguration {

    		}

    		@Configuration(proxyBeanMethods = false)
    		@EnableTransactionManagement(proxyTargetClass = true)
    		@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true",
    				matchIfMissing = true)
    		public static class CglibAutoProxyConfiguration {

    		}

    	}

    	// ...

    }
    ```

### 참조

[https://minkukjo.github.io/framework/2021/05/23/Spring/](https://minkukjo.github.io/framework/2021/05/23/Spring/)
[https://mangkyu.tistory.com/175](https://mangkyu.tistory.com/175)
[https://gmoon92.github.io/spring/aop/2019/04/20/jdk-dynamic-proxy-and-cglib.html](https://gmoon92.github.io/spring/aop/2019/04/20/jdk-dynamic-proxy-and-cglib.html)
[https://suyeonchoi.tistory.com/81](https://suyeonchoi.tistory.com/81)
[https://steady-coding.tistory.com/608](https://steady-coding.tistory.com/608)

---

여기까지가 AOP 관련된 내용이고, 아래는 @Transactional, Reflection에 관련된 내용입니다.
사실 @Transactional 동작원리를 이해하고자 AOP를 고른거라 추가로 공부한 내용이라 보실 필요는 없는데, [@Transactional의 주의점] 파트에서 1, 3번은 AOP 관련 내용이라 참고하면 좋을 것 같습니다!
Reflection은 JDK Dynamic Proxy 관련해서 나와서 간단하게만 학습해봤습니다.

---

# @Transactional

**Spring은 JDK Proxy, Spring Boot는 CGLIb Proxy를 기본으로 한다고 함**

동작 과정

1. target에 대한 호출이 들어오면 AOP proxy가 이를 가로채서(intercept) 가져온다.
2. AOP proxy에서 Transaction Advisor가 commit 또는 rollback 등의 트랜잭션 처리를 한다.
3. 트랜잭션 처리 외에 다른 부가 기능이 있을 경우 해당 Custom Advisor에서 그 처리를 한다.
4. 각 Advisor에서 부가 기능 처리를 마치면 Target Method를 수행한다.
5. interceptor chain을 따라 caller에게 결과를 다시 전달한다.

## @Transactional 주의점

Self Invocation을 어떻게 해결해야 하는지를 비롯하여 주의해야 할 점들을 잘 이해하자!

### 1. private은 트랜잭션 처리를 할 수 없다. (상속 불가)

앞서 트랜잭션이 코드 레벨에서 어떻게 동작하는지 대락적으로 살펴봤다. 프록시 객체는 타겟 객체/인터페이스를 상속 받아서 구현하는데, private으로 되어 있으면 자식인 프록시 객체에서 호출할 수 없다. 따라서 `@Transactional` 이 붙는 메서드, 클래스는 프록시 객체에서 접근 가능한 레벨로 지정해야 한다.

### **2. 우선순위**

@Transactional은 우선순위를 가지고 있다.

**클래스 메소드 > 클래스 > 인터페이스 메소드 > 인터페이스**

### **3. Self Invocation : 같은 클래스내에서 트랜잭션이 걸린 메소드를 호출하면 트랜잭션이 작동하지 않는다. == 트랜잭션은 객체 외부에서 처음 진입하는 메서드를 기준으로 동작한다.**

**내부 메서드 호출로는 “별도” 트랜잭션이 안생긴다.** 객체 외부에서 첫 진입한 메서드가 @Transactional이 있으면 호출된 내부 메서드도 기존 트랜잭션에 “함께” 반영되고 없다면 트랜잭션에는 전혀 참여하지 않게 된다. (당연하다!)

예시

**Class A 에서 @Transactional이 걸린 B 메서드와 C 메서드가 있다. 그 때 외부에서 A.B()를 호출하고, B 메서드 내부에서 C를 호출한다면, B에 대한 트랜잭션은 동작하지만, C에 대한 트랜잭션은 동작 안한다. 예시를 들어서 C에서 에러가 발생하면 C에 대한 트랜잭션은 존재하지 않음으로 롤백 안되고 B에 대한 트랜잭션만 롤백된다.**

사실상 [**트랜잭션은 객체 외부에서 처음 진입하는 메서드를 기준으로 동작한다.] 라는 말과 비슷한 상황인데\*\* 객체에 처음 접근하는 기준으로 AOP가 동작함으로, 첫 진입 메서드에 대해서만 `@Transactional` 이 제 역할을 할 수 있다. (뜻은 다르지만 결국 의도하는 바는 같다. 외부에서 진입하는 메서드에 트랜잭션이 걸려있어야만 제대로 동작한다는 뜻이다. 내부 호출 등은 동작 제대로 안된다)

[최범균님의 프로그래밍 초식 : 초심자가 저지르기 쉬운 DB 코딩 실수 3가지 (트랜잭션, 프라이머리/레플리카)](https://www.notion.so/DB-3-57ee2a99c8714afdaf8d40612e3e560f?pvs=21) 정리 내용에도 나와있다.

### 4. 그렇다면 **Self Invocation의** 해결방법은 무엇이 있을까?!

클래스 분리 등이 있을 수 있겠지만 추가적으로 다른 방법도 알아보자!

### 4.1 AspectJ Mode (Weaving)

- Non-Public 메서드에 적용하고 싶으면 AspectJ Mode를 고려해야한다
- **Weaving**
  - **Advice(공통코드) 를 핵심 코드 로직에 삽입해버리는 것**
- 구현 방식
  1. **Load-Time Weaver**

     객체를 application context에 Load 할때, AspectJ에 의해서 wearving된 객체를 넘겨주는 방식

     1. application context에 로드된 객체의 loading
     2. aspectj weaver에 의한 객체 weaving (@Transaction annotation이 있는 class, method에 대한 transaction 처리가 된 객체로 변경)
     3. 객체의 이용

     performance의 하락

  2. **Compile-Time Weaver**

     compile 시에 aspectj에서 간섭해서 필요한 객체에 weaving을 시켜서 class를 만들어내는 방식

     이렇게 되면, application context의 load시에 다른 절차가 없기 때문에, performance의 하락도 없음

     주의사항으로 lombok과 같은 compile시에 간섭하는 여러 plugin들과의 매우 다채로운 충돌이 발생. 특히 lombok과는 사실상 같이 사용하지 못한다

  3. **참고로 Spring AOP는 Run-Time Weaving이다!!**
- 사용 방법 (**Compile-Time Weaver)**
  1. **4가지 조건 설정**
     1. **AspectJ 형식의 Aspect**
     2. **AspectJ Runtime**
     3. **AspectJ Weaver**
     4. **AspectJ Compiler(AJC)**
     5. 더 자세한 내용은 [여기](https://gmoon92.github.io/spring/aop/2019/05/24/aspectj-of-spring.html)에서
  2. @Transactional은 Proxy Mode와 AspectJ Mode가 있는데, **Proxy Mode가 Default**로 설정되어있다. 이를 바꿔주고, CGLib 모드로 프록시 객체 생성하도록 proxyTargetClass = true도 설정
  ```java
  @Configuration
  @EnableTransactionManagement(proxyTargetClass = true, mode = AdviceMode.ASPECTJ)
  @EnableAspectJAutoProxy
  @ComponentScan({ "com.example"})
  public class AppConfig {
  }
  ```

### 4.2 TransactionTemplate

**프로그래밍적 트랜잭션이라고도 한다 (@Transactional은 선언적 트랜잭션)**

아래 2가지 해결

1. 메서드 레벨에 AOP 가 적용되기 때문에 트랜잭션 단위도 메서드 레벨로 적용되어, 메서드 내에서 지정 불가능

2. self invocation 에서 트랜잭션 적용 불가

추가적으로 트랜잭션의 범위를 최소화할 수 있다.

```java
public class PointTransaction {

	private PointDao pointDao;
    private TransactionTemplate transactionTemplate;

    public void setPointDao(PointDao pointDao) {
    	this.pointDao = pointDao;
    }

    public void setTransactionManager(PlatformTransactionManager transactionManager) {
    	this.transactionTemplate = new TransactionTemplate(transactionManager);
        this.transactionTemplate.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
    }

    public void invoke1() {
    	dolnternalTransaction1();
    }
    public void invoke2() {
    	dolnternalTransaction2();
    }

    // 반환값이 없으면
    public void dolnternalTransaction1() throws Exception {
    	transactionTemplate.execute(new TransactionCallbackWithoutResult() {
        	protected void doInTransactionWithoutResult(TransactionStatus status) {
            	try {
                	pointDao.plusPoint()
                } catch (Exeption ex) {
                	status.setRollbackOnly();
                }
            }
        });
    }

    // 반환값이 있으면
    public void dolnternalTransaction2() throws Exception {
    	transactionTemplate.execute(new TransactionCallback() {
        	protected Object doInTransaction(TransactionStatus status) {
            	try {
                	var result = pointDao.plusPoint();
                } catch (Exeption ex) {
                	status.setRollbackOnly();
                }
                return result;
            }
        });
    }
}
```

### 4.3 IOC 컨테이너의 Bean 활용

Ioc 컨테이너에 등록된 자기 자신의 빈을 사용하는 방법으로 this가 아닌 self라는 참조변수로 자기 자신을 주입하면 된다!

이 경우 bean을 통해 메서드가 호출되기 때문에, 프록시를 타고 호출되는 효과가 있어 AOP가 적용된다.

```java
@Service
public class AService{

	@Autowired
	private AService self;

	//...
}
```

그러나, 이 방법은 아래와 같은 문제점이 있을 수 있다.

1. 순환참조를 발생시킬 수 있음
2. 단위테스트가 어려워짐
3. 추가적인 프록시 객체를 만듦으로서 메모리 사용량 증가함

참조

[https://wan-blog.tistory.com/18](https://wan-blog.tistory.com/18)
[https://hungseong.tistory.com/81](https://hungseong.tistory.com/81)
[https://blog.naver.com/PostView.nhn?blogId=tmondev&logNo=220564638014](https://blog.naver.com/PostView.nhn?blogId=tmondev&logNo=220564638014)
[https://itjava.tistory.com/33](https://itjava.tistory.com/33)
[https://devboi.tistory.com/164](https://devboi.tistory.com/164)

## @Transactional 까보기

```sql
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Reflective
public @interface Transactional {
```

1. `@Target({ElementType.TYPE, ElementType.METHOD})`
   - 어노테이션이 적용될 수 있는 대상을 지정
   - `ElementType.TYPE`은 클래스, 인터페이스, 열거형 등의 타입에 적용될 수 있음을 의미
   - `ElementType.METHOD`는 메서드에 적용될 수 있음을 의미
2. `@Retention(RetentionPolicy.RUNTIME)`
   - 유지 정책을 지정 : `RetentionPolicy.RUNTIME`은 이 어노테이션이 **런타임 시에도 유지되어 리플렉션 등으로 접근할 수 있음을 의미**
3. `@Inherited`
   - 이 어노테이션은 `Transactional` 어노테이션이 상속될 수 있음을 의미
   - **즉, 자식 클래스에서도 `Transactional` 어노테이션을 사용할 수 있음**
4. `@Documented`
   - 이 어노테이션은 `Transactional` 어노테이션이 Javadoc에 포함되도록 합니다. 즉, 문서 생성 시 이 어노테이션 정보가 포함됩니다.
5. **`@Reflective`**
   - **리플렉션 API를 통한 접근을 허용한다 → JDK Dynamic Proxy 때문일까??**

### 참조

[https://velog.io/@ann0905/AOP와-Transactional의-동작-원리](https://velog.io/@ann0905/AOP%EC%99%80-Transactional%EC%9D%98-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC)
[https://minkukjo.github.io/framework/2021/05/23/Spring/](https://minkukjo.github.io/framework/2021/05/23/Spring/)
[https://blog.leaphop.co.kr/blogs/53/Spring*Core_Technolohies_2**AOP**1\_\_AOP의*개념\_이해](https://blog.leaphop.co.kr/blogs/53/Spring_Core_Technolohies_2__AOP__1__AOP%EC%9D%98_%EA%B0%9C%EB%85%90_%EC%9D%B4%ED%95%B4)

# 리플렉션

**런타임** 단계에서 *클래스*, *인터페이스*, *필드*, *메서드*를 검사하거나 수정할 수 있다. 또한 *객체를 인스턴스화*하고 메소드를 호출하고 *필드 값을 가져오거나 설정한다*

클래스 타입을 넘겨주어야 리플렉션을 사용할 수 있음

### 언제 사용하면 좋은가?

**컴파일 할 때 그들(클래스, 필드, 메서드...)의 이름을 모를 때 유용**

**Java는 구체적인 클래스를 지정해야만 클래스 정보에 접근할 수 있다**

본인이 라이브러리를 만들어본다고 생각해 봅시다. 라이브러리를 사용하는 사용자가 어떤 클래스를 만들지 예상할 수 있을까요? 또는 무엇인가 동적으로 만들어주는 프로그램을 만들고자 하면 어려움을 겪게 될 것입니다.

**Jackson**, **hibernate**는 리플렉션을 사용하고 있습니다. 객체를 json 타입으로 변경해 주는 jackson 라이브러리의 ObjectMapper 등도 예시입니다.

### 어떻게 가능한가?

자바에서 JVM이 실행되면 사용자가 작성한 “자바 코드”가 **컴파일러를 거쳐 바이트 코드로 변환되어 static 영역에 저장되는데, Reflection API는 해당 영역에서 값을 가져옵니다.**

```java
public class Human extends Animal {

    private String name;
    private int age;

    public Human(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void nextYear(){
        this.age++;
    }

    private int getAge(){
        return this.age;
    }
}

```

```java
import java.lang.reflect.Method;

public static void main(String[] args) throws Exception {
    Animal animal = new Human("휴먼", 2);

    Class human = Class.forName("Human"); //이름으로 클래스 정보 가져오기
    Method nextYear = human.getMethod("nextYear"); // 클래스 정보에서 메서드 추출
    nextYear.invoke(animal, null);  //메서드 호출(객체, 인자) 즉. age 1증가

    Method getAge = duck.getDeclaredMethod("getAge");
    int age = (int) getAge.invoke(animal, null);

    System.out.println("Age : " + age); // Age : 3
}
```

`getMethods`: 이 Class 객체가 나타내는 클래스 또는 인터페이스의 모든 public 메서드를 반영하는 Method 객체를 포함하는 배열을 반환. 부모 클래스, 인터페이스 메서드도 포함

`getDeclaredMethods`: 이 Class 객체가 나타내는 클래스 또는 인터페이스의 모든 선언된 메서드를 반영하는 Method 객체를 포함하는 배열을 반환. 여기에는 public, protected, 기본(패키지) 액세스 및 private 메서드가 포함되지만 상속된 메서드는 제외

그러나 getDeclaredMethod로 가져왔다고 private method를 실행시킬 수는 없다.

```java
Exception in thread "main" java.lang.IllegalAccessException: class Test cannot access a member of class Human with modifiers "private"
	at java.base/jdk.internal.reflect.Reflection.newIllegalAccessException(Reflection.java:392)
```

이를 위해서는 setAccessible(ture)로 접근 제어자를 우회하면 된다.

```java
public static void main(String[] args) throws Exception {
    Animal animal = new Human("휴먼", 2);

    Class human = Class.forName("Human");
    Method nextYear = human.getMethod("nextYear");
    nextYear.invoke(animal, null);

    Method getAge = human.getDeclaredMethod("getAge");
    getAge.setAccessible(true); // 접근 제어자 우회
    int age = (int) getAge.invoke(animal, null);

    System.out.println("Age : " + age); // Age : 3
}
```

### 참조

[https://ebabby.tistory.com/4](https://ebabby.tistory.com/4)
[https://cocoyong.tistory.com/entry/Spring-Reflection-간단히-알아보기](https://cocoyong.tistory.com/entry/Spring-Reflection-%EA%B0%84%EB%8B%A8%ED%9E%88-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)
[https://stackoverflow.com/questions/43585019/java-reflection-difference-between-getmethods-and-getdeclaredmethods](https://stackoverflow.com/questions/43585019/java-reflection-difference-between-getmethods-and-getdeclaredmethods)
