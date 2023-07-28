## 핵심어

- 함수형 인터페이스
    
    함수형 인터페이스(Functional Interface)는 Java 8에서 소개된 개념으로, 하나의 추상 메서드만을 가진 인터페이스를 말한다.
    
    @FunctionalInterface 어노테이션을 인터페이스에 추가하면, 컴파일러가 하나의 추상 메소드만을 포함하도록 강제할 수 있다.
    
    기능적으로 Java의 함수형 인터페이스는 람다 표현식을 허용하고, 메소드 참조와 생성자 참조를 이용하는 고차 함수(higher-order functions)를 가능케 한다.
    
    ```java
    @FunctionalInterface
    public interface SimpleInterface {
        // Single abstract method
        void doSomething();
    }
    ```
    
- 람다와 람다의 컴파일 방식
    
    람다 표현식은 본질적으로 함수형 프로그래밍에서 사용하는 개념이며, 메소드를 하나의 식으로 표현한 것이다. 
    
    따라서 Java에서 람다는 특별한 유형의 익명 함수라고 할 수 있으며, 코드 블록을 추상 메서드에 바로 전달하는 간결한 방식이다.
    
    독립적인 함수를 만들거나 대체하는 것이 아닌, 람다 표현식은 공식적으로 특정 함수형 인터페이스의 인스턴스를 만든다. 이를 통해 개발자는 메소드를 일급 시민으로 취급하고, 컬렉션에 저장하거나, 메소드의 인수로 전달하거나, 결과로 반환할 수 있다. 그렇기 때문에 람다를 '익명 메서드'보다는 '익명 함수'에 더 가깝다고 할 수 있다.
    
    람다 표현식이 런타임에 작동하는 방식은 invokedynamic이라는 JVM 명령어를 통해 수행된다. invokedynamic은 Java 7에서 추가되었으며, 동적 언어의 지원을 목적으로 설계되었다. invokedynamic을 사용하면, JVM은 람다 코드를 최적화하여 실행할 수 있다. 이러한 최적화는 익명 클래스를 사용할 때와는 차이가 있다.
    
    따라서 람다 표현식은 컴파일 시점에 익명 클래스로 변환되고, 이를 통해 함수형 인터페이스의 메서드를 구현한다. 그러나 런타임에서는 invokedynamic을 사용하여 JVM이 람다를 효율적으로 실행하게끔 돕게된다.
    

---

## 요약

- 컴파일러가 문맥을 살펴서 해주는 타입 추론은 개복잡하다. 너네가 알 필요 없고 언제 명확히 타입을 지정해줘야 하는 지에 대해서만 고려해라.
→ 컴파일러는 제네릭에서 타입 정보 대부분을 얻는다.
- 코드 자체로 설명이 잘 안 되거나, 길어진다면 람다 쓰지 마라.
→ 람다는 3줄 요약으로 써라.
- 열거 타입 생성자에 넘겨지는 인수들은 컴파일 타임에 추론된다. 따라서 열거 타입 생성자 안의 람다는 열거 타입 인스턴스의 멤버에 접근할 수 없다.
→ 열거 타입의 인스턴스(상수)는 런타임에 생성되는 인스턴스이기 때문이다.
→ 따라서 만약 필요하다면, 즉 몇 줄로 구현하기 어렵거나 인스턴스의 멤버를 사용해야한다면 상수별로 익명 클래스를 작성해보자.
- 추상 클래스의 인스턴스를 만들 때에는 람다를 쓸 수 없다 → 익명 클래스를 써야한다.
- 람다는 자신을 참조할 수 없다 → 람다의 this는 바깥 인스턴스를 가리킨다.

---

## 내 생각

![wqerasdfsadf](https://github.com/TightJava/effective_java/assets/105692206/d91033b1-bfe6-435c-b109-6e1a5f9a23a1)


‘그때그때 필요한 경우에 만들어서 쓰기’라는 점에서 람다가 맘에 드는 것 같다.

보는 사람 입장에서 그 흐름을 따라서 읽을 수 있으니 말이다.

하지만 그 작성을 하는 데에 있어서 주의를 많이 기울여야하는 점도 있는 것 같다. 

결국 남이 보기에 쉬워보이려면 그 내부적인 로직을 잘 이해해야할 것 같다.

---

## 예제 코드

- 함수형 인터페이스를 작성하고 람다식 사용해보기

```java
public interface MyOperator<T> {
	T apply(T a, T b);
}
/*---------------------------------------------------*/
public class IntegerCalculator {
	public Integer add(Integer a, Integer b) {
		return Operand.PLUS.apply(a, b);
	}

	public Integer subtract(Integer a, Integer b) {
		return Operand.MINUS.apply(a, b);
	}

	public Integer multiply(Integer a, Integer b) {
		return Operand.TIMES.apply(a, b);
	}

	public Integer divide(Integer a, Integer b) {
		return Operand.DIVIDE.apply(a, b);
	}

	enum Operand {
		PLUS("+", (a, b) -> a + b),
		MINUS("-", (a, b) -> a - b),
		TIMES("*", (a, b) -> a * b),
		DIVIDE("/", (a, b) -> a / b),
		;

		private final String symbol;
		private final MyOperator<Integer> operator;

		Operand(String symbol, MyOperator<Integer> operator) {
			this.symbol = symbol;
			this.operator = operator;
		}

		public int apply(int a, int b) {
			return operator.apply(a, b);
		}

		@Override
		public String toString() {
			return symbol;
		}

		public Operand fromString(String symbol) {
			for (Operand operand : values()) {
				if (operand.symbol.equals(symbol)) {
					return operand;
				}
			}
			throw new IllegalArgumentException();
		}
	}
}
/*---------------------------------------------------*/
@Test
	void 함수형_인터페이스_람다() {
		IntegerCalculator integerCalculator = new IntegerCalculator();

		Assertions.assertThat(integerCalculator.add(1, 2)).isEqualTo(3);
		Assertions.assertThat(integerCalculator.subtract(1, 2)).isEqualTo(-1);
		Assertions.assertThat(integerCalculator.multiply(1, 2)).isEqualTo(2);
		Assertions.assertThat(integerCalculator.divide(1, 2)).isEqualTo(0); // 1/2 -> 0
	}
```
