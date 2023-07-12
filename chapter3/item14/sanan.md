## 핵심어

---

## 요약

- **Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서(Natural Order)가 있음을 뜻한다.** → 예시에서는 핸드폰의 번호(지번, 번호1, 번호2)를 예시로 든다.
- 순서가 있음이 명확한 클래스에 Comparable을 구현하라.
- 정렬된 컬렉션들은 equals가 아닌 compareTo로 동치성을 비교한다.
- **compareTo는 각 필드가 동치인 것이 아닌, 그 순서를 비교한다.**
- 정적 compare 메서드를 활용하거나, 비교자 생성 메서드를 활용하여 비교자를 만들어라.

---

## 내 생각

- 단순 값의 비교가 아닌, ‘순서’라는 논리적 동치성이 필요한 경우에 사용할 것 같다.

---

## 예제 코드

- 클래스

```java
@AllArgsConstructor
@EqualsAndHashCode
public class ComparablePhoneNumber implements Comparable<ComparablePhoneNumber> {

	//비교자 생성 메서드
	private static final Comparator<ComparablePhoneNumber> COMPARATOR =
			Comparator.comparing((ComparablePhoneNumber pn) -> pn.areaCode)
					.thenComparing(pn -> pn.prefix)
					.thenComparing(pn -> pn.lineNumber);
	//정적 compare 메서드
	public static Comparator<Object> hashCodeOrder = new Comparator<>() {
		public int compare(Object o1, Object o2) {
			return o1.hashCode() - o2.hashCode();
		}
	};
	private final String areaCode;
	private final String prefix;
	private final String lineNumber;

	@Override
	public int compareTo(ComparablePhoneNumber phoneNumber) {
		int result = areaCode.compareTo(phoneNumber.areaCode);
		if (result == 0) {
			result = prefix.compareTo(phoneNumber.prefix);
			if (result == 0) {
				result = lineNumber.compareTo(phoneNumber.lineNumber);
			}
		}
		return result;
	}

	public int compareToByComparator(ComparablePhoneNumber phoneNumber) {
		return COMPARATOR.compare(this, phoneNumber);
	}
}
```

- 테스트

```java
public class ComparableTest {

	//compareTo의 반환 타입은 int이다.
	private static final int TRUE = 0;

	@Test
	void Comparable_테스트() {
		ComparablePhoneNumber sameNumber = new ComparablePhoneNumber("010", "1234", "5678");
		ComparablePhoneNumber anotherSameNumber = new ComparablePhoneNumber("010", "1234", "5678");
		ComparablePhoneNumber diffNumber = new ComparablePhoneNumber("010", "1234", "5679");

		/*-----------------------------------기본 필드 비교------------------------------------------*/
		assertEquals(TRUE, sameNumber.compareTo(anotherSameNumber));
		assertNotEquals(TRUE, sameNumber.compareTo(diffNumber));
		/*---------------------------비교자 생성 메서드 - 내부 비교자를 이용-------------------------------*/
		assertEquals(TRUE, sameNumber.compareToByComparator(anotherSameNumber));
		assertNotEquals(TRUE, sameNumber.compareToByComparator(diffNumber));
		/*--------------------------------정적 compare 메서드를 이용-----------------------------------*/
		assertEquals(TRUE, ComparablePhoneNumber.hashCodeOrder.compare(sameNumber, anotherSameNumber));
		assertNotEquals(TRUE, ComparablePhoneNumber.hashCodeOrder.compare(sameNumber, diffNumber));
	}
}
```
