## 핵심어

- 제네릭 싱글턴 팩터리
    
    제네릭 싱글턴 팩터리는 특정 타입에 대해 단 하나의 인스턴스만 생성하는 디자인 패턴이다.
    이를 활용하면 메모리를 절약하고, 인스턴스가 단일 상태를 공유하기 때문에 동기화에 대한 부담을 줄일 수 있다.
    제네릭을 이용한 싱글턴 팩터리의 핵심은, 타입 인자에 따라 서로 다른 타입의 같은 객체를 봉인(encapsulation)하도록 하는 것이다.
    즉, 호출하는 쪽에서는 실제 비즈니스 로직을 모르더라도 필요한 타입의 객체를 얻을 수 있게끔 한다.
    
    ```java
    public class GenericSingletonFactory {
        
        // 싱글턴 인스턴스
        private static Object instance = new Object(); 
    
        // 제네릭 싱글턴 팩터리 메소드
        public static <T> T getInstance(Class<T> type) {
            return type.cast(instance); 
        }
    }
    /*--------------------------------------------------*/
    Object obj = GenericSingletonFactory.getInstance(Object.class);
    ```
    
    하지만 이 코드는 실제로 잘 사용되지 않는데, 이유는 각 타입에 대한 싱글턴 인스턴스를 반환해야 할 때 해당 인스턴스가 실제로 해당 타입의 인스턴스인지 보장할 수 없기 때문이다.(왜 씀?)
    따라서 이 방식은 그닥 유용하지 않고, 일반적인 싱글턴 패턴의 사용이 더 바람직하다고 한다. 이 팩터리 메소드의 주된 용도는 Collections.emptyList(), Collections.emptySet() 등과 같이 빈 컬렉션 인스턴스를 반환하는 것이다.
    → 결국 싱글턴 팩터리이기 때문에, 인스턴스를 새로 생성하는 것이 아니므로(전달 받고 반환하는 레퍼런스는 동일) 한계가 있을 것 같다.
    

- Identity Function(항등 함수)
    
    항등 함수(identity function)는 입력값을 그대로 출력하는 함수를 말한다.
    수학적으로 보면, 항등함수는 모든 실수 x에 대해 f(x) = x 라는 성질을 가진다.
    Java에서 Function.identity()는 이러한 항등 함수를 반환하는 메소드를 제공한다.
    예를 들어, 아래 코드는 항등 함수를 사용해 Stream의 모든 요소를 그대로 새로운 List에 넣는 것이다:
    
    ```java
    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
    List<Integer> sameList = list.stream()
          .map(Function.identity())
          .collect(Collectors.toList());
    ```
    
    코드에서 Function.identity()는 입력을 그대로 반환하는 항등 함수를 만든다.
    따라서 map을 통해 리스트의 각 요소를 그대로 적용, sameList는 원래 리스트와 동일한 요소를 가지게 된다.
    

---

## 요약

- 형변환이 필요한 메서드의 경우에는 제네릭 메서드로 변경하는 것을 고려하라.

---

## 내 생각

- 결국 제네릭의 요점은 ‘컴파일 시 타입 안전하고, 런타임 시에는 타입 소거’와 ‘타입에 대해 유연하기 때문에 재사용성이 높아진다’인 것 같다.
- 와일드 카드<?>와 제네릭 타입<E>의 가장 큰 차이점
    
    수정 가능성(Mutability)와 컴파일 타임시의 타입 추론에서 차이가 있는 것 같다.
    
    ‘어떤 타입이 들어오든 받아들일 수 있음’은 와일드 카드와 제네릭 타입 둘 다 동일하다 - 어차피 제네릭 자체의 특성으로, 공유되니까.
    
    하지만, 내부적으로 사용될 때 type safe(<E>)냐 아니냐(<?>)에서 갈리고, 이로 인해서 커버할 수 있는 범위가 달라지고, 수정 가능성이 갈린다. <E>와 같은 경우에는 해당 호출 시(코드 레벨에서 컴파일 타임)에 타입이 결정되고, 그 타입에 맞게끔 동작함의 제약을 받는다. <?>와 같은 경우에는 무엇이든 받아들일 수 있다.
    
    한편, <?>의 경우에 메서드 내부에서 특정 작업(수정, 교환)등을 수행할 수는 없다.
    

---

## 예제 코드

- 제네릭 싱글톤 팩터리 이용해보기

```java
public class AllTypeAffordableSingletonSet {
	private static final Set EMPTY_SET = new HashSet<>();

	@SuppressWarnings("unchecked")
	public static <T> Set<T> getInstance() {
		return (Set<T>) EMPTY_SET;
	}
}

@Test
	void 제네릭_싱글톤_팩터리() {
		Set<String> stringSet = AllTypeAffordableSingletonSet.getInstance();
		Set<Integer> integerSet = AllTypeAffordableSingletonSet.getInstance();

		stringSet.add("hello");
		integerSet.add(123);
		System.out.println("stringSet = " + stringSet);
		// stringSet = [hello, 123]
	}
```

- 제네릭 타입<E> 안 쓰고 <?>로 사용하면 벌어지는 일
    
    <img width="541" alt="스크린샷 2023-07-21 오전 9 06 03" src="https://github.com/TightJava/effective_java/assets/105692206/2a3c8445-4614-40f2-920f-25748a04d3cc">

    제네릭 타입으로 메서드에서 지정해주면 이런 케이스를 막아줄 수 있다.
    

```java
public static <E> List<E> mergeLists(List<E>... lists) {
    List<E> result = new ArrayList<>();
    for (List<E> list : lists) {
        result.addAll(list);
    }
    return result;
}

List<Integer> a = Arrays.asList(1, 2, 3);
List<Integer> b = Arrays.asList(4, 5, 6);
List<Integer> merged = mergeLists(a, b); // [1, 2, 3, 4, 5, 6]
/*------------------------------------------------------------*/
public static List<?> mergeLists(List<?>... lists) {
    List<?> result = new ArrayList<>();
    for (List<?> list : lists) {
        result.addAll(list); // 컴파일 에러
    }
    return result;
}
```

<img width="710" alt="스크린샷 2023-07-21 오전 9 10 16" src="https://github.com/TightJava/effective_java/assets/105692206/2e9ebc4e-2bcd-4739-b223-e518b9c87cd8">


와일드카드 타입의 경우에는 수정이 불가능 하므로 컴파일 에러가 나타난다.
