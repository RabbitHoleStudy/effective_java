## 핵심어

- 자바의 HashCode
해싱 알고리즘을 통해, 런타임에 인스턴스가 갖는 자체의 식별 값

---

## 요약

- **논리적으로 같은(equals) 객체는 같은 해시코드를 반환해야 한다**.
- Hash를 사용하는 자료구조의 경우, **equals와 hashcode가 둘 다 일치해야 같은 엔트리로** 여긴다.

---

## 내 생각

- 왜 31을 곱하는가?
31이 소수기도 하고.. 아무튼 중복 안 되게 하기 위함이라고한다.
- 결국 중요한 점은 equals와 hashcode가 일관성있게 override 되어야하고, 이는 중요한 자료에 따라 지정하고, 식별하기 위함이라는 점을 알아두자.
- 또한, 오브젝트를 프로퍼티로 갖는 경우에도 해당 오브젝트에 대한 HashCode가 오버라이드 되어 있어야 보장될 수 있다.

---

## 예제 코드

- 예시 클래스

```java
@AllArgsConstructor
public class Contact {

	private final String name;
	private final PhoneNumber phoneNumber;

	@Override
	public boolean equals(Object o) {
		if (this == o) return true;
		if (!(o instanceof Contact)) return false;
		Contact contact = (Contact) o;
		return name.equals(contact.name) &&
				phoneNumber.equals(contact.phoneNumber);
	}
}
/*-----------------------------------------------------------------------------*/
@AllArgsConstructor
public class HashCodeContact {

	private final String name;
	private final PhoneNumber phoneNumber;

	@Override
	public boolean equals(Object o) {
		if (this == o) return true;
		if (!(o instanceof HashCodeContact)) return false;
		HashCodeContact hashCodeContact = (HashCodeContact) o;
		return name.equals(hashCodeContact.name) &&
				phoneNumber.equals(hashCodeContact.phoneNumber);
	}

	@Override
	public int hashCode() {
		int result = name.hashCode();
		result = 31 * result + phoneNumber.hashCode();
		return result;
	}
}
/*-----------------------------------------------------------------------------*/
@AllArgsConstructor
public class PhoneNumber {

	private final String areaCode;
	private final String prefix;
	private final String lineNumber;

	@Override
	public boolean equals(Object o) {
		if (this == o) return true;
		if (!(o instanceof PhoneNumber)) return false;
		PhoneNumber that = (PhoneNumber) o;
		return areaCode.equals(that.areaCode) &&
				prefix.equals(that.prefix) &&
				lineNumber.equals(that.lineNumber);
	}

	@Override
	public int hashCode() {
		int result = areaCode.hashCode();
		result = 31 * result + prefix.hashCode();
		result = 31 * result + lineNumber.hashCode();
		return result;
	}
}
```

- 테스트

```java
public class HashCodeTest {
	@Test
	void HashCode_오버라이드() {
		/*-----------------------------------HashCode 재정의 X------------------------------------------*/
		Map<Contact, PhoneNumber> hashMap = new HashMap<>();
		hashMap.put(new Contact("홍길동", new PhoneNumber("010", "1234", "5678")), new PhoneNumber("010", "1234", "5678"));
		hashMap.put(new Contact("홍길동", new PhoneNumber("010", "1234", "5678")), new PhoneNumber("010", "1234", "5678"));
		assertNotEquals(1, hashMap.size()); //같은 엔트리임에도 불구하고 2개가 들어가는 이유는 hashCode를 오버라이드 하지 않았기 때문이다.
		/*-----------------------------------HashCode 재정의 O------------------------------------------*/
		Map<HashCodeContact, PhoneNumber> hashMap2 = new HashMap<>();
		hashMap2.put(new HashCodeContact("홍길동", new PhoneNumber("010", "1234", "5678")), new PhoneNumber("010", "1234", "5678"));
		hashMap2.put(new HashCodeContact("홍길동", new PhoneNumber("010", "1234", "5678")), new PhoneNumber("010", "1234", "5678"));
		assertEquals(1, hashMap2.size()); //hashCode를 오버라이드 하였기 때문에 같은 엔트리는 1개만 들어간다.
		//PhoneNumber 또한 HashCode가 Override 되어 있어야 한다.
	}
}
```
