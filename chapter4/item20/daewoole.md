# 아이템 20 - 추상 클래스보다는 인터페이스를 우선하라

작성자: 대욱 이
챕터: 4장 - 클래스와 인터페이스
최종 편집: 2023년 7월 21일 오전 2:39
생성 시각: 2023년 7월 20일 오후 11:04

```yaml
utterances: 
  repo: TightJava/effective_java
```

## 개요

---

- 인터페이스와 추상 클래스 모두 자바가 제공하는 다중 구현 메커니즘이다. 각 메커니즘이 가진 차이점과 장단점에도 불구하고 왜 인터페이스를 우선해야하는지 알아보자

## 개념 및 용어

---

### mix-in interface

- 믹스인을 구현한 클래스에 원래의 “주된 타입” 외의 선택적 행위(기능)을 제공하는 인터페이스
    - ex. Comparable 인터페이스를 구현한 클래스의 인스턴스들은 서로의 순서를 정할 수 있다.

### skeletal implementation

- 필요한 구현을 인터페이스의 디폴트 메서드 혹은 추상 클래스로 미리 구현한 것

## 내용

---

### 추상 클래스

- 추상 클래스를 구현한 클래스는 반드시 추상 클래스의 하위 클래스가 된다.
- 기존 클래스위에 새로운 추상 클래스를 끼워넣기 어렵다.
- 현실에는 계층을 구분하기 어려운 개념도 존재한다.

### 인터페이스

- 인터페이스의 명세대로 잘 구현하기만 한다면, 다른 클래스를 상속했더라도 같은 타입(해당 인터페이스의 구현 클래스)으로 취급된다.
- 기존 클래스에 새로운 인터페이스를 구현하기 용이하다.
- 계층구조가 없는 타입 프레임워크를 만들 수 있다.
- 믹스인 정의에 알맞다.

```java
public interface Developer {

	void work();

	void getSalary(Integer salary);

	default void workOvertime() {
		System.out.println("What the fuck");
	}
}
```

- default method 사용 가능

```java
public class DeveloperImpl implements Developer {
	
	public void work() {
		System.out.println("Programming");
	}

	public void getSalary(Integer salary) {
		System.out.println("I got " + salary);
	}
}
```

- DeveloperImpl 구현

```java
public interface Marketing {

	void makePPT();
}
```

```java
public class DeveloperImpl implements Developer, Marketing {

	@Override
	public void programming() {
		System.out.println("Programming");
	}

	public void getSalary(Integer salary) {
		System.out.println("I got " + salary);
	}
	
	public void makePPT() {
		System.out.println("makePPT");
	}
}
```

- 기존의 클래스인 DeveloperInpl에 새로운 mix-in interface인 Marketing을 (추가) 구현

```java
// Developer interface에 대한 골격 구현 추상 클래스

public abstract class AbstractDeveloper implements Developer {
	
	public void programming() {
		System.out.println("Implement programming method in skeletal implementation class");
	}
	
	public void getSalary(Integer salary) {
		System.out.println("salary in skeletal implementation class" + salary);
	}
	
}
```

```java
public class DeveloperImpl implements Developer {

// 다음의 inner class는 골격 구현 클래스를 상속하고, 구현 클래스 DeveloperImpl로부터 인터페이       스의 구현을 위임받는다.
	private static class AbstractDeveloperDelegator extends AbstractDeveloper {

		@Override
		public void programming() {
			System.out.println("DeveloperImpl call programming method in AbstractDeveloper by using delegation pattern");
		}
	}

	AbstractDeveloperDelegator abstractDeveloperDelegator = new AbstractDeveloperDelegator();

	@Override
	public void programming() {
		abstractDeveloperDelegator.programming();
	}

	@Override
	public void getSalary(Integer salary) {
		abstractDeveloperDelegator.getSalary(salary);
	}
}
```

- 위와 같은 방식으로, Developer의 구현 클래스인 DeveloperImpl은 단일 상속의 문제로부터 벗어날 수 있으며, 여러 인터페이스와 추상클래스로부터 기능을 물려받을 수 있는 확장성을 가진다.

## 결론

---

- 추상 클래스로 할 수 있는 대부분의 일은 인터페이스로 할 수 있고, 디폴트 메서드와 골격 구현 클래스 제공 등의 방식으로 확장성을 높이고, 다중 상속의 장점을 제공할 수 있는 인터페이스를 사용하자!

## 질문

---

- 골격 구현의 범위는 어디까지?

## 참고자료

[JAVA 상속 문제 해결을 위한 디자인 패턴: composition and skeletal implementation](https://velog.io/@kms8571/%ED%8F%AC%EC%8A%A4%ED%8C%85-%EC%8A%A4%ED%84%B0%EB%94%94-JAVA%EC%9D%98-%EC%83%81%EC%86%8D-%EB%AC%B8%EC%A0%9C-%ED%95%B4%EA%B2%B0%EC%9D%84-%EC%9C%84%ED%95%9C-%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4-composition-and-skeletal-implementation)