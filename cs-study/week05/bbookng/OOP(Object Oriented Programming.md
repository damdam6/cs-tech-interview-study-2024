# OOP (Object-Oriented Programming)

- 객체 지향 프로그래밍

- 객체의 관점에서 프로그래밍을 하는 것.

- 프로그래밍에서 필요한 데이터를 추상화시켜 객체를 만들고, 객체들 간의 상호작용을 통해 로직을 구성하는 프로그래밍 방법론

- Java, C++, Python ... 

  <img src="https://blog.kakaocdn.net/dn/catioE/btqKXiiBObK/unAbRuMcBYUuhHU3cjQOS1/img.png" alt="img" style="zoom:50%;" />

  

---

## 📌 절차적 프로그래밍 (Procedure Programming)

- 프로그램을 일련의 절차나 함수의 집합으로 보고, 이를 순차적으로 실행하여 문제를 해결하는 방식
- C언어

![img](https://blog.kakaocdn.net/dn/bhovFZ/btqK15vuG7x/oZaTsITHzEKxU1lmDnuR7k/img.png)

- **절차적 프로그래밍**

```
1. 강일구라는 사람은 머리 2개, 다리 3개, 팔 4개를 가지고 있고, 경찰이다.
2. 강일구.소개함수()
3. 여진구라는 사람은 머리 2개, 다리 3개, 팔 4개를 가지고 있고, 의사이다.
4. 여진구.소개함수()
```

- **객체 지향 프로그래밍**

```
1. class 정의 (인간 데이터, 함수, 추가 변수)
2. 강일구 = class 객체(경찰)
3. 여진구 = class 객체(의사)
5. 강일구.소개함수()
6. 여진구.소개함수()
```



### 💡 절차적 프로그래밍의 장단점

#### 1. 장점

- 객체나 클래스를 만들 필요 없이 바로 프로그램을 코딩할 수 있다. 
- 필요한 기능을 함수로 만들어 두기 때문에 같은 코드를 복사하지 않고 호출하여 사용할 수 있다. 
- 프로그램의 흐름을 쉽게 추적할 수 있다. 

#### 2. 단점

- 각 코드가 매우 유기성이 높기 때문에 수정하기가 힘들다. (새로운 데이터나 기능을 추가하기가 어려움)
- 프로그램 전체에서 코드를 재사용할 수 없어 개발 비용과 시간이 늘어난다.
- 디버깅이 어렵다.



---

## 📌 OOP의 5대 원칙

### 1. 단일 책임 원칙 (Single Responsibility Principle)
- 모든 클래스는 각각 하나의 책임만 가져야 한다. 
- 모듈이 변경되는 이유가 한 가지 여야 함을 뜻함.
- 한 클래스는 한 가지 책임에 관한 변경사항이 생겼을 때만 코드를 수정하게 되는 구조가 좋은 구조.

### 2. 개방-폐쇄 원칙 (Open Closed Principle)
- 확장에는 열려있고, 수정에는 닫혀있는. 기존의 코드를 변경하지 않으면서(Closed), 기능을 추가할 수 있도록(Open) 설계가 되어야 한다는 원칙. 
- 기능 추가 요청이 오면 확장을 통해 손쉽게 구현하면서, 확장에 따른 클래스 수정은 최소화 하도록. 
- **확장에 열려있다.**
  - 모듈의 확장성 보장.
  - 새로운 변경 사항이 발생했을 때 유연하게 코드를 추가함으로써 어플리케이션의 기능을 큰 힘을 들이지 않고 확장

- **변경에 닫혀있다.**
  - 객체를 직접적으로 수정하는 것은 제한.
  - 객체를 직접 수정하지 않고도 변경사항을 적용할 수 있도록 설계해야 함. 

- **OCP를 위반한 예시**

```java
class Animal {
	String type;
    
    Animal(String type) {
    	this.type = type;
    }
}

// 동물 타입을 받아 각 동물에 맞춰 울음소리를 내게 하는 클래스 모듈
class HelloAnimal {
    void hello(Animal animal) {
        if(animal.type.equals("Cat")) {
            System.out.println("냐옹");
        } else if(animal.type.equals("Dog")) {
            System.out.println("멍멍");
        }
    }
}

public class Main {
    public static void main(String[] args) {
        HelloAnimal hello = new HelloAnimal();
        
        Animal cat = new Animal("Cat");
        Animal dog = new Animal("Dog");

        hello.hello(cat); // 냐옹
        hello.hello(dog); // 멍멍
    }
}


// 새로운 동물을 추가하려면 ? 
class HelloAnimal {
	// 기능을 확장하기 위해서는 클래스 내부 구성을 일일히 수정해야 하는 번거로움이 생긴다.
    void hello(Animal animal) {
        if (animal.type.equals("Cat")) {
            System.out.println("냐옹");
        } else if (animal.type.equals("Dog")) {
            System.out.println("멍멍");
        } else if (animal.type.equals("Sheep")) {
            System.out.println("메에에");
        } else if (animal.type.equals("Lion")) {
            System.out.println("어흥");
        }
        // ...
    }
}

public class Main {
    public static void main(String[] args) {
        HelloAnimal hello = new HelloAnimal();

        Animal cat = new Animal("Cat");
        Animal dog = new Animal("Dog");

        Animal sheep = new Animal("Sheep");
        Animal lion = new Animal("Lion");

        hello.hello(cat); // 냐옹
        hello.hello(dog); // 멍멍
        hello.hello(sheep); 
        hello.hello(lion);
    }
}
```

- OCP 를 따른 예시

```java
// 추상화
abstract class Animal {
    abstract void speak();
}

class Cat extends Animal { // 상속
    void speak() {
        System.out.println("냐옹");
    }
}

class Dog extends Animal { // 상속
    void speak() {
        System.out.println("멍멍");
    }
}

class HelloAnimal {
    void hello(Animal animal) {
        animal.speak();
    }
}

public class Main {
    public static void main(String[] args) {
        HelloAnimal hello = new HelloAnimal();

        Animal cat = new Cat();
        Animal dog = new Dog();

        hello.hello(cat); // 냐옹
        hello.hello(dog); // 멍멍
    }
}

// 위와 같이 설계한다면 

// 추상클래스를 상속만 하면 메소드 강제 구현 규칙으로 규격화만 하면 확장에 제한 없다 (opened)
class Sheep extends Animal {
    void speak() {
        System.out.println("매에에");
    }
}

class Lion extends Animal {
    void speak() {
        System.out.println("어흥");
    }
}

// 기능 확장으로 인한 클래스가 추가되어도, 더이상 수정할 필요가 없어진다 (closed)
class HelloAnimal {
    void hello(Animal animal) {
        animal.speak();
    }
}

public class Main {
    public static void main(String[] args) {
        HelloAnimal hello = new HelloAnimal();

        Animal cat = new Cat();
        Animal dog = new Dog();

        Animal sheep = new Sheep();
        Animal lion = new Lion();

        hello.hello(cat); // 냐옹
        hello.hello(dog); // 멍멍
        hello.hello(sheep); // 매에에
        hello.hello(lion); // 어흥
    }
}
```



### 3. 리스코프 치환 원칙 (Liskov Substitution Principle)
- 자식 클래스는 언제나 자신의 부모 클래스를 대체할 수 있다는 원칙. 즉, 부모 클래스가 들어갈 자리에 자식 클래스를 넣어도 계획대로 잘 작동해야 한다. 
- 자식클래스는 부모 클래스의 책임을 무시하거나 재정의하지 않고 확장만 수행하도록 해야 LSP를 만족한다. 

![img](https://velog.velcdn.com/images/harinnnnn/post/2f2c6e85-553f-4afa-83c1-2d2abf722cea/image.png)

### 4. 인터페이스 분리 원칙 (Interface Segregation Principle)
- 한 클래스는 자신이 사용하지 않는 인터페이스는 구현하지 말아야 한다.
- 인터페이스를 잘게 분리함으로써 클라이언트의 목적과 용도에 적합한 인터페이스만을 제공하는 것. 
- 반드시 객체가 자신에게 필요한 기능만을 가지도록 제한하는 것.
- 불필요한 상속과 구현을 최대한 방지함으로써 객체의 불필요한 책임을 제거. 

![img](https://velog.velcdn.com/images/harinnnnn/post/f56cf080-bd64-4714-86fa-05c2ed3dda33/image.png)

- User1, User2, User3 모두가 OPS를 상속받고 있지만 각각의 객체들은 오직 op1, op2, op3 메서드만 사용함. 그렇다면 다른 두 개의 메서드를 사용하지 않음에도 불구하고 사용하지 않는 메서드에 변경이 일어나면 영향을 받게됨.



- ISP 를 위반한 예시

```java
// 스마트폰 인터페이스
interface ISmartPhone {
    void call(String number); // 통화 기능
    void message(String number, String text); // 문제 메세지 전송 기능
    void wirelessCharge(); // 무선 충전 기능
    void AR(); // 증강 현실(AR) 기능
    void biometrics(); // 생체 인식 기능
}

// 최신 폰들은 가능
class S20 implements ISmartPhone {
    public void call(String number) {
    }

    public void message(String number, String text) {
    }

    public void wirelessCharge() {
    }

    public void AR() {
    }

    public void biometrics() {
    }
}

class S21 implements ISmartPhone {
    public void call(String number) {
    }

    public void message(String number, String text) {
    }

    public void wirelessCharge() {
    }

    public void AR() {
    }

    public void biometrics() {
    }
}

// 하지만 구식은 안됨. 쓸데 없는 메서드를 구현해야 함. 
class S3 implements ISmartPhone {
    public void call(String number) {
    }

    public void message(String number, String text) {
    }

    public void wirelessCharge() {
        System.out.println("지원 하지 않는 기능 입니다.");
    }

    public void AR() {
        System.out.println("지원 하지 않는 기능 입니다.");
    }

    public void biometrics() {
        System.out.println("지원 하지 않는 기능 입니다.");
    }
}
```



- ISP 를 지킨 예시

```java
// 기능별로 인터페이스 분리
interface IPhone {
    void call(String number); // 통화 기능
    void message(String number, String text); // 문제 메세지 전송 기능
}

interface WirelessChargable {
    void wirelessCharge(); // 무선 충전 기능
}

interface ARable {
    void AR(); // 증강 현실(AR) 기능
}

interface Biometricsable {
    void biometrics(); // 생체 인식 기능
}

// 각각 필요한 기능만 implements 하면 됨
class S21 implements IPhone, WirelessChargable, ARable, Biometricsable {
    public void call(String number) {
    }

    public void message(String number, String text) {
    }

    public void wirelessCharge() {
    }

    public void AR() {
    }

    public void biometrics() {
    }
}

class S3 implements IPhone {
    public void call(String number) {
    }

    public void message(String number, String text) {
    }
}
```



### 5. 의존 역전 원칙 (Dependency Inversion Principle)
- 의존 관계를 맺을 때 변화하기 쉬운 것 또는 자주 변화하는 것 보다는 변화하기 어려운 것, 거의 변화가 없는 것에 의존하라는 것. 
- 구체적인 클래스보다 인터페이스나 추상 클래스와 관계를 맺으라는 것. 
- 고수준 모듈은 저수준 모듈의 구현에 의존해서는 안된다. 대신 저수준 모듈이 고수준 모듈에서 정의한 **추상 타입**에 의존해야 한다.

> 고수준 모듈 : 어떤 의미 있는 단일 기능을 제공하는 모듈 (Interface, 추상 클래스)
>
> 저수준 모듈 : 고수준 모듈의 기능을 구현하기 위해 필요한 하위 기능의 실제 구현 (메인 클래스, 객체)

- 어떤 클래스를 참조해서 사용해야 하는 상황이 생긴다면, 그 클래스를 직접 참조하는 것이 아니라 그 대상의 상위 요소(추상 클래스 Or 인터페이스)로 참조하라는 원칙. 



- DIP를 위반한 예시

```java
public class Kid {
    private Robot toy;

    public void setToy(Robot toy) {
        this.toy = toy;
    }

    public void play() {
        System.out.println(toy.toString());
    }
}

public class Main{
    public static void main(String[] args) {
        Robot robot = new Robot();
        Kid k = new Kid();
        k.setToy(robot);
        k.play();
    }
}

// Kid는 robot 이라는 구체적인 객체에 의존하고 있음. 
// 새로운 장난감이 추가되면 아래와 같이 Kid 클래스를 수정해야 함. 

public class Kid {

    private Robot toy;
    private Lego toy; //레고 추가
    
    // 아이가 가지고 노는 장난감의 종류만큼 Kid 클래스 내에 메서드가 존재해야함.
    public void setToy(Robot toy) {
        this.toy = toy;
    }
    public void setToy(Lego toy) {
        this.toy = toy;
    }

    public void play() {
        System.out.println(toy.toString());
    }
    
}
```

- DIP를 적용한 예시

```java
// Toy 라는 추상클래스와 의존관계를 맺도록 설계
public class Kid {
    private Toy toy;

    public void setToy(Toy toy) {
        this.toy = toy;
    }

    public void play() {
        System.out.println(toy.toString());
    }
}

public class Robot extends Toy {
    public String toString() {
        return "Robot";
    }
}

// 새로운 장난감을 추가하고 싶다면 ? 

public class Lego extends Toy {
    public String toString() {
        return "Lego";
    }
}

public class Main{
    public static void main(String[] args) {
        Toy lego = new Lego();
        Kid k = new Kid();
        k.setToy(lego);
        k.play();
    }
}
```





### * 정리
- SRP : 어떤 클래스를 변경해야 하는 이유는 오직 하나뿐이어야 한다. 
- OCP : 자신의 확장에는 열려 있고, 주변의 변화에 대해서는 닫혀 있어야 한다. 
- LSP : 서브 타입은 언제나 자신의 기반 타입으로 교체할 수 있어야 한다.
- ISP : 범용적인 인터페이스 하나보다 내가 쓸 인터페이스 따로 만들어라. 
- DIP : 구현체에 의존하지 말고 인터페이스에 의존해라. 

---



## 📌 OOP의 특징

### 1. 추상화 (Abstraction)
- 특정 개념이나 개체를 보았을 때 특정 관점에서 관심있거나 중요한 부분만 추려내는 작업.
- 어떤 대상/집단의 공통적이고 본질적인 특징을 추출하여 정의한 것.
- 어떤 대상을 구현할 때, 그 대상의 본질적인 특징을 정의하고, 이것에 기반하여 대상을 객체로 구현하는 것. 
- 이 때, 본질적인 특징을 정의하는 데 활용되는 개념이 **abstract class / interface**
- 공통의 속성이나 기능을 묶어 이름을 붙이는 것
- 객체 지향 프로그래밍에서 클래스를 정의하는 것을 추상화라고 한다. 
### 2. 캡슐화 (Encapsulation)
- 데이터와 프로세스를 하나의 객체에 위치하도록 만드는 것.
- 클래스 내의 연관된 속성(property)이나 함수(method)를 하나의 캡슐로 묶어 외부로부터 클래스로의 접근을 최소화하는 것.
- **하나의 목적**을 위해 필요한 데이터나 메소드를 하나로 묶는 것.
- 데이터는 외부에서 해당 객체에 접근 시 직접 접근이 아닌 함수를 통해서만 접근하여야 한다. 
- 장점
	- 외부로부터 클래스의 변수, 함수를 보호
	- 외부에는 필요한 요소만 노출하고, 내부의 상세한 동작은 은닉
- 접근제한자
	- `public` : 하위 클래스와 인스턴스에서 접근 가능
	- `static` : 클래스와 외부에서 접근 가능. 인스턴스에서는 접근 불가
	- `private` : 클래스 본인만 접근 가능, 하위클래스, 인스턴스 접근 불가
	- `protected` : 클래스 본인, 하위 클래스에서 접근 가능, 인스턴스 접근 불가 
- 유지보수가 용이하다.
### 3. 상속 (Inheritance)
- 부모 객체의 특징을 그대로 물려받는 것, 자바에서는 인터페이스 상속과 클래스 상속으로 나뉨.
- 대상을 객체로 추상화/구현할 때, 기존에 구현한 클래스를 재활용하여 구현할 수 있는 것. 
- 기존 메서드와 변수를 물려받되, 필요한 기능을 더 추가하거나 자식클래에 맞게 재정의하는 방법

### 4. 다형성 (Polymorphism)
- 같은 요청으로부터 응답이 객체의 타입에 따라 나르게 나타나는 것. 
- 어떤 객체의 속성이나 기능이 상황에 따라 여러 형태로 변할 수 있다는 것.
- 메서드 오브라이딩/오버로딩
- 개발 유연성, 코드 재사용성을 제고시킬 수 있음. 
- 상위 객체의 타입으로 하위 객체를 참조할 수 있음. 

---



## 📌 OOP의 장점과 단점

### 1. 장점

- 코드 재사용성 증가
  - 클래스를 가져와서 사용할 수 있고 상속을 통해 확장해서 사용할 수 있음
- 생산성 향상 및 대형 프로젝트에 적합
  - 클래스 단위로 모듈화 시켜서 개발할 수 있으므로 대형 프로젝트에 적합
- 유지보수가 용이하다
  - 캡슐화를 통해 주변에 영향을 거의 미치지 않기 때문에 문제가 생기는 클래스만 고치면 됨



### 2. 단점

- 설계 시 많은 시간과 노력이 필요
  - 다양한 객체들의 상호작용을 통해 프로그램이 구성되므로 설계에 많은 시간과 노력이 필요
- 실행 속도가 상대적으로 느리다
  - 클래스로 정의하고 그 안에서 또 변수와 함수를 만들 경우에 이들을 메모리에 저장해야 함. 객체가 호출 받으면 `객체 호출 - 스택 저장 - 지역 변수 저장 - 코드 실행 - 반환 값 return - 호출 정보 제거` 과정을 거쳐야하기 때문에 절차지향보다 느림

