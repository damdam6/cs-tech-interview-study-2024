# Template Method Pattern

> 로직을 단계 별로 나눠야 하는 상황에서 적용한다. 
> 단계별로 나눈 로직들이 앞으로 수정될 가느성이 있을 경우 더 효율적이다. 

- 여러 클래스에서 공통으로 사용하는 메서드를 템플릿화 하여 상위 클래스에서 정의하고, 하위 클래스마다 세부 동작 사항을 다르게 구현하는 패턴
- 즉, 변하지 않는 기능(템플릿)은 상위 클래스에 만들어두고 자주 변경되며 확장할 기능은 하위 클래스에서 만들도록 하여, 상위의 메서드 실행 동작 순서는 고정하면서 세부 실행 내용은 다양화 될 수 있는 경우에 사용된다. 
### 💡 조건
- 클래스는 추상(abstract) 으로 만든다. 
- 단계를 진행하는 메소드는 수정이 불가능하도록 `final` 키워드를 추가한다. 
- 각 단계들은 외부는 막고, 자식들만 활용할 수 있도록 `protected` 로 선언한다. 

### 💡 예시
피자를 만들 때는 크게 **반죽 → 토핑 → 굽기** 3단계로 이루어져있다. 

이 단계는 항상 유지되며, 순서가 바뀔 일은 없다. 물론 실제로는 도우에 따라 반죽이 달라질 수 있겠지만, 일단 모든 피자의 반죽과 굽기는 동일하다고 가정하자. 그러면 피자 종류에 따라 토핑만 바꾸면 된다. 

```java
abstract class Pizza {
	
	protected void 반죽() { System.out.println("반죽!"); }
	abstract void 토핑() {}
	protected void 굽기() { System.out.println("굽기!"); }
	
	final void makePizza() { // 상속 받은 클래스에서 수정 불가 
		this.반죽();
		this.토핑();
		this.굽기();
	}
}
```

```java
class PotatoPizza extends Pizza {

	@Override
	void 토핑() {
		System.out.println("감자 넣기 !");
	}
}

class TomatoPizza extends Pizza {
	
	@Override
	void 토핑() {
		System.out.println("토마토 넣기 !");
	}
}
```
- abstract 키워드를 통해 자식 클래스에서는 선택적으로 메서드를 오버라이드 할 수 있게 된다. 

---
[Ref1](https://velog.io/@shasha/%ED%97%A4%EB%93%9C%ED%8D%BC%EC%8A%A4%ED%8A%B8-%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4-Chapter-08.-%ED%85%9C%ED%94%8C%EB%A6%BF-%EB%A9%94%EC%86%8C%EB%93%9C-%ED%8C%A8%ED%84%B4)
[Ref2](https://gyoogle.dev/blog/design-pattern/Template%20Method%20Pattern.html)