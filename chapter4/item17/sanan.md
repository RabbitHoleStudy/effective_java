## 핵심어

- Mutable(변경 가능성)
- Immutable Class(불변 클래스)
기본적인 박싱 클래스
- 함수형과 절차/명령형의 차이 - 피연산자의 상태 변경
- 동반 클래스
    
    동반 클래스는 클래스와 동일한 이름을 가진 클래스다. 
    컴패니언 클래스는 클래스와 같은 파일/패키지에 선언되며, 클래스의 인스턴스 없이도 사용할 수 있다.
    
    - 클래스의 인스턴스 없이 사용할 수 있는 정적 메서드 또는 필드를 선언하려는 경우
    - 클래스의 인스턴스와 관련된 값을 저장하려는 경우
    - 클래스의 인스턴스와 관련된 로직을 구현하려는 경우

---

## 요약

- 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.
→ 합당한 이유가 없다면 모든 필드는 private final이어야 한다.
- 불변으로 만들 수 없다면, 최대한 가변 요소를 줄여라.
- 불변 클래스를 구성하는 다섯 가지 규칙
    1. 모든 필드를 final로 선언한다.
    2. 모든 필드를 private으로 선언한다.
    3. Setter를 제공하지 않는다.
    4. 상속 불가능하게 만든다(확장 할 수 없도록, 하위 클래스가 없도록).
    5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
    → 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야 한다.
    필요하다면 방어적 복사.
- 불변 객체는 Thread-Safe다.

---

## 내 생각

- 불변 객체의 새 인스턴스 반환 vs 방어적 복사
방어적 복사는 가변 필드를 가지고 있는 클래스의 상태를 보장하기 위함 + 깊은 복사를 위한 더 큰 비용을 들이는 방법일 수 있다. 불변 객체의 새 인스턴스 반환은 불필요한 부분이라고 생각한다. 만약
- 성능 이슈는 없는가?
아마도 잦은 생성이 이뤄져야한다면, BigInteger의 경우처럼 Mutable한 동반 클래스를 내부적으로 이용해서 성능을 높이는 방법도 있을 것 같다(복잡할 것 같지만).

---

## 예제 코드

- 동반 클래스 구현

```java
@RequiredArgsConstructor(access = AccessLevel.PACKAGE)
@EqualsAndHashCode
public class Car {
	private final String maker;
	private final String model;
	private final int year;
}

class CarFactory {
	public static Car createCar(String maker, String model, int year) {
		return new Car(maker, model, year);
	}
}
```

- 불변 객체 구현

```java
@AllArgsConstructor
@Getter
public class InstanceOnHeap {
	private int count;

	public static InstanceOnHeap of(int count) {
		return new InstanceOnHeap(count);
	}

	public void add() {
		count++;
	}
}
/*-----------------------------------------------------------------------------*/
@AllArgsConstructor
public final class Immutable {
	private final String name;
	private final String address;
	private final int count;
	private InstanceOnHeap mutableCount; // 참조는 외부에서 받을 수 없음

	public String getName() {
		return name;
	}

	public String getAddress() {
		return address;
	}

	public int getCount() {
		return count;
	}

	public InstanceOnHeap getMutableCountDefensively() {
		return InstanceOnHeap.of(mutableCount.getCount());
	}

	public void increaseMutableCount() {
		mutableCount.add();
	}

	public Immutable functionalLikeImmutableAdd() {
		return new Immutable(name, address, count + 1, mutableCount);
	}
}
```

- 테스트

```java
public class ImmutableTest {

	@Test
	void 불변_클래스() {
		Immutable immutable = new Immutable("name", "address", 0, InstanceOnHeap.of(0));
		assertEquals(0, immutable.getCount());
		assertEquals(0, immutable.getMutableCountDefensively().getCount());
		InstanceOnHeap defensive = immutable.getMutableCountDefensively();
		InstanceOnHeap defensive2 = immutable.getMutableCountDefensively(); // 방어적 복사(스냅샷)이기 때문에 다른 인스턴스다.
		assertNotEquals(defensive.hashCode(), defensive2.hashCode());
		/*------------------------------------함수형 따라해보기-----------------------------------------*/
		long startTime = System.currentTimeMillis();
		for (int i = 0; i < 1000000; i++) {
			immutable = immutable.functionalLikeImmutableAdd();
		}
		long endTime = System.currentTimeMillis();
		System.out.println("함수형 따라하기 : " + (endTime - startTime) + "ms"); // 8ms
		
		startTime = System.currentTimeMillis();
		for (int i = 0; i < 1000000; i++) {
			immutable.increaseMutableCount();
		}
		endTime = System.currentTimeMillis();
		System.out.println("그냥 더하기 : " + (endTime - startTime) + "ms"); // 4ms
	}

	@Test
	void 동반_클래스() {
		Car car = CarFactory.createCar("maker", "model", 2021);
		Car car2 = new Car("maker", "model", 2021);
		assertEquals(car, car2);
	}
}
```
