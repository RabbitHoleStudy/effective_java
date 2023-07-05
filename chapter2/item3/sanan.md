## 핵심어

- 싱글턴
싱글톤은 일종의 소프트웨어 디자인 패턴으로서, **생성자가 여러번 호출되더라도 실제로 생성된 객체는 하나**이고, 최초의 생성자가 생성한 객체를 리턴하는 방식을 얘기한다. 즉, 프로그램 런타임동안 해당 클래스는 하나의 인스턴스만 사용한다는 의미이다.
- 싱글턴 클래스의 직렬화
    
    [☕ 싱글톤 객체가 깨져버리는 경우 (역직렬화 / 리플렉션)](https://inpa.tistory.com/entry/JAVA-☕-싱글톤-객체-깨뜨리는-방법-역직렬화-리플렉션)
    
- Serializable?
    
    [[Java] 직렬화(Serialization)란 무엇일까?](https://devlog-wjdrbs96.tistory.com/268)
    
- Transient?

  변수에 붙이는 키워드로, 해당 변수가 직렬화되지 않게끔 설정한다.
    
    [Java transient이란?](https://nesoy.github.io/articles/2018-06/Java-transient)
    
- readResolve?
    
    메서드를 직접 정의하여 역직렬화 과정에서 만들어진 인스턴스 대신에 기존에 생성된 싱글톤 인스턴스를 반환하도록 하면 된다.
    
    만일 역직렬화 과정에서 자동으로 호출되는 `readObject` 메서드가 있더라도 `readResolve` 메서드에서 반환한 인스턴스로 대체된다.
    `readObject` 메서드를 통해 만들어진 인스턴스는 가비지 컬렉션 대상이 된다.
    
    [자바 직렬화: readResolve와 writeReplace](https://madplay.github.io/post/what-is-readresolve-method-and-writereplace-method)
    
    [☕ 싱글톤 객체가 깨져버리는 경우 (역직렬화 / 리플렉션)](https://inpa.tistory.com/entry/JAVA-☕-싱글톤-객체-깨뜨리는-방법-역직렬화-리플렉션)
    

---

## 요약

싱글턴 막 쓰면 테스트가 어려워진단다~
→ 하지만 스프링이 해냈죠?

public static final(필드 방식)을 통해 싱글턴임을 명백히 드러낼 수 있다.

정적 팩터리 메서드를 통해 유연하게 싱글턴을 사용할 수 있다.

---

## 내 생각

매 호출마다 같은 인스턴스들을 생성해내는 부분들을 줄이면 당연히 부하나 메모리 사용량도 적어지니 좋아질 것 같다. 스프링이 싱글턴 패턴으로 인한 테스트의 어려움을 DI와 테스트 라이브러리로 해결해준다고 생각한다.

---

## 예제 코드

- public 필드 방식의 싱글턴

```java
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class PublicSniper {

	public static final PublicSniper INSTANCE = new PublicSniper();

	public void shoot() {
		System.out.println("PUBLIC: 탕~!");
	}
```

- static 팩토리 메서드의 싱글턴

```java
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class FactorySniper {

	private static final FactorySniper INSTANCE = new FactorySniper("정상수", 33);
	private static final FactorySniper INSTANCE_IN_SPAIN = new FactorySniper("Fernando SangSu", 32);
	private final String name;
	private final Integer age;

	public static FactorySniper callSangSu() {
		return INSTANCE;
	}

	public static FactorySniper callSangSuInSpain() { // 호출별로 다른 인스턴스를 넘겨주는 방법
		return INSTANCE_IN_SPAIN;
	}

	public void shoot() {
		if (this.name.equals("정상수")) {
			System.out.println("FACTORY: 탕~!");
		} else {
			System.out.println("FACTORY: Señoritang~!");
		}
	}
}
```

- 열거 타입의 싱글턴

```java
public enum EnumSniper {
	INSTANCE;

	private final String name = "정상수";

	void shoot() {
		System.out.println("ENUM: 탕~!");
	}
}
```

- readResolve를 이용한 역직렬화 문제 해결해보기

```java
@AllArgsConstructor(access = lombok.AccessLevel.PRIVATE)
public class SerializableSniper implements Serializable {

	public static final SerializableSniper INSTANCE = new SerializableSniper();

	public void shoot() {
		System.out.println("SERIALIZABLE: 탕~!");
	}

	private Object readResolve() { // 이 선언을 통해 역직렬화 시에도 싱글턴을 보장할 수 있다.
		return INSTANCE;
	}
}

/*-----------------------------------------------------------------------------*/

public class SingletonTest {

	@Test
	void 싱글톤_테스트() throws IOException, ClassNotFoundException {
		PublicSniper.INSTANCE.shoot(); // Public 필드 방식의 싱글턴 이용
		/*-----------------------------------------------------------------------------*/
		FactorySniper.callSangSu().shoot(); // 정적 팩터리 메서드를 이용
		FactorySniper.callSangSuInSpain().shoot(); // 정적 팩터리 메서드의 응용(다른 호출, 다른 싱글턴)
		/*-----------------------------------------------------------------------------*/
		Supplier<FactorySniper> sangSuSupplier = FactorySniper::callSangSu; // 공급자, 람다를 이용한 싱글턴
		sangSuSupplier.get().shoot(); // Supplier는 Lazy Evaluation을 한다고 한다. 나중에 더 찾아봐야할 것 같다.
		/*-----------------------------------------------------------------------------*/
		EnumSniper.INSTANCE.shoot(); // 열거 타입 방식의 싱글턴 이용
		/*-----------------------------------------------------------------------------*/
		SerializableSniper sniper = SerializableSniper.INSTANCE;

		// 직렬화 - 블로그 참조
		String fileName = "singleton.obj";
		ObjectOutputStream out = new ObjectOutputStream(new BufferedOutputStream(new FileOutputStream(fileName)));
		out.writeObject(sniper);
		out.close();

		// 역직렬화
		ObjectInputStream in = new ObjectInputStream(new BufferedInputStream(new FileInputStream(fileName)));
		SerializableSniper sniperAfterSerialDeserialization = (SerializableSniper) in.readObject();
		in.close();

		/* 기존의 싱글턴을 반환하는 private Object readResolve가 클래스 내에 선언되어있지 않으면 false, 선언되어 있으면 true */
		assertEquals(sniper, sniperAfterSerialDeserialization);
	}
}

// 결과
// PUBLIC: 탕~!
// FACTORY: 탕~!
// FACTORY: Señoritang~!
// FACTORY: 탕~!
// ENUM: 탕~!
```
