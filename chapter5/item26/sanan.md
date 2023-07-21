## 핵심어

- 제네릭(Generic)
    
    **클래스나 메소드에서 사용할 내부 데이터 타입을 컴파일 시에 미리 지정하는 방법**을 말한다.
    제네릭은 **데이터 타입을 파라미터화하여 코드의 재사용성을 높이고, 컴파일 시점에 타입 안정성을 확보**한다.
    이를 통해서 **타입 체크를 컴파일 시점에 수행함으로써 잘못된 타입 사용에 대한 문제를 미리 방지(List<String>에 Integer X)**할 수 있다.
    
    리턴 타입이나 메소드의 파라미터 타입 등을 더 쉽게 예측할 수 있다. → 코드의 가독성을 높이고, 잘못된 타입 사용으로 인한 런타임 에러를 방지하는데 도움이 된다.
    
- 제네릭 국룰 표기
    - Type은 T(Type)로 쓴다.
    → GenericClass<T>
    - 배열, 컬렉션의 원소는 E(Element)로 쓴다.
    → List<E>
    - 맵의 Key, Value는 K(Key), V(Value)로 쓴다.
    → Map<K, V>
    - S, U, V(알파벳 순)등은 세번째, 네번째 등등으로 사용한다.
    → <T, U, V>
- 제네릭 클래스, 인터페이스
    
    선언에 타입 매개변수가 쓰이면 제네릭 클래스, 인터페이스라고 한다.
    
    - 예제 코드
        
        ```java
        // 무엇이든 담는 박스 제네릭 클래스
        public class Box<T> {
           private T t;
        
           public void set(T t) { this.t = t; }
           public T get() { return t; }
        }
        ```
        
        ```java
        public interface GenericInterface<T> {
            void set(T t);
            T get();
        }
        
        public class GenericClass<T> implements GenericInterface<T> {
            private T t;
        
            @Override
            public void set(T t) { this.t = t; }
        
            @Override
            public T get() { return t; }
        }
        ```
        
    
    ![스크린샷 2023-07-17 오후 8.05.11.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ecb9490a-cfa6-449c-bcaa-721397c7ab24/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-07-17_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.05.11.png)
    

---

## 요약

- Raw 타입(’List’와 같은)은 사용하지 마라!
→ List<E>와 E에 대응하는 실제 타입 매개변수를 명시해라!
- Set<Object>는 어떤 타입의 객체도 저장할 수 있는 매개변수화 타입이고, Set<?>는 모종의 타입 객체만 저장할 수 있는 와일드카드 타입이다.
- Set<0bject>와 Set<?>는 안전하지만, Raw 타입인 set은 안전하지 않다.
- 런타임에는 제네릭 타입 정보가 지워지므로 instanceof 연산자는 비한정적 와일드카드 타입(List<?>) 이외의 매개변수화 타입(List<String>)에는 사용할 수 없다.

---

## 내 생각

- 제네릭 타입(List<E>)말고 비한정적 와일드카드 타입(List<?>)는 언제 쓰는 걸까?
    
    List<?>은 항목 타입을 모르거나 좁힐 수 없는 경우에 사용한다.
    ?는 제네릭타입을 가리키는 와일드카드로 unknown을 의미하며, 이것이 의미하는 바는 "모든 종류의 타입"이다.
    이 말은, List<?>는 어떤 객체 타입이든 들어올 수 있으나 타입 안정성을 위해 가변 작업(추가 등)은 허용되지 않는다.
    
    와일드카드 타입을 사용하면, 제네릭 클래스나 메소드가 미리 정의된 특정 타입이 아닌 다양한 타입을 처리할 수 있게 된다.
    
    예를 들어, 제네릭 메소드를 작성하고자 하고 이 메소드가 List<String>, List<Integer>, List<뭐시깽이>.. 등의 다양한 타입의 List를 파라미터로 받을 수 있게 만들고 싶다면, 와일드카드를 사용하면 된다.
    
    ```java
    public static void printList(List<?> list) {
        for (Object item : list) {
            System.out.println(item);
        }
    }
    
    List<Integer> integerList = Arrays.asList(1, 2, 3);
    List<String> stringList = Arrays.asList("one", "two", "three");
    
    printList(integerList); // Valid
    printList(stringList); // Valid
    ```
    
    Raw 타입인 경우에 타입 미지정으로 안전하지 않고, 가변적으로 사용하는 경우에는 <E>와 같이 제네릭 타입을 지정해주거나, 어떤 타입이 들어올지는 모르겠으나, readonly로만 수행하려할 때(어차피 몰라서 아무것도 못 함)에는 <?>와 같이 사용하면 된다는 얘기로 이해했다.
    
- 한정적 와일드카드 타입(Wildcard Type Bound)는 언제 쓸까?
    
    한정적 와일드카드 타입은 제네릭 타입이 특정 범위 내의 타입이라는 것을 표현하기 위해 사용된다.
    이때 '한정'이라는 용어는 제네릭이 받아들이는 타입의 범위를 제한(limitation)하는 것을 의미한다.
    예를 들어 List<? extends Number>는 숫자 타입의 서브 클래스를 원소로 가지는 리스트를 의미하며, List<? super Integer>는 Integer의 상위 클래스를 원소로 가지는 리스트를 의미한다.
    
    ```java
    List<? extends Number> numbers = new ArrayList<Integer>();
    numbers = new ArrayList<Float>();
    numbers = new ArrayList<Double>();
    
    List<? super Integer> superNumbers = new ArrayList<Number>();
    ```
    

---

## 예제 코드

- Raw 타입으로 제네릭 클래스를 사용해서 문제 일으키기

```java
public static void printList(List<String> list) {
		for (String elem : list)
			System.out.println(elem + " ");
		System.out.println();
	}
/*------------------------------------------------*/
	@Test
	void Raw_제네릭() {

		List rawList = new ArrayList();
		rawList.add("Raw");
		rawList.add(1);

		Assertions.assertThatThrownBy(() -> {
			printList(rawList);
		}).isInstanceOf(ClassCastException.class);
	}
```

- 제네릭 클래스와 제네릭 인터페이스를 이용한 구현

```java
public class Box<T> {
	private final T something;

	public Box(T something) {
		this.something = something;
	}

	public T unbox() {
		return something;
	}
}
/*------------------------------------------------*/
public interface Checker<T> {
	default boolean isEmpty(Box<?> box) {
		return box.unbox() == null;
	}

default boolean isCoincident(Box<?> box, Class<T> clazz) {
		return clazz.isInstance(box.unbox());
	}

	boolean isLarge(T item);
}
/*------------------------------------------------*/
public class IntegerChecker implements Checker<Integer> {
	@Override
	public boolean isLarge(Integer item) {
		return item > 100;
	}
}
/*------------------------------------------------*/
@Test
	void 제네릭() {
		Box<Integer> integerBox = new Box<>(1);
		Box<String> stringBox = new Box<>("hi");
		Checker<Integer> checker = new IntegerChecker();

		assertThat(checker.isEmpty(integerBox)).isFalse();
		assertThat(checker.isCoincident(integerBox, Integer.class)).isTrue();
		assertThat(checker.isCoincident(stringBox, Integer.class)).isFalse();
		assertThat(checker.isLarge(integerBox.unbox())).isFalse();
	}
```
