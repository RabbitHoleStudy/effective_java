## 핵심어

- 공변(**Co**variance), 반공변(**Contra**variance), 불공변(**In**variance), 무공변(**Non**-variance)
    - 공변(**Co**variance)
        
        공변성은 **부모 클래스의 변환 방향과 자식 클래스의 변환 방향이 같을 때** 적용되는 용어다.
        예를 들면, 만약 A가 B의 하위 타입이면, Producer<? extends A>도 Producer<? extends B>의 하위 타입이라는 것이다.
        
        ```java
        // A = Apple, B = Fruit
        List<? extends Fruit> fruits = new ArrayList<Apple>(); // 공변
        fruits.add(new Apple());  // 컴파일 에러 - Fruit의 하위클래스 뭐든 들어갈 수 있으므로 write하기 어려움
        Fruit fruit = fruits.get(0);  // 가능
        ```
        
    - 반공변(**Contra**variance)
        
        반공변성은 **부모 클래스의 변환 방향과 자식 클래스의 변환 방향이 반대일 때** 적용되는 용어다.
        예를 들어, A가 B의 하위타입일 때, Consumer<? super B>는 Consumer<? super A>의 하위타입이다.
        
        ```java
        // A = Apple, B = Fruit
        List<? super Apple> apples = new ArrayList<Fruit>(); // 반공변
        apples.add(new Apple());  // 가능 - Fruit은 Apple의 super이니까.
        Apple apple = apples.get(0);  // 컴파일 에러 - Apple의 super에 뭐가 더 있을지 모르므로 read하기 어려움.
        ```
        
    - 불공변(**In**variance)
        
        불공변성은 부모 클래스와 자식 클래스 간에 아무런 관계가 없을 때 적용되는 용어다.
        예를 들어, List<Apple>과 List<Fruit> 사이에는 아무런 관계가 없다.
        
        ```java
        List<Apple> apples = new ArrayList<Apple>();
        List<Fruit> fruits = apples; // 컴파일 에러! 불공변
        ```
        
    - 무공변(**Non**-variance)
        
        무공변이란 공변도 반공변도 아닌 경우를 말한다.
        자바에서는 명시적으로 공변이나 반공변을 지정하지 않은 제네릭 클래스나 인터페이스는 무공변이다. 예를 들어, List<T>는 무공변이다.
        
    
    정리: ‘변성’과 관련한 개념이 나온 이유는 제네릭과 관련한 클래스들 간의 상속에 대해서 유연성을 주기 위함이다. 이 변성을 가능케 하는 것이 제네릭의 와일드 카드 타입이다 - 일반 제네릭 타입은 무공변이다.
    
    <img width="764" alt="스크린샷 2023-07-21 오후 9 59 44" src="https://github.com/TightJava/effective_java/assets/105692206/d53df0be-e37c-4fde-b1e7-f2a215c75515">

    
    https://velog.io/@lsb156/covariance-contravariance
    
- Target Typing(목표 타이핑)
    
    JVM에서 사용되는 용어 중 하나이다.
    이는 컴파일러가 표현식의 유형을 그 표현식이 나타나는 위치에 따라 결정하는 기능을 지칭한다.
    Java 8부터 람다와 메소드 레퍼런스와 같은 새로운 검사형 표현식이 추가되면서 이 용어가 자주 사용되는데, 이러한 표현식들은 그 자체로는 명확한 "타입"이 없지만, 그것들이 사용되는 컨텍스트에 따라 "타겟 타입"이 결정된다.
    예를 들어, 다음과 같은 람다 표현식을 살펴보자.
    
    `() → “Hello World!”`
    
    위의 람다는 Supplier<String>, Callable<String> 또는 다른 모든 함수적 인터페이스의 컨텍스트에서 사용할 수 있다.
    따라서, 컴파일러는 함수적 인터페이스의 시그니처를 기준으로 람다의 타입을 결정한다.
    

---

## 요약

- 유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라.
- PECS(Producer - extends, Consumer - super)
- 사용자가 와일드카드 타입을 신경 써야 한다면 그 API에 문제가 있을 가능성이 크다.
- (Comparable과 같은 인터페이스를) 직접 구현하지 않고 다른 타입을 확장한 타입을 지원하기 위해 와일드카드가 필요하다.
    
    <img width="497" alt="스크린샷 2023-07-20 오전 7 58 26" src="https://github.com/TightJava/effective_java/assets/105692206/628b48f9-f2f4-403d-b2df-ffda92cdf89c">

    
    ScheduledFuture는 Delayed에 대한 Extends로, Delayed의 Comparable을 상속한다. 이 때, 제네릭 메서드가 ‘Comparable<E>’를 사용하게 되면, ScheduledFuture의 Comparable<ScheduledFuture>의 구현을 기대하게되는데, 이에 대한 구현은 없기 때문에(상위 클래스의 구현을 상속하기 때문에) 확장성이 적어질 수 있다.
    
- 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라.

---

## 내 생각

- Raw 타입(’List’), 제네릭 타입(’List<E>’), 와일드 카드 타입(’List<?>’)
    
    Raw 타입으로 지정하면 무엇이든 담는(마치 파이썬의 것과 같이) 것이 된다. Java는 결국 강타입 언어이므로 이는 의도치 않은 동작으로 이어진다.
    
    한편 제네릭 타입으로 지정하면 컴파일 타임시에 타입을 지정하되, 여러 타입에 대한 동일한 작업을 수행할 수 있고, 원하는 제약사항을 걸어놓을 수 있다(’T extends Comparable<T>’ 등).
    
    와일드 카드 타입의 경우에는 컴파일 타임시에 타입을 추론할 수 없으므로 수정, 변경 따위의 작업들을 수행할 수는 없다. 이를 사용하는 가장 큰 이유는 Raw 타입이 갖는 위험성은 줄이고 제네릭을 유연하고 안전하게 사용하기 위함이다. 즉, Read-Only의 경우에는 와일드카드를 사용하여 그 범위를 명확하게 제한할 수 있고, 수정 가능해야하는 경우에는 제네릭 타입을 사용하여 유연하게 작업을 수행할 수 있는 것이다.
    

---

## 예제 코드

- SPEC 구현하기

```java
public class Fruit {
}

public class Apple extends Fruit {
	@Override
	public String toString() {
		return "Apple";
	}
}

public class WildCardCollection {
	static <T> void copyToCollection(Collection<? extends T> src, Collection<? super T> dest) {
		for (T t : src) {
			dest.add(t);
		}
	}

	static void printCollection(Collection<? super Fruit> collection) {
		for (Object obj : collection) {
			System.out.println(obj);
		}
	}
}

@Test
	void SPEC() {
		List<Apple> apples = new ArrayList<>();
		apples.add(new Apple());

		List<Fruit> fruits = new ArrayList<>();

		WildCardCollection.copyToCollection(apples, fruits);

		WildCardCollection.printCollection(fruits);
//		WildCardCollection.printCollection(apples); 하위 타입이므로 컴파일 에러 
	}
```
