## 핵심어

- Static Factory Method
클래스의 인스턴스를 반환하는 정적 메서드.
- Boxed Class
원시 타입을 Wrap한 자바에서 제공하는 기본 클래스. 원시타입과 암묵적인 상호호환(Boxing - Unboxing, 연산 등)을 제공한다.

---

## 요약

### 장점

- 이름을 가질 수 있다.
한 클래스에 같은 시그니처가 여러 개 필요하다면, 각 차이를 드러내는 이름으로 짓자.
- 호출될 때마다 인스턴스를 새로 생성하지 않을 수 있다.
인스턴스를 미리 만들어 놓거나, 새로 생성한 인스턴스를 캐싱하여 재활용 할 수 있다.
- 반환 타입의 하위 타입 객체를 반환할 수 있다.
구현 클래스를 공개하지 않고도, 그 객체를 반환할 수 있다.
- 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
하위 클래스이기만 하면 된다. 클라이언트는 팩터리가 주는 객체가 무엇인지 알 필요가 없다.
- 정적 팩터리 메서드를 작성하는 시점에 반환할 객체의 클래스가 존재하지 않아도 된다.
해당 인터페이스를 implements하게 될 것들에 대한 명세가 가능하다.

### 단점

- 팩터리 메서드만 제공하는 경우 하위 클래스를 만들 수 없다.
- 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

---

## 내 생각

이름적인 면에서도 그렇고, 무엇보다 들어오는 매개변수에 따라서 내부 구현을 숨기고 하위 객체들을 그때그때 맞게끔 반환해줄 수 있다는 점이 메리트가 있는 것 같다.

한편으로는 매개변수가 너무 많아지는 경우에 가독성이(생성자도 마찬가지지만) 안좋아지는 것 같은데, Builder와 반쯤 섞은 버전은 없을까 싶긴하다.

---

## 예제 코드

- 클래스

```java
@RequiredArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class ItemWithPrice implements Item {
	private final String name;
	private final Integer price;
}

/*------------------------------------------------------*/

@RequiredArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class ItemWithOrigin implements Item{
	private final String name;
	private final String origin;
}

/*------------------------------------------------------*/

@RequiredArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class ItemWithBarcode implements Item {
	private final String name;
	private final String barcode;
}

/*------------------------------------------------------*/

public interface Item {

	static ItemWithPrice bestSellingItemWithPrice = new ItemWithPrice("Best Selling Item", 100);
	static Item withPrice(String name, Integer price) {
		if (name.equals(bestSellingItemWithPrice.getName()))
			return bestSellingItemWithPrice; // 참조만 하는 인스턴스(멤버가 final)이므로 가능
		return new ItemWithPrice(name, price);
	}

	static Item withBarcode(String name, String barcode) {
		return new ItemWithBarcode(name, barcode);
	}

	static Item withOrigin(String name, String origin) {
		return new ItemWithOrigin(name, origin);
	}
}
```

- 인스턴스를 새로이 생성하지 않고(싱글턴, 캐싱) 반환해보기

```java
@Test
	void 중복_반환() {
		Item bestSellingItem1 = Item.withPrice("베스트셀러", 10000);
		Item bestSellingItem2 = Item.withPrice("베스트셀러", 20000);
		Item diffItem1 = Item.withPrice("삼겹살", 3000);
		Item diffItem2 = Item.withPrice("삼겹살", 4000);

		assertEquals(bestSellingItem1.hashCode(), bestSellingItem2.hashCode());
		assertNotEquals(diffItem1.hashCode(), diffItem2.hashCode());
	}
```

- 반환 타입의 하위 객체를 반환해보기 + 입력 매개변수에 따라 다른 클래스의 객체 반환해보기

```java
@Test
	void 매개변수에_따른_다른_하위객체() {
		Item itemWithBarcode = Item.withBarcode("광천김", "1lIlII!!ll1");
		Item itemWithPrice = Item.withPrice("삼다수", 1000);
		Item itemWithOrigin = Item.withOrigin("국산김치", "중국");

		Assertions.assertEquals(ItemWithBarcode.class, itemWithBarcode.getClass());
		Assertions.assertEquals(ItemWithPrice.class, itemWithPrice.getClass());
		Assertions.assertEquals(ItemWithOrigin.class, itemWithOrigin.getClass());
	}
```

공유 되지 않는 기능이 필요한 지점이 있는걸 보면(getPrice가 필요한데, Item에 그걸 달아버리면 다른 하위 객체들이 못함) 썩 좋은 예시는 아닌 듯.
