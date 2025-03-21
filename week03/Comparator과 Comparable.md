### Comparable? Comparator?
- 객체를 비교할 수 있도록 만들어주는 "인터페이스"
``` java
public class Persion{
	int name;
	int age;

	/*
		무엇을 기준으로 정렬을 해줄 것인가?
		-> 이러한 문제점을 해결하기 위해 Comparator/Comparable이 사용된다.
	*/
}
```
- 인터페이스이기 때문에 메소드를 구현해주어야 한다.
- 둘의 차이는 무엇인가?
	- Comparable: 비교대상(자기 자신과 매개변수), 포함 패키지(java.jang)
	- Comparator: 비교대상(두 매개변수 객체), 포함 패키지(java.util)

### Comparable
- 자기 자신과 매개변수 객체를 비교 한다.
- [Comparable에 선언된 메소드](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Comparable.html) : int compareTo( T o )

``` java
public class Person implements Comparable<Person>{
	String name;
	int age;

	public Person(){
	}

	//나이를 기준으로 내림 차순 정렬
	//방법 1
	@Override
	public int compareTo(Person p){
		return this.age - p.age
	}
	//방법2
	public int compareTo(Person p){
		if(this.age > p.age) return 1;
		else if(this.age == p.age) return 0;
		else return -1;
	}

}
```

### Comparator
- 도 매개변수 객체를 비교
- [Comparator에 선언된 메소드](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Comparator.html) : 다수 존재, 크기 비교를 위해서는 int compare(T o1, T o2) 구현 필요
``` java
public class Person implements Comparable<Person>{
	String name;
	int age;
	int score

	//나이를 기준으로 내림 차순 정렬
	//방법 1
	@Override
	public int compare(Person p1, Person p2){
		return p1.age - p2.age
	}
	//방법2
	public int compareTo(Person p1, Person p2){
		if(p1.age > p2.age) return 1;
		else if(p1.age == p2.age) return 0;
		else return -1;
	}

}
```
- Comparable의 CompareTo는 선행 원소가 자기 자신이 되고, 후행 원소가 매개변수로 들어오는 p가 되는 반먼에, Comparator의 compare은 선행 원소가 p1이 되고, 후행 원소가 p2가 된다.
- 즉, 자기 자신은 원소 비교에 영향이 없음.

> [!warning] 매개변수의 차로 비교 시 주의 사항
> - 뺄셈 과정에서 오버플로우, ~~언더플로우~~가 발생할 가능성이 있다.
> - 대소비교에 있어 이러한 오버플로우가 발생할 여지가 있는지를 반드시 확인해야한다.

### 사용
- Comparable 인터페이스는 클래스에 기본적으로 implement하여 구현해야함
- Comparable 인터페이스는 익명 객체를 생성해 여러 개의 정렬 로직을 생성할 수 있다는 장점이 있음.
``` java
// 익명 객체로 Comparator 생성
PriorityQueue<Person> pq = new PriorityQueue<>(new Comparator<Person>() {
    @Override
    public int compare(Person p1, Person p2) {
        // 1차 비교: 점수
        int scoreCompare = Integer.compare(p1.score, p2.score);
        if (scoreCompare != 0) return scoreCompare;
        
        // 2차 비교: 나이
        int ageCompare = Integer.compare(p1.age, p2.age);
        if (ageCompare != 0) return ageCompare;
        
        // 3차 비교: 이름(문자열 순)
        return p1.name.compareTo(p2.name);
    }
});

// 람다 식으로 Comparator 생성
Comparator<Person> scoreAgeNameComparator = Comparator
    .comparingInt((Person p) -> p.score)
    .thenComparingInt(p -> p.age)
    .thenComparing(p -> p.name);
PriorityQueue<Person> pq = new PriorityQueue<>(scoreAgeNameComparator);

PriorityQueue<Person> pq = new PriorityQueue<>(
    Comparator.comparingInt((Person p) -> p.score)
              .thenComparingInt(p -> p.age)
              .thenComparing(p -> p.name)
);
```

-  성능 상에는 차이가 거의 없다고 한다!
- Comparable은 클래스의 기본 정렬 기준을 정의할 때, Comparator는 다양한 정렬 기준을 동적으로 적용하고자 할 때 사용하는 것이 적합합니다.