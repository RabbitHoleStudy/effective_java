# @Override 애너테이션을 일관되게 사용하라

**이번 아이템 한 줄 요약**: 상위 클래스 메소드 재정의할 때 **@Override**를 꼭 달자!

### 잘못 작성한 예시

```java
public class Bigram {
	private final char first;
	private final char second;

	public Bigram(char first, char second) {
		this.first = first;
		this.second = second;
	}

	public boolean equals(Bigram b) { // 이상한 점이 느껴지시나요?
		return b.first == first && b.second == second;
	}

	public int hashCode() {
		return 31 * first + second;
	}

	public static void main(String[] args) {
		Set<Bigram> s = new HashSet<>();
		for (int i = 0; i < 10; i++)
			for (char ch = 'a' ; ch <= 'z' ; ch++)
				s.add(new Bigram(ch, ch));
		System.out.println(s.size());
	}
}
```

```java
260
```

**Set**은 원소가 동일하면 같은 값으로 인식하기 때문에 **Set**에 **(a, a) ~ (z, z)**까지의 **Bigram** 값을 10번씩 저장하려고 시도하고, 출력하면 **26**이 나오는 것이 정상적일 것이다.

그런데 **260**이라고 나와버렸다..?!

### 정상적인 예시

```java
package chapter6.item40;

import java.util.HashSet;
import java.util.Set;

public class Bigram {
	private final char first;
	private final char second;

	public Bigram(char first, char second) {
		this.first = first;
		this.second = second;
	}

	@Override
	public boolean equals(Object o) {
		if (!(o instanceof Bigram))
			return false;
		Bigram b = (Bigram) o;
		return b.first == first && b.second == second;
	}

	public int hashCode() {
		return 31 * first + second;
	}

	public static void main(String[] args) {
		Set<Bigram> s = new HashSet<>();
		for (int i = 0; i < 10; i++)
			for (char ch = 'a' ; ch <= 'z' ; ch++)
				s.add(new Bigram(ch, ch));
		System.out.println(s.size());
	}
}
```

```java
26
```

원인은 재정의하려고 했던 equals()의 인자를 Object가 아닌 Bigram로 작성했기 때문이다..!

이 때문에 `재정의(Overriding)`가 아니라 `다중정의(Overloading)`로 인식이 되었던 것이다..!

그래서 같은 값을 가지는 **Bigram** 객체 10개씩이 서로 다른 객체로 인식되어 총 260개의 원소가 생겨버렸던 것이다.

Object의 equals를 재정의하려면 `equals(Object o)`로 작성해야 한다.

```java
@Override
public boolean equals(Bigram b) {
	return b.first == first && b.second == second;
}
```

만약 이런식으로 **@Override**를 달아줬다면 컴파일러는 ***메서드는 상위 클래스의 메서드를 재정의하지 않습니다*** 라는 경고문을 던져주어 잘못된 부분을 바로 발견하고 수정할 수 있었을 것이다..!

### 저자의 핵심 정리

재정의한 모든 메소드에서 애너테이션을 의식적으로 달면 여러분이 실수했을 때 컴파일러가 바로 알려줄 것이다. 

예외는 한가지 뿐이다. 

구체 클래스에서 상위 클래스의 추상 메소드를 재정의한 경우엔 애너테이션을 달지 않아도 된다. (그래도 다는게 좋다.)
