# Singleton Pattern

> 애플리케이션이 시작될 때, 어떤 클래스가 최초 한 번만 메모리를 할당(static)하고 해당 메모리에 인스턴스를 만들어 사용하는 패턴

즉, 싱글톤 패턴은 '하나'의 인스턴스만 생성하여 사용하는 디자인 패턴이다.
**인스턴스가 필요할 때, 똑같은 인스턴스를 만들지 않고 기존의 인스턴스를 활용하는 것 !** 

- 생성자가 여러번 호출돼도, 실제로 생성되는 객체는 하나이며 최초로 생성된 이후에 호출된 생성자는 이미 생성한 객체를 반환시키도록 만드는 것. 
- java에서는 생성자를 private 으로 선언해 다른 곳에서 생성하지 못하도록 만들고, `getInstance()` 메서드를 통해 받아서 사용하도록 구현

### 💡 왜 쓰나요 ?
>먼저, 객체를 생성할 때마다 메모리 영역을 할당받아야 한다. 하지만 한 번의 new를 통해 객체를 생성한다면 메모리 낭비를 방지할 수 있다. 
>또한 싱글톤으로 구현한 인스턴스는 '전역'이므로, 다른 클래스의 인스턴스들이 데이터를 공유하는 것이 가능한 장점이다. 
##### 1. 메모리 측면의 이점
싱글톤 패턴을 사용하게 된다면 한 개의 인스턴스만을 고정 메모리 영역에 생성하고, 추후 해당 객체에 접근할 때 메모리 낭비를 방지할 수 있다. 
##### 2. 속도 측면의 이점
생성된 인스턴스를 사용할 때는 이미 생성된 인스턴스를 활용하여 속도 측면에 이점이 있다. 
##### 3. 데이터 공유가 쉽다. 
전역으로 사용하는 인스턴스이기 때문에 다른 여러 클래스에서 데이터를 공유하며 사용할 수 있다. 하지만 동시성 문제가 발생할 수 있어 이 점은 유의하여 설계하여야 한다. 


### 💡 많이 사용하는 경우가 언제인가요 ?
주로 공통된 객체를 여러 개 생성해서 사용해야 하는 상황
- 데이터베이스에서 커넥션 풀, 스레드 풀, 캐시, 로그 기록 객체 등
- 안드로이드 앱 : 각 액티비티 들이나, 클래스마다 주요 클래스들을 하나하나 전달하는게 번거롭기 때문에 싱글톤 클래스를 만들어 어디서든 접근하도록 설계 
- 또한 인스턴스가 절대적으로 한 개만 존재하는 것을 보증하고 싶을 때 사용함

### 💡 싱글톤 패턴 구현하기
```java
public class Singleton {
	// 단 1개만 존재해야 하는 객체의 인스턴스로 static 으로 선언
	private static Singleton instance;
	
	// private 생성자로 외부에서 객체 생성을 막아야 한다. 
	private Singleton() {
	}
	
	// 외부에서는 getInstance() 로 instance 를 반환
	public static Singleton getInstance() {
		if (instance == null) {
			instance = new Singleton();
		}
		return instance;
	}
}
```
싱글톤 패턴의 기본적인 구현 방법은 다음과 같습니다. 

먼저 `private static` 으로 Singleton 객체의 instance를 선언하고 `getInstance()` 메서드가 처음 실행될 때만 하나의 instance가 생성되고 그 후에는 이미 생성되어진 instance를 return 하는 방식으로 진행이 됩니다. 

여기서 핵심은 `private` 으로 된 기본 생성자입니다. 생성자를 `private`로 생성을 하며 외부에서 새로운 객체의 생성을 막아줘야 합니다. 

```java
// 같은 instance 인지 Test
public class Application {
	public static void main(String[] args) {
		Singleton sintleton1 = Singleton.getInstance();
		Singleton sintleton2 = Singleton.getInstance();
		
		System.out.println(singleton1);
		System.out.println(singleton2);
		
		/** Output
         * Singleton@15db9742
         * Singleton@15db9742
         **/
	}
}
```
싱글톤 객체를 생성하는 위의 코드를 실행해보면 두 객체가 하나의 인스턴스를 사용하여 같은 주소 값을 출력하는 것을 확인할 수 있습니다. 

### 💡 Multi-thread 에서의 싱글톤
- multi-thread 환경에서 싱글 톤을 사용하게 된다면 다음과 같은 문제가 발생할 수 있습니다. 
##### 1. 여러 개의 인스턴스 생성
Multi-thread 환경에서 instance가 없을 때 동시에 아래의 `getInstance()` 메서드를 실행하는 경우 각각 새로운 instance를 생성할 수 있습니다. 
```java
public static Singleton getInstance() {
		if (instance == null) {
			instance = new Singleton();
		}
		return instance;
	}
```

##### 2. 변수 값의 일관성 실패
다음과 같은 코드가 실행이 되었을 때 여러 개의 thread 에서 `plusCount()` 를 동시에 실행한다면 일관되지 않은 값들이 생길 수 있습니다. 
```java
public class Singleton {
	private static Singleton instance;
	private static int count = 0;
	
	private Singleton() {
	}
	
	public static Singleton getInstance() {
		if (instance == null) {
			instance = new Singleton();
		}
		return instance;
	}
	
	public static void plusCount() {
		count ++;
	}
}
```

### 💡 해결법
##### 1. 정적 변수 선언에서 인스턴스 생성
이러한 문제는 아래와 같이 `static` 변수로 singleton 인스턴스를 생성하는 방법으로 해결할 수 있습니다. 아래와 같이 초기에 인스턴스를 생성하게 된다면 multi-thread 환경에서도 다른 객체들은 `getInstance()` 를 통해 하나의 인스턴스를 공유할 수 있습니다. 
```java
public class Singleton() {
	private static Singleton instance = new Singleton();
	
	private Singleton() {
	}
	
	public static Singleton getInstance() {
		return instance;
	}
}
```

##### 2. Lazy Initialization (게으른 초기화) - `synchronized` 의 사용
아래의 코드와 같이 `synchronized` 를 적용하여 multi-thread에서의 동시성 문제를 해결하는 방법입니다. 하지만 **해당 방법은 Thread-safe를 보장하기 위해 성능 저하가 발생**할 것입니다. 
```java
public class Singleton {
	public class Singleton {
		private static Singleton instance;
		
		private Singleton() {}
		
		public static synchronized Singleton getInstance() {
			if (instance == null) {
				instance = new Singleton();
			}
			return instance;
		}
	}
}
```

##### 2-1. + Double-checked Locking
- 위 방법의 성능 저하를 완화시키는 방법
```java
public class Singleton {
	public class Singleton {
		private static Singleton instance;
		
		private Singleton() {}
		
		public static Singleton getInstance() {
			if (instance == null) {
				synchronized (Singleton.class) {
					if (instance == null) {
						instance = new Singleton();
					}
				}
			}
			return instance;
		}
	}
}
```
먼저 조건문으로 인스턴스의 존재 여부를 확인한 다음 두 번째 조건문에서 `synchronized` 를 통해 동기화를 시켜 인스턴스를 생성합니다. 
스레드를 안전하게 만들면서, 처음 생성 이후에는 `synchronized` 를 실행하지 않기 때문에 성능저하가 완화가 가능합니다. 

##### 3. Intialization on demand holder idiom (holder에 의한 초기화)
클래스 안에 클래스(holder)를 두어 JVM의 클래스 로더 메커니즘과 클래스가 로드되는 시점을 이용한 방법
```java
public class Something {
	private Something() {
	}
	
	private static class LazyHolder {
		public static final Something INSTANCE = new Something();
	}
	
	public static Something getInstance() {
		return LazyHolder.INSTANCE;
	}
}
```
`getInstance()` 메서드를 호출 할 때 `LazyHolder` 가 로드되며, 이 때 `INSATANCE` 가 초기화됩니다. 그리고 `INSTANCE` 는 정적 변수이기 때문에 한 번 초기화되면 다시는 초기화되지 않습니다. 이는 **JVM이 보장하는 특성**입니다. 
더불어 `INSTANCE` 변수는 `final` 로 선언되어 있어 값이 변경될 수 없기 때문에 이는 싱글톤 인스턴스가 생성된 이후 다른 값으로 바뀌지 않는다는 것을 보장해줍니다. 

- **동기화 문제 해결** : 기존의 동기화 방식을 사용한 싱글톤 패턴에서는 동기화 블록을 통해 안전하게 객체를 초기화했지만, 이는 성능에 영향을 미칠 수 있습니다. 하지만 이 방법은 **동기화를 전혀 사용하지 않으면서도 스레드 안전성을 보장**합니다. 그 이유는 JVM이 클래스 로딩 시점을 제어하고 있기 때문입니다. 
- **게으른 초기화 (Lazy Initialization)** : 이 방식은 필요할 때까지 인스턴스를 생성하지 않습니다. 즉, `getInstance()` 메서드가 처음 호출되기 전까지는 인스턴스가 생성되지 않으므로, 메모리 사용이 최적화됩니다. 
- **단순성** : 코드가 단순하고 명확하며, 복잡한 동기화 코드를 작성할 필요가 없습니다. 따라서 코드 유지보수가 쉽습니다. 

> 동기화 문제를 **JVM의 책임**으로 넘겨서, 개발자가 복잡한 동기화 로직을 작성할 필요가 없이 안전하게 싱글톤 객체를 초기화할 수 있게 합니다. 또한, **JVM이 보장하는 클래스 초기화의 원자적 특성** 덕분에 여러 스레드가 동시에 `getInstance()` 를 호출하더라도 동기화 문제가 발생하지 않습니다. 
---
### 📌 `synchronized` 란?

> `synchronized`는 메서드나 블록을 하나의 스레드만 접근할 수 있도록 잠금을 걸어주는 키워드입니다. 이는 공유 자원에 대한 동시 접근을 제한하여 데이터의 일관성과 동기화 문제를 해결할 수 있게 해줍니다. `synchronized`를 사용하면 해당 코드 영역에 대한 접근을 단일 스레드로 제한하여, 여러 스레드가 동시에 같은 자원을 수정하는 것을 방지할 수 있습니다.

### 📌 클래스 초기화의 원자적 특성이란?

> '원자적'이란 용어는 '나눌 수 없는'이라는 뜻으로, 작업이 완전히 끝나거나 아예 시작되지 않는 상태를 의미합니다. 즉, 작업 중간에 멈추거나 중단되지 않습니다. 자바에서 클래스를 처음 사용할 때, JVM은 그 클래스를 '초기화'하는데, 이 초기화 과정이 원자적으로 이루어집니다. 이는 여러 스레드가 동시에 같은 클래스를 사용하려 해도, 초기화 과정이 한 번에 하나의 스레드에 의해서만 수행되고 완료될 때까지 다른 스레드는 기다려야 한다는 의미입니다.
---
[Ref1](https://velog.io/@seongwon97/%EC%8B%B1%EA%B8%80%ED%86%A4Singleton-%ED%8C%A8%ED%84%B4%EC%9D%B4%EB%9E%80#%EC%8B%B1%EA%B8%80%ED%86%A4-%ED%8C%A8%ED%84%B4%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0)
[Ref2](https://gyoogle.dev/blog/design-pattern/Singleton%20Pattern.html)
[클래스는 언제 로딩되고 초기화되는가](https://velog.io/@skyepodium/%ED%81%B4%EB%9E%98%EC%8A%A4%EB%8A%94-%EC%96%B8%EC%A0%9C-%EB%A1%9C%EB%94%A9%EB%90%98%EA%B3%A0-%EC%B4%88%EA%B8%B0%ED%99%94%EB%90%98%EB%8A%94%EA%B0%80#1-%EC%B4%88%EA%B8%B0%ED%99%94)