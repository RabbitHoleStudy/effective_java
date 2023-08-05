### 요약

- API를 설계할 때 람다를 염두에 두고 설계하자.
- 필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하자.
- 직접 구현해야 하는 경우, `@FunctionalInterface` 애노테이션을 붙여주고 메서드 다중정의에 주의하자.

---

### 표준 함수형 인터페이스

자바 표준 라이브러리인 `java.util.function` 패키지에는 총 43개의 인터페이스가 담겨있다. **필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라.**

- API가 다루는 개념의 수가 줄어들어 익히기 더 쉬워진다.
- 표준 함수형 인터페이스들은 유용한 디폴트 메서드를 많이 제공하므로 다른 코드와의 상호운용성도 크게 좋아진다.

### 기본 인터페이스

기본 인터페이스 6개만 기억해두면 필요할 때 나머지 인터페이스를 유추해서 찾아쓸 수 있을 것이다.

| 인터페이스 | 함수 시그니처 | 설명 | 예 |
| --- | --- | --- | --- |
| UnaryOperator<T> | T apply(T t) | 반환값과 인수의 타입이 같은 함수. 인자의 개수가 1개. | String::toLowerCase |
| BinaryOperator<T> | T apply(T t1, T t2) | 반환값과 인수의 타입이 같은 함수. 인자의 개수가 2개. | BigInteger::add |
| Predicate<T> | boolean test(T t) | 인수 하나를 받아 boolean을 반환하는 함수 | Collection::isEmpty |
| Function<T, R> | R apply(T t) | 인수와 반환 타입이 다른 함수 | Arrays::asList |
| Supplier<T> | T get() | 인수를 받지 않고 값을 반환하는 함수 | Instant::now |
| Consumer<T> | void accept(T t) | 인수를 하나 받고 반환값은 없는 함수 | System.out::println |
- 기본 인터페이스는 기본 타입인 int, long, double용으로 각 3개씩 변형이 있다.
    
    ex) IntPredicate, LongBinaryOperator
    
- 표준 함수형 인터페이스는 대부분 기본 타입만 지원하는데, 그렇다고 **기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자.** (아이템 61)
    
    계산량이 많을 때는 성능이 매우 느려진다.
    

### 표준 함수형 인터페이스가 아니라 직접 작성하는 것이 좋은 경우

- 표준 인터페이스 중 필요한 용도에 맞는 게 없는 경우
    
    ex) 매개변수 3개를 받는 Predicate
    
- 구조적으로 똑같은 표준 함수형 인터페이스가 있더라도 직접 작성해야만 할 때가 있다. 다음 3가지 조건 중에 하나 이상을 만족한다면, 직접 작성하는 것을 고려해보자.
    1. 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.
    2. 반드시 따라야 하는 규약이 있다.
    3. 유용한 디폴트 메서드를 제공할 수 있다.
    
    ex) Comparator<T> 인터페이스
    
    구조적으로는 ToIntBiFuction<T, U> 인터페이스와 같지만, Comparator라는 이름이 용도를 보다 명확히 설명해준다(조건 1). 또한, `compare()`는 반드시 따라야 하는 규약이 있고(조건 2), Comparator는 `reversed()`, `thenComparing()` 등의 메서드를 제공한다(조건 3).
    

### @FunctionalInterface

전용 함수형 인터페이스를 작성하는 경우 `@FunctionalInterface` 애노테이션을 꼭 붙여주자. 그 이유는 @Override를 사용하는 이유가 비슷하게, 프로그래머의 의도를 명시하기 위함이다.

- 해당 클래스의 코드나 설명 문서를 읽을 사람에게 그 인터페이스가 람다용으로 설계된 것임을 알려줌
- 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해줌
- 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아줌

### 함수형 인터페이스를 API에서 사용할 때 주의사항

서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중 정의해서는 안 된다. 모호함으로 인해 문제가 발생할 수 있다.

ex) ExecutorService의 submit 메서드

```java
// current/ExecutorService: Callable<T>를 받는 것과 Runnable을 받는 것을 다중정의했다.
public interface ExecutorService extends Executor {
		<T> Future<T> submit(Callable<T> task);
		Future<?> submit(Runnable task);
}

// item 52
ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println);  // Reference to 'println' is ambiguous, both 'println()' and 'println(boolean)' match
```
