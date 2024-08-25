# Error Handling

Date: July 29, 2024 → July 30, 2024
Status: In progress
Categories: Back-study:task
Tags: Backend, CS학습, Tech면접

# Clean Code 번역하기

## Chapter 7: Error Handling

 클린 코드에 관한 책에서 에러 핸들링에 관한 색션이 있는 것은 이상해 보일 수 있다. 에러 핸들링은 우리가 프로그래밍을 할 때 우리가 해야 하는 것들 중 하나일 뿐이다. 입력(Input)은 비정상적일 수 있고 장치가 실패할 수 있다. 간단히 말해서, 상황이  잘못될 수 있으며 , 그럴 때 프로그래머로서 우리는 우리의 코드가 해야할 일을 제대로 수행하도록 할 책임이 있다.

 클린 코드와의 연결은, 그렇지만, 분명해야 한다. 많은 코드 베이스는 완벽하게 에러 핸들링에 지배당한다. 내가 지배당한다고 말하는 것은, 에러 핸들링이 그들이 하는 모든 것이란 의미는 아니다. 나는 모든 흩어진 에러 핸들링 탓에 코드가 하는 일을 거의 알기 어렵다는 것을 의미한다. 에러 핸들링은 중요하지만, 만약 그것이 로직을 가린다면 그건 잘못되었다.

In this chapter I’ll outline a number of techniques and considerations that you can use to write code that is both clean and robust—code that handles errors with grace and style.

### Use Exceptions Rather Than Return Codes

### Return 코드 대신 예외처리를 사용해라

과거에는 예외가 없는 언어들이 많았다. 그런 언어에서 에러 헨들링과 리포팅 방식은 한정적이었다. 너는 호출자(caller)가 확인할 수 있는 error flag를 세팅하거나 error code를 반환했다.

```java
public class DeviceController {
 ...
 public void sendShutDown() {
 DeviceHandle handle = getHandle(DEV1);
 // Check the state of the device
 if (handle != DeviceHandle.INVALID) {
 // Save the device status to the record field
 retrieveDeviceRecord(handle);
 // If not suspended, shut down
 if (record.getStatus() != DEVICE_SUSPENDED) {
 pauseDevice(handle);
 clearDeviceWorkQueue(handle);
 closeDevice(handle);
 } else {
 logger.log("Device suspended. Unable to shut down");
 }
 } else {
 logger.log("Invalid handle for: " + DEV1.toString());
 }
 }
 ...
}
```

이러한 접근의 문제는 이것들이 호출자를 어지럽힌다는 점이다. 호출자는 호출 후 즉시 에러를 체킹해야 한다. 불행하게도, 이건 잊어버리기 쉽다. 이러한 이유 때문에, 당신이 에러를 맞닥뜨렸을 때 예외를 던지는 것이 더 낫다. 호출 코드가 더 클린하다. 로직이 에러 핸들링으로 인해 가려지지 않는다.

```java
public class DeviceController {
 ...
 public void sendShutDown() {
 try {
 tryToShutDown();
 } catch (DeviceShutDownError e) {
 logger.log(e);
 }
 }
 private void tryToShutDown() throws DeviceShutDownError {
 DeviceHandle handle = getHandle(DEV1);
 DeviceRecord record = retrieveDeviceRecord(handle);
 pauseDevice(handle);
 clearDeviceWorkQueue(handle);
 closeDevice(handle);
 }
 private DeviceHandle getHandle(DeviceID id) {
 ...
 throw new DeviceShutDownError("Invalid handle for: " + id.toString());
 ...
 }
 ...
}
```

이게 얼마나 더 깔끔한지 보라. 이건 단순히 미학적인 문제가 아니다. 코드가 더 낫다. 왜냐하면 얽혀있던 장치 종료 알고리즘과 에러 핸들링이 이제 분리되었기 때문이다. 이제 각각의 관심사를 보고 독립적으로 이해할 수 있다.

### Write Your `Try-Catch-Finally` Statement First

### `Try-Catch-Finally`를 먼저 써라

예외에 관한 가장 흥미로운 점 중 하나는 그것들이 너의 프로그램 내의 범위를 정의한다는 점이다. `try-catch-finally` 의 `try` 부분에서 코드를 실행할 때, 실행은 어느 시점에서든 중단될 수 있으며 `catch` 부분에서 다시 시작할 수 있음을 의미한다. 

이러한 면에서 `try` 블록은 거래와 같다. `catch`는 `try`에서 무슨 일이 일어나는지와 관계 없이 프로그램을 일관된 상태로 유지해야 한다. 이러한 이유로 예외를 던질 수 있는 코드를 쓸 때, `try-catch-finally`로 시작하는 것은 좋은 연습입니다. 이것은 당신이 그 코드의 사용자가 무엇을 기대해야 하는 지를 정의하는데, `try`에서 실행되는 코드에서 어떤 문제가 생기더라도 상관 없다.

예시를 한 번 보자. 우리는 파일에 접근하여 직렬화 된 객체를 읽는 코드를 써야한다.

우리는 파일이 존재하지 않을 때 예외를 받는 것을보여주는 테스트에서 시작할 것이다.

```java
 @Test(expected = StorageException.class)
 public void retrieveSectionShouldThrowOnInvalidFileName() {
 sectionStore.retrieveSection("invalid - file");
 }
```

```java
public List<RecordedGrip> retrieveSection(String sectionName) {
 // dummy return until we have a real implementation
 return new ArrayList<RecordedGrip>();
}
```

우리의 테스트는 예외를 발생 시키지 않기 때문에 실패했다. 다음으로, 우리는 우리의 유효하지 않은 파일에 접근 시도를 하는 구현을 변경할 것이다.

```java
public List<RecordedGrip> retrieveSection(String sectionName) {
 try {
 FileInputStream stream = new FileInputStream(sectionName)
 } catch (Exception e) {
 throw new StorageException("retrieval error", e);
 }
 return new ArrayList<RecordedGrip>();
}
```

이제, 우리의 테스트는 통과했다. 예외를 잡았기 때문이다. 이 시점에서 우리는 리팩토링을 할 수 있다. 우리는 fileInputStream 생성자에서 실제로 발생하는 유형인 FileNotFoundExcepton에 맞춰, 우리가 잡는 예외의 타입을 좁힐 수 있다.

```java
public List<RecordedGrip> retrieveSection(String sectionName) {
 try {
 FileInputStream stream = new FileInputStream(sectionName);
 stream.close();
 } catch (FileNotFoundException e) {
 throw new StorageException("retrieval error”, e);
 }
 return new ArrayList<RecordedGrip>();
}
```

이제 우리는 `try-catch` 구조를 통해 범위를 결정했고, 우리가 필요한 나머지 로직에 대해 TDD를 사용할 수 있다. 로직은 `FileInputStream`과 `close` 사이에 더해질 수 있으며 어떤 것도 잘못되지 않았다고 취급할 수 있다.

예외를 강제하는 테스트를 쓰도록 하고, 당신의 핸들러가 당신의 테스트를 만족시킬 수 있는 동작을 더해라. 이것은 try 블록의 트랜잭션 범위를 먼저 설계하도록 하고, 트랜잭션의 특성을 유지하는데 도움이 된다.

### Use Unchecked Exceptions

Uncheck 예외를 사용하기

```java
              ---> Throwable <--- 
              |    (checked)     |
              |                  |
              |                  |
      ---> Exception           Error
      |    (checked)        (unchecked)
      |
RuntimeException
  (unchecked)
```

토론은 끝났다. 자바 프로그래머들은 몇 년 동안 checked 예외의 장단점에 대해 토론해왔다. checked 예외가 자바 첫 번째 버전에서 소개 되었을 때, 좋은 아이디어처럼 보였다. 모든 메서드의 시그니처에는 호출자에게 전달할 수 있는 예외가 나열되어야 한다. 더욱이 이러한 예외는 메소드 타입의 한 부분이다. 너의 코드는 메서드 시그니처가 코드에서 발생할 수 있는 것과 일치하지 않으면 컴파일 되지 않는다.

그 당시에는 checked 예외가 좋은 생각이라고 생각했다. 그리고 그렇다, 그것들은 몇가지 이점을 제공한다. 그러나 이제 그것들은 견고한 소프트웨어를 제작하는데 필수적이지 않다는게 분명하다. C#에는 checked 예외가 없으며, 여러차례의 시도에도 불구하고 C++ 또한 없다. 그렇지만 python이나 ruby에도 없다. 견고한 소프트웨어는 이 모든 언어에서 가능하다. 그렇기 때문에, 우리는 checked 예외가 그 대가에 합당한 값어치가 있는지 진지하게 결정해야한다.

무슨 대가? checked exception의 대가는 Open/Close principle(OCP 원칙) 위반이다. 만약 네가 checked 예외를 너의 코드의 메소드에서 보내고, `catch`가 3 level위에 있다면, 그 예외를 잡는 catch 구문사이의 모든 메서드 시그니처에 해당 예외를 선언해야 한다. 이건 낮은 레벨의 소프트웨어의 변화가 많은 높은 레벨의 시그니처 변화를 강제한다는 것을 의미한다. 변경된 모듈은 아누런 관련 사항이 변경되지 않았더라도 다시 빌드하고 재배포 해야한다.

큰 시스템의 호출 계층을 고려해보자. 최상위 함수가 그 아래 함수를 호출하면 그 함수는 그것들 아래의 함수를 더 호출하고, 이는 무한히 이어진다. 이제 하위 함수가 예외를 던져야 하는 방식으로 수정되었다고 가정해보자. 만약 그 예외가 checked라면 함수 시그니처는 `throws` 절을 추가해야 한다. 하지만 이건, 우리가 수정한 함수를 부르는 모든 함수가 또한 새로운 예외를 catch하거나 throw절을 추가하기 위해 수정되야 하는 것을 의미한다. 무한히. 그 결과 최하위 렙레에서 최상위 레벨까지 변경사항이 연쇄적으로 발생한다. 모든 경로의 함수가 낮은 레벨의 예외 디테일을 알아야 하기 때문에 캡슐화가 깨진다. 예외처리의 목적이 거리를 둔 곳에서도 오류를 처리할 수 있도록 하는 것임을 고려할때, checked 예외가 이러한 캡슐화를 깨트리는 건 안타까운 일이다.

checked 예외는 중요한 라이브러리를 작성할 때 유용할 수 있다. 너는 반드시 이를 처리 해야 하기 때문이다. 그러나 일반적인 애플리케이션 개발에서는 의존성 비용이 이점보다 크다.

### Provide Context with Exceptions

### 예외에 맥락을 제공해라

네가 던지는 모든 예외는 소스와 에러의 위치를 결정할 수 있는 충분한 문맥을 제공해야 한다. 자바에서, 너는 stack trace를 모든 예외에서 확인할 수 있다. 하지만 stack trace는 실패한 작업의 의도를 설명할 수 없다.

정보가 담긴 에러 메시지를 만들고 너의 예외에 보내라. 실패한 작업과 실패 종류를 언급해라. 만일 너가 너의 애플리케이션에서 로깅하는 경우, 너의 catch에서 에러를 로깅 하기에 충분한 정보를 함께 보내라. 

### Define Exception Classes in Terms of a Caller’s Needs

### 호출자의 필요에 따른 예외 클래스를 정의해라

에러를 구분하는 많은 방법이 있다. 우리는 그들의 소스로 구분할 수 있다. 그것들이 하나의 컴포넌트에서 왔는가, 아니면 다른 곳에서 왔는가? 혹은 그들의 종류에 기반하자. 그것들의 기기 실패, 네트워크 실패, 혹은 프로그래밍 에러인가? 하지만 우리가 애플리케이션에서 예외 클래스를 정의할 때, 우리의 가장 중요한 고려는 그들이 어떻게 잡히는가이다.

나쁜 예외 클래스화 예시를 보자. 여기 서드파티라이브러리를 부를때의  `try-catch-finally` 문이있다. 이것은 그 호출이 부를 수 있는 모든 예외를 커버한다.

```java
ACMEPort port = new ACMEPort(12);
 try {
 port.open();
 } catch (DeviceResponseException e) {
 reportPortError(e);
 logger.log("Device response exception", e);
 } catch (ATM1212UnlockedException e) {
 reportPortError(e);
 logger.log("Unlock exception", e);
 } catch (GMXError e) {
 reportPortError(e);
 logger.log("Device response exception");
 } finally {
 …
 }
```

이러한 구문은 많은 중복을 포함하고 놀라서는 안된다. 대부분의 예외 처리 상황에서, 실제 원인에 관계 없이 수행하는 작업이 비교적 표준적이다. 우리는 오류를 기록하고 계속 진행할 수 있도록 해야한다.

이 상황에서, 우리는 우리가 하는 것이 예외 상황과 관련 없이 대략적으로 같다는 사실을 알고 있기 때문에, 호출하는 api를 감싸고 공통 예외 유형을 반환하도록 함으로써 코드의 상당한 단순화가 가능해진다.

```java
LocalPort port = new LocalPort(12);
 try {
 port.open();
 } catch (PortDeviceFailure e) {
 reportError(e);
 logger.log(e.getMessage(), e);
 } finally {
 …
 }
 
 
 public class LocalPort {
 private ACMEPort innerPort;
 public LocalPort(int portNumber) {
 innerPort = new ACMEPort(portNumber);
 }
 public void open() {
 try {
 innerPort.open();
 } catch (DeviceResponseException e) {
 throw new PortDeviceFailure(e);
 } catch (ATM1212UnlockedException e) {
 throw new PortDeviceFailure(e);
 } catch (GMXError e) {
  throw new PortDeviceFailure(e);
 }
 }
 …
}
```

우리가 `ACMEPort` 라고 정의한 것과 같은 래퍼는 매우 융요하다. 사실 third-party API들을 래핑하는 것이 가장 좋은 연습이다. third-party-api를 래핑한다면 너는 너의 의존성을 줄일 수 있고, 다른 문제 없이 미래에 다른 라이브러리로 옮겨갈 수 있다. 래핑은 또한 third-party 호출을 모방하여 너의 코드를 테스트할 때 더 쉬워진다.

래핑의 또 다른 장점은 너는 특정한 벤더의 api 디자인 선택에 얽매일 필요가 없다는 것이다. 너느 네가 편한 api를 고를 수 잇다. 예시에 따름녀 우리는 하나의 예시 유형인 `port` 기기 실패를 정의했고, 우리가 더 클린한 코드를 짤 수 있다는 걸 발견했다.

(*One final advantage of wrapping is that you aren’t tied to a particular vendor’s API
design choices. You can define an API that you feel comfortable with. In the preceding
example, we defined a single exception type for port device failure and found that we
could write much cleaner code.)*

자주 하나의 예외 클래스가 특정 코드영역에서는 충분한 경우가 많다. 예외와 전달되는 정보가 에러를 구별할 수 있다. 다른 예외를 잡지 않고 통과 시키고 싶을 때만 다른 예외 클래스를 사용하라.

### Define the Normal Flow

### 정상 흐름을 정의하자

만일 네가 이전 섹션에서 조언한 내용을 따랐으면, 비즈니스 로직과 예외 핸들링 사이에 잘 분리된 결과에 도달하게 될 것이다. 코드 대부분이 깔끔하고 불필요한 장식이 없는 알고리즘처럼 보이기 시작할 것이다. 하지만 이렇게 하므로서 에러 감지는 프로그램의 가장자리로 밀려난다. 외부 api를 래핑하여 자체 예외를 던지고, 코드 상단에 handler을 정의함으로써 중단된 연산 처리가 가능하다. 대부분의 경우 이건 좋은 접근이지만, 때때로 중단을 원하지 않을 수 있다.

예시를 보자. 여기 청구 애플리케이션에서 비용을 합산하는 이상한 코드이다.

```java
try {
 MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
 m_total += expenses.getTotal();
} catch(MealExpensesNotFound e) {
 m_total += getMealPerDiem();
}
```

이러한 사업에서 식비가 청구되면 총액에 포함된다. 식비가 청구되지 않으면 직원은 그날의 일일 식비를 받는다. 예외가 로직을 어지럽힌다. 특수한 케이스를 처리할 필요가 없으면 더 낫지 않을까? 만약 우리가 그렇지 않다면, 우리의 코드는 더 단순해보일 것이다.

```java
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();
```

우리가 코드를 깔끔하게 만들 수 있을 것인가? 우리는 할 수 있다는 것이 밝혀졌다. 우리는 ExpenseReportDAO 를 바꿔서 Meal Expense 객체를 항상 반환하도록 한다. 만일 meal expenses,가 없다면 per diem 총계로 반환하는 MealExpense 객체를 반환한다.

```java
public class PerDiemMealExpenses implements MealExpenses {
 public int getTotal() {
 // return the per diem default
 }
}
```

이건 ‘특수 사례 패턴’이라고 한다. 특수한 경우를 처리하도록 클래스나 객체를 생성하여 특수한 케이스를 처리하도록 한다. 그럼으로써 클라이언트 코드가 예외 동작을 처리할 필요가 없다. 그 동작은 특수 사례 객체에 캡슐화 된다.

### Don’t Return Null

### null을 반환하지 말것

나는 에러 핸들링의 어떠한 논의에서도 우리가 오류를 유발하는 행동에 대한 언급을 포함해야 한다고 생각한다. 그 중 하나는 `null` 을 반환하는 것이다. 나는 `null` 을 확인하는 내용이 담긴 코드를 거의 한 줄마다 가진 애플리케이션을 셀 수 없을 만큼 많이 봤다.

```java
 public void registerItem(Item item) {
 if (item != null) {
 ItemRegistry registry = peristentStore.getItemRegistry();
 if (registry != null) {
 Item existing = registry.getItem(item.getID());
 if (existing.getBillingPeriod().hasRetailOwner()) {
 existing.register(item);
 }
 }
 }
 }
```

만약 네가 코드 베이스가 이렇게 생긴 코드에 대해 일하고 있다면, 그렇게 나빠보이진 않을 수 있지만 그건 나쁘다! 우리가 `null` 을 반환할 때, 우리는 본질적으로 우리의 일을 만들고 호출자에게 문제를 넘기고 있다. null 체크가 누락 되어도 애플리케이션이 통제 불능 상태에 빠질 수 있다.

중첩된 if문에서 두 번째 줄에 null 체크가 없다는 사실을 발견했는가? 만약 peristentStore가 null이라면 런타임에 어떤 일이 발생했을까? 우리는 런타임에 nullPointerException을 겪게 될 것이며 누군가는 최상위 수준에서 nullpointeresception을 잡고 있어야 할 것이다. 둘 다 별로인 방식이다. 애플리케이션 깊숙한 곳에서 nullpointerException이 발생하면 대체 어떻게 정확히 대응해야할까?

위의 코드의 문제점이 null 체크를 빼먹은데 문제가 있다고 이야기하는 것은 쉬우나, 사실, 문제는 그게 너무 많다는데 있다. 만일 null을 반환하고자 하는 유혹을 느낀다면, 예외를 던지거나 특수 케이스 객체를 대신 반환하라. 만일 너가 null-반환 메소드를 third-party-api에서 부른다면, 그 메소드를  예외를 던지고 특수 객체를 반환하는 형태로 래핑하는 것을 고려해라.

많은 경우, 특수 케이스 객체는 쉬운 해결책이 된다. 

```java
List<Employee> employees = getEmployees();
if (employees != null) {
 for(Employee e : employees) {
 totalPay += e.getPay();
 }
}
```

지금은 getEmployees 가 null을 반환할 수 있다. 그렇지만 그렇게 해야할까? 만일 빈 배열을 반환하도록 한다면 우리는 코드를 깨끗하게 할 수 있다.

```java
List<Employee> employees = getEmployees();
for(Employee e : employees) {
 totalPay += e.getPay();
}
```

운이 좋게도, 자바는 collections.emptyList()를 가지고 있다. 그리고 그건 목적을 위해 사용할 수 있는 미리 정의된 불변의 리스트를 반환한다.

```java
public List<Employee> getEmployees() {
 if( .. there are no employees .. )
 return Collections.emptyList();
}
```

이런 식으로 코드를 짜면, NullPointerExceptions의 발생 기회를 최소화할 수 있고 코드가 클린해질 것이다.

### Don’t Pass Null

### null을 넘기지 말것

메소드에서 `null` 을 반환하는 것은 나쁘지만, null을 전달 하는 것은 더 나쁘다. 네가 null을 넘겨야 하는 API로 작업을 하지 않는 이상, 너는 null을 넘기는 작업은 어디서는 하지 않아야 한다.

```java
public class MetricsCalculator
{
 public double xProjection(Point p1, Point p2) {
 return (p2.x – p1.x) * 1.5;
 }
 …
}
```

null을 인수로 전달하면 어떻게 될까?

```java
calculator.xProjection(null, new Point(12, 13));
```

당연히 NullPointException을 얻는다.

```java
public class MetricsCalculator
{ 
public double xProjection(Point p1, Point p2) {
 if (p1 == null || p2 == null) {
 throw InvalidArgumentException(
 "Invalid argument for MetricsCalculator.xProjection");
 }
 return (p2.x – p1.x) * 1.5;
 }
}
```

```java
public class MetricsCalculator
{
 public double xProjection(Point p1, Point p2) {
 assert p1 != null : "p1 should not be null";
 assert p2 != null : "p2 should not be null";
 return (p2.x – p1.x) * 1.5;
 }
}
```

이건 좋은 문서화지만, 문제를 해결하진 못한다. 만약 누가 null을 넘기면 우리는 여전히 런타임에러가 뜰 것 가이다.

대부분의 프로그래밍 언어에서 호출자가 사고로 null을 전달했을때 해결할 좋은 방법은 없다. 왜냐하면 이런 경우에 이성적인 접근은 pull을 넘기는 것을 기본적으로 막는 것이기 때문이다. 이렇게 하면 너는 null에 문제가 있다는 것을 표시하고 부주의한 실수가 줄어들 수 있다.

### Conclusion

클린 코드는 가독성이 좋아야 하지만 동시에 견고할 필요가 있다. 이건 충돌하는 목표가 아니다. 우리는 견고한 클린코드를 작성할 수 있다. 만약 에러 핸들링을 별도로 고려할 수 있는 만큼, 오류 처리에 대해 독립적인 로직으로 생각할 수 있다. 우리가 그게 가능해지면 우리는 그것을 논리적으로 다룰 수 있으며 코드의 유지 보수성을 향상 시킬 수 있다.