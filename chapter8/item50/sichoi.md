# 적시에 방어적 복사본을 만들라

아이템 한 줄 요약: 방어적 복사써라. 근데 비용이 너무 크거나 클라이언트가 믿을만하거나, 클라이언트가 바보여서 불변식 깨뜨려도 별 상관 없다면 안써도 된다.

자바는 클라이언트가 어떤짓을 해도 불변식이 지켜지는 안전한 언어이다.

그러나 클라이언트는 이런 불변식에 혈안이 되어 있다고 생각할정도로 방어적으로 프로그래밍해야 한다.

보안과 관련한 기본적이지만 중요한 내용들을 다루었던 아이템이었던 것 같다.

### 방어적 복사가 적용되지 않은 예시

```java
import java.util.Date;
import lombok.Getter;

@Getter
public final class DangerousPeriod {
	private final Date start;
	private final Date end;

	public DangerousPeriod(Date start, Date end) {
		if (start.compareTo(end) > 0)
			throw new IllegalArgumentException(start + " 가 " + end + " 보다 늦습니다.");
		this.start = start;
		this.end = end;
	}

	public static void main(String[] args) {
		Date start = new Date();
		Date end = new Date();
		DangerousPeriod dp = new DangerousPeriod(start, end);
		System.out.println("Before: " + dp.getEnd());
		end.setYear(78); // dp의 내부를 수정했다.
		System.out.println("After: " + dp.getEnd());
	}
}
```

```java
Before: Mon Jul 31 18:27:01 KST 2023
After: Mon Jul 31 18:27:01 KST 1978
```

Period라는 객체를 기간이 한 번 정해지면 수정이 안되도록 하려는 의도로 작성했다.

하지만 클라이언트가 이 의도를 이해하지 못하고, `end.setYear(78)`로 Period 내부 멤버변수를 너무나도 쉽게 바꿔버릴 수가 있다..!


<img width="720" alt="Untitled" src="https://github.com/TightJava/effective_java/assets/83565255/88e08815-bec9-4870-925d-51bcbf5a116a">


참고로 **`java.util.Date`는 너무 낡은 API이므로 앞으로 사용하면 안된다…!**

`Date.SetYear()`의 경우 대놓고 **Deprecated** 되었다고 나온다.

### 방어적 복사가 적용된 예시(완벽 적용 x)

```java
import java.util.Date;
import lombok.Getter;

@Getter
public final class SafePeriod {
	private final Date start;
	private final Date end;

	public SafePeriod(Date start, Date end) {
		this.start = new Date(start.getTime());
		this.end = new Date(end.getTime());
		if (this.start.compareTo(this.end) > 0)
			throw new IllegalArgumentException(start + " 가 " + end + " 보다 늦습니다.");
	}

	public static void main(String[] args) {
		Date start = new Date();
		Date end = new Date();
		SafePeriod dp = new SafePeriod(start, end);
		System.out.println("Before: " + dp.getEnd());
		end.setYear(78); // dp의 내부를 수정했다.
		System.out.println("After: " + dp.getEnd());
	}
}
```

```java
Before: Mon Jul 31 18:32:52 KST 2023
After: Mon Jul 31 18:32:52 KST 2023
```

이번에는 방어적 복사가 제대로 적용된 예시다.

생성자에서 `this.end = new Date(end.getTime());` 와 같이 매개변수로 받은 값을 다시 복사하여 사용하고 있다.

- **TOCTOU(Time of Check/Time of Use) 공격**
    
    멀티스레딩 환경에서는 원본 객체의 유효성을 검사한 후 복사본을 만드려고 하는 찰나의 취약한 순간에 다른 스레드에서 원본 객체를 수정해버리는 위험이 있다. 
    
    TOCTOU 공격은 이런 
    
    따라서 **매개변수의 유효성을 검사하기 전에 방어적 복사본을 만들어야 한다.**
    

### 방어적 복사가 적용된 예시(완벽 적용 x)

사실 앞선 코드는 방어적 복사가 완벽하게 적용되지 않았다.

```java
public static void main(String[] args) {
		Date start = new Date();
		Date end = new Date();
		SafePeriod dp = new SafePeriod(start, end);
		System.out.println("Before1: " + dp.getEnd());
		end.setYear(78); // dp의 내부를 수정했다.
		System.out.println("After1: " + dp.getEnd());

//		이렇게 하면 뚫리지롱~
		System.out.println("Before2: " + dp.getEnd());
		dp.getEnd().setYear(78); // dp의 내부를 수정했다.
		System.out.println("After2: " + dp.getEnd());
	}
```

getter 메소드가 내부의 가변 정보를 드러내기 때문에 `dp.getEnd().setYear(78);` 로 접근하면 객체 내부 정보를 수정할 수 있다.

```java
import java.util.Date;

public final class SuperSafePeriod {
	private final Date start;
	private final Date end;

	public SuperSafePeriod(Date start, Date end) {
		this.start = new Date(start.getTime());
		this.end = new Date(end.getTime());
		if (this.start.compareTo(this.end) > 0)
			throw new IllegalArgumentException(start + " 가 " + end + " 보다 늦습니다.");
	}

	public Date start() {
		return new Date(start.getTime());
	}

	public Date end() {
		return new Date(end.getTime());
	}

	public static void main(String[] args) {
		Date start = new Date();
		Date end = new Date();
		SuperSafePeriod dp = new SuperSafePeriod(start, end);
		System.out.println("Before1: " + dp.end());
		end.setYear(78); // dp의 내부를 수정했다.
		System.out.println("After1: " + dp.end());

		System.out.println("Before2: " + dp.end());
		dp.end().setYear(78); // dp의 내부를 수정했다.
		System.out.println("After2: " + dp.end());
	}
}
```

```java
Before1: Mon Jul 31 18:56:16 KST 2023
After1: Mon Jul 31 18:56:16 KST 2023
Before2: Mon Jul 31 18:56:16 KST 2023
After2: Mon Jul 31 18:56:16 KST 2023
```

이제는 getter에서도 **방어적 복사본**을 반환하도록 바꾸었기 때문에, Period는 이제 완벽하게 불변이 되었다!

- **Date 말고 LocalDateTime을 쓰자..!**
    
    ```java
    import java.time.LocalDateTime;
    import lombok.Getter;
    
    @Getter
    public final class LocalDateTimePeriod {
    	private final LocalDateTime start;
    	private final LocalDateTime end;
    
    	public LocalDateTimePeriod(LocalDateTime start, LocalDateTime end) {
    		this.start = start;
    		this.end = end;
    		if (this.start.compareTo(this.end) > 0)
    			throw new IllegalArgumentException(start + " 가 " + end + " 보다 늦습니다.");
    	}
    
    	public LocalDateTime start() {
    		return start;
    	}
    
    	public LocalDateTime end() {
    		return end;
    	}
    
    	public static void main(String[] args) {
    		LocalDateTime start = LocalDateTime.now();
    		LocalDateTime end = LocalDateTime.now();
    		LocalDateTimePeriod dp = new LocalDateTimePeriod(start, end);
    		System.out.println("Before1: " + dp.end());
    		end = end.plusYears(78); // 바꿔도 안되지롱~
    		System.out.println("After1: " + dp.end());
    
    		System.out.println("Before2: " + dp.end());
    		end = end.plusYears(78); // 바꿔도 안되지롱~
    		System.out.println("After2: " + dp.end());
    	}
    }
    ```
    
    Period 예시에서 사실 제일 좋은 방법은 구닥다리 Date를 안쓰는 것이다. **LocalDateTime**이라는 채신 라이브러리를 쓴다면 어떤 방법을 쓰던 원본 객체를 변경할 수 없다.
    

### 방어적 복사를 항상 써야할까?

하지만 항상 방어적 복사를 해야하는 것이 아니다. 

성능 저하가 따르기도 하고 (새로운 객체를 만들어야 하기 때문에), 애초에 항상 쓸 수 있는 것도 아니다.

**클라이언트가 신뢰할만한 경우**

같은 패키지에 속한 클래스이고, protected 키워드로 해당 패키지에서만 사용 가능하게 해두어서, 호출자가 컴포넌트 내부를 수정하지 않으리라는 신뢰(?)가 있다면 방어적 복사를 생략하는 것을 고려할 수도 있다. (대신 이 경우엔 매개변수나 반환값을 수정하지 말아야 함이 분명히 명세가 되어야할 것이다.)

**통제권을 넘겨받은 클라이언트가 트롤링(?)을 해도 영향이 없는 경우**

방어적 복사를 생략해서 통제권을 넘겨받은 클라이언트가 불변식을 깨더라도 그 영향이 클라이언트에게만 국한되는 경우에 한정하여 주의해야 한다.

이런 예시는 **래퍼 클래스 패턴**이 적용된 경우를 들 수 있다.

- **래퍼 클래스 패턴** 예시
    
    ```java
    public class WrapperClassExample {
    
    	public static class WrappedData {
    		private int data;
    
    		public WrappedData(int data) {
    			this.data = data;
    		}
    
    		public void setData(int data) {
    			this.data = data;
    		}
    
    		public int getData() {
    			return data;
    		}
    	}
    
    	public static class TestData {
    		private final WrappedData wrappedData;
    
    		public TestData(WrappedData wrappedData) {
    			this.wrappedData = new WrappedData(wrappedData.getData());
    		}
    
    		public WrappedData getWrappedData() {
    			// Return defensive copy
    			return new WrappedData(wrappedData.getData());
    		}
    	}
    
    	public static void main(String[] args) {
    		WrappedData originalData = new WrappedData(5);
    		TestData testData = new TestData(originalData);
    
    		System.out.println("Original data: " + originalData.getData());
    
    		// Client modifies its own copy of data
    		WrappedData clientData = testData.getWrappedData();
    		clientData.setData(10);
    
    		System.out.println("Original data after client modification: " + originalData.getData());
    		System.out.println("Client's data: " + clientData.getData());
    	}
    }
    ```
    
    클라이언트가 TestData에게 받은 WrappedData를 쉽게 수정하여 불변식을 깨뜨릴 수 있기는 하지만, WrappedData를 수정하더라도 원본 객체인 originalData는 영향을 받지 않는다.
    

래퍼 클래스는 특성상 클라이언트가 래퍼에 넘긴 객체에 접근할 수 있어, 쉽게 불변식을 깨뜨릴 수 있지만 그 영향은 오직 클라이언트만 받는다.

### 저자의 핵심 정리

클래스가 클라이언트로부터 받는 혹은 클라이언트로 반환하는 구성요소가 가변이라면 그 요소는 반드시 방어적으로 복사해야 한다.

복사 비용이 너무 크거나 클라이언트가 그 요소를 잘못 수정할 일이 없음을 신뢰한다면 방어적 복사를 수행하는 대신 해당 구성요소를 수정했을 때의 책임이 클라이언트에 있음을 문서에 명시하도록 하자.
