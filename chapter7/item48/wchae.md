
# 아이템48 - 스트림 병렬화는 주의해서 사용하라

# 스트림 병렬화

![Untitled](https://github.com/TightJava/effective_java/assets/13278955/65725d59-5936-4802-997d-4b5ea923287b)

- `Stream.parallel()` 로 스트림 실행을 병렬화 할 수 있다.
- 여러 스레드가 돌면서 병렬처리를 한다.
- 기본적으로는 작업 순서가 보장이 안된다.

---

# 병렬화의 안좋은 예

![download-1](https://github.com/TightJava/effective_java/assets/13278955/ae8ea598-6546-4bf2-8990-56a041f1c7f9)

## Parallel( ) & limit( )

- 책 예제
    
    ```java
    import static java.math.BigInteger.ONE;
    import static java.math.BigInteger.TWO;
    
    import java.math.BigInteger;
    import java.util.stream.Stream;
    
    public class PrimeMaker {
    
    	public static void main(String[] args) {
    		primes()
    //				.parallel()
    				.map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
    				.filter(mersenne -> mersenne.isProbablePrime(50))
    				.limit(20)
    				.forEach(System.out::println);
    	}
    	static Stream<BigInteger> primes() { // 2 이후의 소수를 계속 리턴
    		return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    	}
    
    }
    ```
    
- limit 을 쓸때, 병렬처리인 parallel 을 쓰면 더 느려진다.
    
    ![why](https://github.com/TightJava/effective_java/assets/13278955/727e9b49-55a0-40ed-b26a-9e98a1984f81)

    - limit을 쓸경우 싱글로 계산할때는 20 까지 돌고 딱 끝난다
    - 병렬을 쓸경우, CPU 코어가 남는다면 limit 보다 더 처리한 후 남은 결과값을 버리게 된다.
    - 쿼드코어 라고 가정할때…
        - 20번째 소수점 계산을 수행할때 1번코어가 담당 한다고 가정
            - 2번 코어 21 번째 (연산시간 2배)
            - 3번 코어 22번째 (4배)
            - 4번 코어 23번째 (8배)
- 단순화 예제
    
    ```java
    public class MyTestParallel {
    
    	public static void main(String[] args) {
    		long start =0, end = 0;
    
    		start = System.currentTimeMillis();
    		Stream.iterate(1, MyTestParallel::add)
    				.parallel()
    				.limit(10)
    				.forEach(System.out::println);
    		end = System.currentTimeMillis();
    		System.out.println("after = " + (end - start));
    		System.out.println("====================================");
    
    		start = System.currentTimeMillis();
    		Stream.iterate(1, MyTestParallel::add)
    				.limit(10)
    				.forEach(System.out::println);
    		end = System.currentTimeMillis();
    		System.out.println("after = " + (end - start));
    		
    	}
    
    }
    ```
    
    ```java
    7 2 1 4 5 6 8 9 3 10
    after = 4
    ====================================
    1 2 3 4 5 6 7 8 9 10
    after = 1
    ```
    

---

# 좋은 시너지

- `ArrayList`, `HashMap`, `HashSet`, `ConcurrentHashMap` 의 인스턴스
- `[ ]`배열, `int` 범위, `long` 범위 일때

⇒ 병렬화의 효과가 가장 좋다.

- 나누는 작업은 `Spliterator` 가 담당
- 참조 지역성이 뛰어나다

## 참조지역성

- 이웃한 원소의 참조들이 메모리에 연속에서 저장
- 실제 객체가 메모리에서 서로 떨어져있다면 나빠짐
    
    ⇒ 스레드는 데이터가 주 메모리→캐시 메모리로 전송되어 오기를 기다림
    →레이턴시 발생
    
- 배열이 가장 좋다. 
참조가 아닌 데이터 자체가 메모리에 연속해서 저장된다.

⇒ 배열과 시너지가 가장 좋다. 참조지역성이 좋은애들과 친하다

## 스트림의 종단 연산

- 종단 연산 중 병렬화에 가장 적합한 것은 축소(reduction)다.

### 종단 연산이란?

파이프라인에서 만들어진 모든 원소를 하나로 합치는 작업

- 연산 : `reduce`, `min`, `max`, `count`, `sum`
- 조건부 : `anyMatch`, `allMatch`, `noneMath`

BUT

- 가변 축소 (mutable reduction)을 수행하는 Stream 의 `collect` 메소드는 병렬화에 부적합
합치는 부담이 크다.

## 직접 구현한 Stream, Iterable, Collection 이라면?

- `spliterator` 메서드를 만드시 재정의하고, 스트림의 병렬화 성능을 강도높게 테스트해봐라
    
    ⇒ 직접 구현하는거는 어려움, 이 책에서 안알려줄 정도로 어려움
    
    ⇒ 그냥 웬만하면 하지말라고
    
    ![하지마이shake it](https://github.com/TightJava/effective_java/assets/13278955/34be9a09-9924-4b73-ada3-68d86f6c9970)

    하지마이shake it
    

---

# Stream 규약

- Stream의 `reduce` 연산에 건네지는 accumulator 와 combiner 함수는 반드시 지켜야한다.

- 결합법칙 만족 (associative) (a OP b ) OP c == a OP (b OP c)
- 간섭받지 않아야함 (non-interfering) (파이프라인 수행동안 데이터소스가 변경 X)
- 상태 없음 (stateless)

⇒ 이게 지켜지지 않아도 파이프라인이 순차적으로 실행되면 올바른 결과를 얻을 수도 있다.

---

# 병렬에서 순서를 순차처럼 하려면?

`.forEach()` ⇒ `forEachOrdered()`

순서 보장

---

# 과연 병렬이 효과있을까.Araboja

추정해보기

- 스트림 안의 원소 수와 원소당 수행되는 코드 줄 수를 곱하자
    - 원소 수 * 수행되는 코드 줄 수

```java
Stream.iterate(1, MyTestParallel::add)
				.limit(10)
				.forEach(System.out::println);
```

- `Iterate. limit (10)` ⇒ 원소수 10개 * `forEach(System.out::prinln)`; == 10
    
    ⇒ 의미없음
    

```java
public class UsefulParallel {

	static long pi(long n) {
		return LongStream.rangeClosed(2, n)
				.mapToObj(BigInteger::valueOf)
				.filter(i -> i.isProbablePrime(50))
				.count();
	}

	static long piWithParallel(long n) {
		return LongStream.rangeClosed(2, n) // 2부터 n 까지 범위로 리턴
				.mapToObj(BigInteger::valueOf) // 2~ n 까지 숫자들 각 BigInteger 변환
				.filter(i -> i.isProbablePrime(50)) // 2 ~ n 숫자를 각각 소수인지 확인
				.count(); // 소수인것들은 통과되어서 count (reduction 함수)
	}

	public static void main(String[] args) {
		long s = 0, e = 0;
		s = System.currentTimeMillis();
		pi(1000L);
		e = System.currentTimeMillis();
		System.out.println("time = " + (e - s));

		System.out.println("=================================");
		s = System.currentTimeMillis();
		piWithParallel(1000L);
		e = System.currentTimeMillis();
		System.out.println("time = " + (e - s));

time = 20
=================================
time = 6
	}
}
```

---

# 요약

- Stream 의 병렬처리인 parallel() 이 존재한다
- 얘를 쓰려면 주의해야할것들이 있다. 잘 못 쓰면 싱글보다 느리다.
    - 순서가 보장이 안된다는것이다. ⇒ 순서가 중요한데서는 쓰면 안된다.
    - limit 을 걸어서 사용할때는 사용하지 않는게 더 빠를 수 있다.
    - collect() 는 합치는 부담이 크다.
    - reduction 함수들은 쓰기 좋다.
- 쓸꺼면 계산해보고 확실히 낫다고 계산이 되었을때만 사용하자 + 성능지표 작성하면서 해라
