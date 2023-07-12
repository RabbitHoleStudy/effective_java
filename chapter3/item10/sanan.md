## 핵심어

- **리스코프 치환 원칙(LSP, Liskov Subsitution Principle)**
**서브 타입은 언제나 기반 타입으로 교체(업캐스팅)할 수 있어야 한다는 것.**

---

## 요약

- 재정의하지 말아야 할 케이스
    - 각 인스턴스가 본질적으로 고유한 경우 - Thread와 같은 인스턴스
    - 인스턴스의 논리적 동치성을 검사할 일이 없는 경우
    → 각 인스턴스가 본질적으로 고유한 경우가 대체로 해당하는 것 같다.
    - 상위 클래스에서 Override한 equals가 딱 들어맞는 경우
    → 굳이 다시 만들 필요가 없는 경우
    - 클래스가 private / package-private이고 equals 메서드를 호출할 일이 없는 경우
    → 정 싫다면 equals를 사용하면 throw하도록 재정의할 수도 있다.
- 재정의해야 할 케이스
    - 객체 식별성이 아닌, 논리적 동치성을 확인해야하는데 상위 클래스(혹은 본인)의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때
    → 논리적 동치성이라는 말이 좀 어려워 보이긴 하지만, **비즈니스 로직상 같은 값으로 취급되어야 한다**는 의미로 해석했다.

---

## 내 생각

메모리에 존재하는 레퍼런스 주소로서의 비교냐, 아니면 해당 프로퍼티들의 동일함으로 비교하는 것이냐, 혹은 프로퍼티가 아니더라도 + 주소가 아니더라도 특정 식별자(엔티티의 ID 등)로 비교를 하느냐 등등 **원하는 비즈니스 로직에 따라서 그 동일성에 대한 관계를 설정하는 데에 사용하는 기본적인 메서드라고 생각**한다.

---

## 예제 코드

- 레퍼런스 주소로서의 비교(Default)

```java
@AllArgsConstructor
public class Reference {
	private final String name;
}
```

- 프로퍼티가 같은지에 대한 비교

```java
@AllArgsConstructor
public class Bundle {
	private final String item;
	private final Integer price;
	private final Integer quantity;

	@Override
	public boolean equals(Object o) {
		if (this == o) return true;
		if (!(o instanceof Bundle)) return false;
		Bundle bundle = (Bundle) o;
		return item.equals(bundle.item) && price.equals(bundle.price) && quantity.equals(bundle.quantity);
	}
}
```

- 특정 프로퍼티에 대한 비교

```java
@AllArgsConstructor
public class Entity {
	private final int id;
	private final String name;
	private final Integer age;

	@Override
	public boolean equals(Object o) {
		if (this == o) return true;
		if (!(o instanceof Entity)) return false;
		Entity entity = (Entity) o;
		return id == entity.id;
	}
}
```

- 테스트

```java
public class EqualsTest {

	@Test
	void equals_오버라이드() {
		/*----------------------------------다른 레퍼런스-----------------------------------*/
		Reference 서울사는_정상수 = new Reference("정상수");
		Reference 부산사는_정상수 = new Reference("정상수");
		assertNotEquals(서울사는_정상수, 부산사는_정상수); // Equals !Override && 다른 인스턴스
		/*----------------------------------모든 프로퍼티에 대한 equals-----------------------------------*/
		Bundle 계란_한판 = new Bundle("계란", 10000, 30);
		Bundle 계란_한판_유통기한_지남 = new Bundle("계란", 10000, 30);
		assertEquals(계란_한판, 계란_한판_유통기한_지남); // 같은 프로퍼티인 경우에 대한 Equals Override
		/*----------------------------------특정 프로퍼티에 대한 equals-----------------------------------*/
		Entity 정상수 = new Entity(1, "정상수", 30);
		Entity 정상수_런타임_중_나이먹음 = new Entity(1, "정상수", 31);
		assertEquals(정상수, 정상수_런타임_중_나이먹음); // 특정 프로퍼티인 경우에 대한 Equals Override
	}
}
```
