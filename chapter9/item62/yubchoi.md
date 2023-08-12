# 다른 타입이 적절하다면 문자열 사용을 피하라
**📌 기본 타입이든 참조 타입이든 적절한 값 타입이 있다면 그것을 사용하고, 없다면 새로 작성하라. 편하다고 문자열을 남용하지 말자.**

### 문자열은 다른 값 타입을 대신하기에 적합하지 않다.
문자열은 열거 타입이나 혼합 타입을 대신하거나 권한(capacity)을 표현하기에 적합하지 않다.   
입력 받은 데이터가 무엇이냐에 따라 적절한 타입을 사용하자. 적절한 타입이 없다면 새로 하나 정의할 수도 있다.
- 수치형 → int, float, BigInteger
- 예/아니오 질문의 대답 → 적절한 열거 타입이나 boolean

### 잘못된 예 - 혼합 타입을 문자열로 처리
```java
String  key = className  + "#" + iterator.next();
```
- 구분자 `#`을 기준으로 문자열을 파싱해야 하는데 그렇게 되면 속도와 정확성이 떨어진다.
- 적절한 equals, toString, compareTo 메서드를 제공할 수 없다.

**private 정적 멤버 클래스로 선언하자.**
```java
public class OuterClass {
	private static class CompoundKey {
		private final String className;
		private final String nextValue;

		public CompoundKey(String className, String nextValue) {
			this.className = className;
			this.nextValue = nextValue;
		}

		@Override
		public String toString() {
			return className + "#" + nextValue;
		}
	}
}
```

### 잘못된 예 - 문자열을 사용해 권한을 구분
```java
public class ThreadLocal {
	private ThreadLocal() {
	}

	// 현 스레드의 값을 키로 구분해 저장
	public static void set(String key, Object value) {
	}

	// (키가 가리키는) 현 스레드의 값을 반환
	public static Object get(String key) {
		return key;
	}
}
```
스레드 구분용 문자열 키가 global namespace 공간에서 공유된다.
- 의도치 않게 서로 다른 두 클라이언트가 같은 키를 사용한다면 스레드 구분이 불가해진다.
- 악의적인 클라이언트로 인한 보안 문제점도 있다.

**Key 클래스를 통해 권한을 구분하자.**
```java
public class ThreadLocal {
	private ThreadLocal() {
	}

	public static class Key {
		Key() { }
	}

	// 위조 불가능한 고유 키를 생성
	public static Key getKey() {
		return new Key();
	}

	public static void set(Key key, Object value);

	public static Object get(Key key);
}
```
하지만 더 개선할 수 있다. **set과 get은 정적 메서드일 필요가 없으니 Key 클래스의 인스턴스 메서드로 바꾸자.** 그렇게 되면 톱레벨 클래스인 ThreadLocal이 하는 일이 없어지므로 ThreadLocal을 지우고 Key의 이름을 ThreadLocal로 바꾸자.   
또, **타입 안전하게 사용하기 위해 매개변수화 타입으로 ThreadLocal을 선언하자.**
```java
public final class ThreadLocal<T> {
	public ThreadLocal();
	public void set(T value);
	public T get();
}
```
