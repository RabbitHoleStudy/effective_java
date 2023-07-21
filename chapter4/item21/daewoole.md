# 아이템 21 - 인터페이스는 구현하는 쪽을 생각해 설계하라

작성자: 대욱 이
챕터: 4장 - 클래스와 인터페이스
최종 편집: 2023년 7월 21일 오후 8:32
생성 시각: 2023년 7월 21일 오후 7:38

```yaml
utterances: 
  repo: TightJava/effective_java
```

## 개요

---

- 자바8 이후부터 사용가능하게 된 디폴트 메서드를 이용하면 인터페이스의 확장성을 높일 수 있지만, 기존에 존재하는 구현체들과 연동되지 않는 위험이 항상 도사리고 있다. 이에 대해 알아보자!

## 내용

---

## 예제코드 (문제가 없는 일반적인 상황)

---

```java
public interface Developer {

	void programming();

	void getSalary(Integer salary);

	default void workOvertime() {
		System.out.println("What the fuck");
	}
}
```

```java
public class DeveloperImpl implements Developer{

	@Override
	public void programming() {
		System.out.println("programming");
	}

	@Override
	public void getSalary(Integer salary) {
		System.out.println("salary" + salary);
	}
}
```

```java
public interface Developer {

	void programming();

	void getSalary(Integer salary);

	// default method 추가
	default void workOvertime() {
		System.out.println("What the fuck");
	}
}
```

- 기존의 Developer interface에 새로운 default method를 추가해도 기존의 구현체(DeveloperImpl)에는 영향이 없다.

## 예제코드 (문제 발생 상황)

---

- 아파치 버전의 SynchronizedCollection은 모든 메서드에서 클라이언트가 제공한 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼클래스이다. 즉, 모든 메서드 호출을 알아서 동기화해준다.

```java
// 자바 8의 Collection interface에 추가된 디폴트 메서드
default boolean removeIf(Predicate<? super E> filter) {
	Objects.requireNonNull(filter);
	boolean result = false;
	for (Iterator<E> it = iterator(); it.hasNext(); ) {
		if (filter.test(it.next())) {
			it.remove();
			result = true;
		}
	}
	return result;
}
```

- 하지만, 새로 추가된 removeIf 메서드를 아파치 버전의 SynchronizedCollection에서는 재정의하지 않고 있기 때문에 SynchronizedCollection 인스턴스를 여러 스레드가 공유하는 환경에서 한 스레드가 removeIf를 호출하면 예기치 못한 결과가 발생할 수 있다.

## 내용

---

- 디폴트 메서드는 인터페이스의 메서드를 제거하거나 기존 메서드의 시그니처를 수정하는 용도로 쓰이면 안된다. → 기존 클라이언트(기존 구현체)를 망가뜨리게 됨
- 디폴트 메서드는 컴파일에 성공할 수는 있지만, 이후에 기존의 구현체에서 런타임 오류를 일으킬 수 있다.

## 결론

---

- 항상 API를 사용하는 클라이언트 입장에서 인터페이스를 설계해야 한다.