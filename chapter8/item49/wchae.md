# 아이템 49 - 매개변수가 유효한지 검사하라

# 대부분의 매개변수 검사가 필요한시점

- -1 < index
- 객체 참조 ≠ null

⇒ 가능한 에러들을 문서화

⇒ 메서드 몸체가 시작되기 전에 검사 (미리 예외로 튕겨낼것)

---

# @Throws 문서화

- public 과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화 해라
    - @throws 자바독 태그 사용
    - 일반적으로 처리해야하는 애들
        - IllegalArgumentException
        - IndexOutOfBoundsException
        - NullPointerException
        
        ⇒ 제약을 문서화한다면 제약을 어겼을 때 발생하는 예외도 기술
        

```java
/**
     * Returns a BigInteger whose value is {@code (this mod m}).  This method
     * differs from {@code remainder} in that it always returns a
     * <i>non-negative</i> BigInteger.
     *
     * @param  m the modulus.
     * @return {@code this mod m}
     * @throws ArithmeticException {@code m} &le; 0
     * @see    #remainder
     */
    public BigInteger mod(BigInteger m) {
        if (m.signum <= 0)
            throw new ArithmeticException("BigInteger: modulus not positive");

        BigInteger result = this.remainder(m);
        return (result.signum >= 0 ? result : result.add(m));
    }
```

- m.signum ⇒ m 이 null 일때 NPE 를 던진다.
그런데 왜? @throws 안함?
    - 장문
        
        ```java
        /**
         * Immutable arbitrary-precision integers.  All operations behave as if
         * BigIntegers were represented in two's-complement notation (like Java's
         * primitive integer types).  BigInteger provides analogues to all of Java's
         * primitive integer operators, and all relevant methods from java.lang.Math.
         * Additionally, BigInteger provides operations for modular arithmetic, GCD
         * calculation, primality testing, prime generation, bit manipulation,
         * and a few other miscellaneous operations.
         *
         * <p>Semantics of arithmetic operations exactly mimic those of Java's integer
         * arithmetic operators, as defined in <i>The Java Language Specification</i>.
         * For example, division by zero throws an {@code ArithmeticException}, and
         * division of a negative by a positive yields a negative (or zero) remainder.
         *
         * <p>Semantics of shift operations extend those of Java's shift operators
         * to allow for negative shift distances.  A right-shift with a negative
         * shift distance results in a left shift, and vice-versa.  The unsigned
         * right shift operator ({@code >>>}) is omitted since this operation
         * only makes sense for a fixed sized word and not for a
         * representation conceptually having an infinite number of leading
         * virtual sign bits.
         *
         * <p>Semantics of bitwise logical operations exactly mimic those of Java's
         * bitwise integer operators.  The binary operators ({@code and},
         * {@code or}, {@code xor}) implicitly perform sign extension on the shorter
         * of the two operands prior to performing the operation.
         *
         * <p>Comparison operations perform signed integer comparisons, analogous to
         * those performed by Java's relational and equality operators.
         *
         * <p>Modular arithmetic operations are provided to compute residues, perform
         * exponentiation, and compute multiplicative inverses.  These methods always
         * return a non-negative result, between {@code 0} and {@code (modulus - 1)},
         * inclusive.
         *
         * <p>Bit operations operate on a single bit of the two's-complement
         * representation of their operand.  If necessary, the operand is sign-extended
         * so that it contains the designated bit.  None of the single-bit
         * operations can produce a BigInteger with a different sign from the
         * BigInteger being operated on, as they affect only a single bit, and the
         * arbitrarily large abstraction provided by this class ensures that conceptually
         * there are infinitely many "virtual sign bits" preceding each BigInteger.
         *
         * <p>For the sake of brevity and clarity, pseudo-code is used throughout the
         * descriptions of BigInteger methods.  The pseudo-code expression
         * {@code (i + j)} is shorthand for "a BigInteger whose value is
         * that of the BigInteger {@code i} plus that of the BigInteger {@code j}."
         * The pseudo-code expression {@code (i == j)} is shorthand for
         * "{@code true} if and only if the BigInteger {@code i} represents the same
         * value as the BigInteger {@code j}."  Other pseudo-code expressions are
         * interpreted similarly.
         *
         * <p>All methods and constructors in this class throw
         * {@code NullPointerException} when passed
         * a null object reference for any input parameter.
         *
         * BigInteger must support values in the range
         * -2<sup>{@code Integer.MAX_VALUE}</sup> (exclusive) to
         * +2<sup>{@code Integer.MAX_VALUE}</sup> (exclusive)
         * and may support values outside of that range.
         *
         * An {@code ArithmeticException} is thrown when a BigInteger
         * constructor or method would generate a value outside of the
         * supported range.
         *
         * The range of probable prime values is limited and may be less than
         * the full supported positive range of {@code BigInteger}.
         * The range must be at least 1 to 2<sup>500000000</sup>.
         *
         * @implNote
         * In the reference implementation, BigInteger constructors and
         * operations throw {@code ArithmeticException} when the result is out
         * of the supported range of
         * -2<sup>{@code Integer.MAX_VALUE}</sup> (exclusive) to
         * +2<sup>{@code Integer.MAX_VALUE}</sup> (exclusive).
         *
         * @see     BigDecimal
         * @jls     4.2.2 Integer Operations
         * @author  Josh Bloch
         * @author  Michael McCloskey
         * @author  Alan Eliasen
         * @author  Timothy Buktu
         * @since 1.1
         */
        ```
        
    
    ```java
    /* 
    ... 생략 ...
    * <p>All methods and constructors in this class throw
     * {@code NullPointerException} when passed
     * a null object reference for any input parameter.
    ... 생략 ...
    */
    public class BigInteger extends Number implements Comparable<BigInteger>
    ```
    
    ⇒ 짜잔 클래스단에서 써놨습니다.
    

---

# Objects.requireNonNull( )

- null 검사를 수동으로 하지 않아도 된다.
- 원하는 예외 메세지도 작성 가능
- 어디서는 null 검사 메서드로 사용 가능
- 함수 원형
    - `Objects.requrieNonNull(Object obj, String message)`

```java
public class Parameters {

	static class User {
		String name;
		User(){};
		User(String name){
			this.name = name;
		}
	}
	static void validator(User user){
		Objects.requireNonNull(user, "정신안차려?");
		Objects.requireNonNull(user.name, "이름이 없어? 말이 되냐?");
	}

	public static void main(String[] args) {
		User user = new User("김두한");
		try {
		validator(user);
		validator(null);
		}catch (NullPointerException e){
			System.out.println("e = " + e);
		}
		try {
		validator(new User());

		}catch (NullPointerException e){
			System.out.println("e = " + e);
		}
	}
}

e = java.lang.NullPointerException: 정신안차려? 
e = java.lang.NullPointerException: 이름이 없어? 말이 되냐?
```

---

# Private 메서드라면?

- 패키지 제작자인 여러분이 메서드가 호출되는 상황을 통제할 수 있다.
오직 유효한 값만이 메서드에 넘겨지리라는 것을 보증하고 그렇게 해야한다.

## assert

- assert 로 단언한 조건이 무조건 참이라고 선언한다.

```java
public class PrivateMethodValidate {

	private static void sort(long a[], int offset, int length) {
		assert a != null;
		assert offset >= 0 && offset <= a.length;
		assert length >= 0 && length <= a.length - offset;
	}

	public static void main(String[] args) {
		sort(null, 0, 0);
		sort(null, 10, 10);
	}

}
```

- 일반적인 유효성 검사와 다르다
    - 실패시 AssertionError 를 던진다.
    - 런타임에 아무런 효과도, 아무런 성능 저하도 없다 
    ( java 실행시 -ea, —enableassertions 플래그시 런타임 영향)

- 아무일도 일어나지 않는다. ⇒ 명세로 작성한다는 뜻인듯?
우린 Junit 테스트 코드를 작성하니 괜찮지 않을까??

---

# 나중에 쓰기 위해 저장하는 매개변수는 더 신경써라

```java
public static List<Integer> createListViewNoValidate(int[] array) {
		List<Integer> result = new ArrayList<>();
		for (int i : array) {
			result.add(i);
		}
		return result;
	}
```

⇒ array 가 null 일경우 
`for (int i : array)` 여기서 에러가 나는데, 만약 array 를 바로 사용하는게 아니었다면 디버깅이 꽤 어려웠을 것이다.

```java
public static List<Integer> createListView(int[] array) {
		Objects.requireNonNull(array, "array is NULL");
		List<Integer> result = new ArrayList<>();
		for (int i : array) {
			result.add(i);
		}
		return result;
	}
```

⇒ 메소드에 진입하자마자 알게된다.

---

# (예외 )메서드 몸체 실행 이후에 매개변수 유효성을 검사하는 경우

- 유효성 검사 비용이 지나치게 높을 때
- 실용적이지 않을 때
- 계산 과정에서 암묵적으로 검사가 수행될 때

## 암묵적 검사 예시

```java
public static void main(String[] args) {
		List list = new ArrayList<>(); // 테스트를 위해 RAW 타입 사용
		list.add(10);
		list.add(1);
		list.add("S"); // 잘못된 타입
		list.add(8);
		list.add(3);

		for (Object o : list) {
			if (!Integer.TYPE.isInstance(o))
				throw new ClassCastException("Not int type");
		}
		Collections.sort(list);
		list.stream().forEach(System.out::println);
	}
```

⇒ Collections.sort 안에서 ClassCastException 을 던진다

⇒ 따라서 쓸때없이 검사 로직으로 낭비할 필요가 없다.

![16d3fe6c055a53d0](https://github.com/TightJava/effective_java/assets/13278955/99cf1036-1813-4201-a8ba-6fa954198831)

- 또 너무 암묵적 검사가 수행될 것을 너무 믿지는 마라
- 실패 원자성을 해칠 수 있다.

# 요약

- 매개변수는 최대한 범용적으로 설계 해라
- 하지만, 유효성 검사는 꼼꼼히 잘 해라
- 제약들을 문서화해라 → @Throws
- 명시적으로 검사하되 
메소드안에서 검사하는 암묵적 검사가 있을때는 
필요없이 검사하지마라
    
    ![download](https://github.com/TightJava/effective_java/assets/13278955/14eb6543-8f82-48b4-b33b-3f27d008eb80)


---

