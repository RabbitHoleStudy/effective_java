## 핵심어

- Bit Field
- EnumSet

---

## 요약

- Bit Field보다 EnumSet이 더 낫다.
→ EnumSet 자체는 비트 필드를 사용하는 것과 유사하게 특정 크기(64)이하로는 long, 산술 연산을 통해 좋은 성능을 갖는다. 결론 : 너넨 몰라도 되니까 이걸로 써라.
- 자바 9까지는 불변 EnumSet이 없었는데, 지금은 있니?
    - 자바 11까지도 안나왔다 아직 있니?
    - 근데 아마 별로 필요 없다고 생각해서 안 만드는듯?
    - unmodifiable Set으로 래핑해서 구현할 수 있다고 한다.
- 기왕이면 인터페이스로 처리해라.

---

## 내 생각

**EnumSet이 아닌 Set과 같은 상위 클래스(혹은 인터페이스)로 멤버에 주입하게 되는 경우에, 하위 구현인 EnumSet이 갖는 장점을 잃는게 아닐까?**
→ EnumSet을 Set으로 참조하더라도 실제 객체는 EnumSet이므로 EnumSet의 성능 이점을 잃지는 않습니다. 하지만 EnumSet의 특정 메서드에 접근할 수 없으므로 이점에서는 제한을 받을 수 있습니다.

---

**그렇다면 상속의 경우에는?**

→ Java에서 상속이 일어나면, 하위 클래스는 상위 클래스의 모든 공용 (public) 및 보호된 (protected) 메서드에 접근할 수 있습니다. 하지만 이는 EnumSet과 표준 Set 인터페이스 사이의 다형성과는 약간 다르며, EnumSet의 특정 메서드를 상위 클래스인 AbstractSet 또는 Set 인터페이스를 통해 호출할 수 없다는 것에 변화가 없습니다.
다시 말해, EnumSet에서 특정 메서드를 사용하려면 참조 유형이 실제로 EnumSet이어야 합니다. 여기서 말하는 특정 메서드라는 것은 EnumSet에서만 제공되는 메서드들을 말합니다.
상속의 경우에는 하위 클래스가 상위 클래스의 메서드에 접근할 수 있다는 것이지, 특정 하위 클래스 (EnumSet이 비록 AbstractSet을 상속하지만) 의 메서드가 상위 클래스 (Set 또는 AbstractSet) 를 통해 접근 가능하게 되는 것은 아닙니다.
따라서, EnumSet을 Set 타입으로 참조하거나 다룰 경우에는, EnumSet의 특색 있는 메서드들에 접근할 수 없는 점은 변하지 않습니다. 이후에 그 변수를 다시 EnumSet으로 캐스팅하지 않는 이상요. 이는 상속의 관점에서도 같습니다.

---

**하위 클래스가 상위 클래스로 업캐스팅 됐을 때, 본인이 Override했던 메서드는 Override 했던 흐름대로 사용되는거지?**

→ 하위 클래스에서 오버라이드한 메서드를 따라갑니다. Java에서 멤버 함수는 동적으로 연결되므로, 실제 오브젝트 타입의 메서드가 호출됩니다.

![스크린샷 2023-07-24 오전 8.53.08.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c46ef84b-b0cb-4f06-a73a-91459ee25b0a/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-07-24_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_8.53.08.png)

```java
class Animal {
		public void bow() {
			System.out.println("왕왕");
		}
	}

	class Cat extends Animal {
		@Override
		public void bow() {
			System.out.println("야옹");
		}
	}

@Test
	void 오버라이드() {
		Animal catUpperCast = new Cat();
		catUpperCast.bow(); // 야옹
	}
```

---

## 예제 코드

- Bit Field로 구현하기

```java
public class MyText {
	public static final int STYLE_BOLD = 1 << 0; // 1
	public static final int STYLE_ITALIC = 1 << 1; // 2
	public static final int STYLE_UNDERLINE = 1 << 2; // 4
	public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8
	private int styles;

	// 스타일 적용
	public void applyStyle(int style) {
		styles |= style;
	}

	// 스타일 제거
	public void removeStyle(int style) {
		styles &= ~style;
	}
}
```

- EnumSet으로 구현하기

```java
@RequiredArgsConstructor
public class MyTextES {
	private final Set<Style> styles; // 주입할 때 EnumSet으로 넣는 것이 좋을 듯

	public void applyStyles(Set<Style> styles) {
		this.styles.addAll(styles);
	}

	public void removeStyles(Set<Style> styles) {
		this.styles.removeAll(styles);
	}

	public enum Style {
		BOLD, ITALIC, UNDERLINE, STRIKETHROUGH
	}
}
```
