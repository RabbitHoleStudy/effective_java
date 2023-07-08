## 핵심어

- Dependency Injection(DI, 의존성 주입)
’사용하는 자원을 [매개변수로] 넣어준다’라고 이해하고 있다.
’의존성’이라는 단어를 사용하는 이유는, 그 자원이 변했을 때 영향을 받는(의존성)을 갖기 때문이다.
- Dependency Inversion Principle(DIP, 의존성 역전 원칙)
    
    > 고차원 모듈은 저차원 모듈에 의존하면 안된다. 
    이 모듈 모두 다른 추상화된 것에 의존해야 한다.
    > 
    > 
    > 추상화 된 것은 구체적인 것에 의존하면 안 된다. 
    > 구체적인 것이 추상화된 것에 의존해야 한다.
    >  - 마틴 파울러
    > 
    
    즉, 고차원 모듈이 추상화된 클래스(인터페이스)에 대해 의존하고, 그 구현체(하위 모듈)들이 해당 인터페이스에 따르도록(의존하도록)하는 것이다.
    
    이 때 중요한 점은 단순히 인터페이스를 쓴다고 DIP가 아니라,
    `**고차원 모듈이 원하는 동작을 인터페이스에 명세하고 이를 하위 모듈이 따르게끔 하는 것**`이라는 점이다.
    
- Inversion of Control (IoC, 제어의 역전)
**제어 반전**, **제어의 반전**, **역제어**는 프로그래머가 작성한 프로그램이 재사용 라이브러리의 흐름 제어를 받게 되는 소프트웨어 디자인 패턴을 말한다. [(참고)](https://medium.com/@jang.wangsu/di-inversion-of-control-container-%EB%9E%80-12ecd70ac7ea)
즉, 코드 레벨에서의 직접 제어(예시로 의존성 주입)이 아닌, 프레임워크를 통해 제어되는 것이다.
→ 프로그래머가 작성한 프로그램이 외부 라이브러리의 코드를 호출하는 것이 아닌, 외부 라이브러리(스프링)이 프로그래머가 작성한 코드를 호출하는 것이다(예시로 @Autowired를 들 수 있을 것 같다).
- IoC Container
스프링에서 생성된 인스턴스들을 추가적으로 관리(Bean)하고, 의존성을 관리(DI)해주는 컨테이너다.

---

## 요약

클래스가 하나 이상의 자원에 의존하고, 그 자원에 영향을 받는다면 싱글턴이나 정적 유틸리티 클래스는 사용하지 않는 것이 좋다.

한 클래스에서 필요한 외부 자원을 그대로 때려 박아놓으면 코드 자체가 유연하지 않고(바뀐 경우 수정해야 함) 테스트하기 어려워진다(이미 박혀있기 때문).

의존 객체 주입이 유연성과 테스트 용이성을 개선해주긴 하지만, 코드를 어지럽게 만든다 - 스프링과 같은 프레임워크를 사용하면 이 어질러짐을 해소할 수 있다(어떻게?).

---

## 내 생각

- 스프링이 어질어질한 객체 주입을 예쁘게 만드는 방법
만약 스프링의 IoC 컨테이너가 없었다면 일일히 해당하는 매개변수에 대한 생성자들을 적고, 그것을 주입 받는.. 또 그것을 주입 받는.. 하는 구조들을 계속 변경해야할 것이다. 하지만, 제어의 역전을 통해서 그런 부분들(일일히 갈아끼우기)을 신경쓰지 않고 편하게 사용하게끔 해준다.

---

## 예제 코드

Dictionary를 통해서, 언어에 따라 인사를 다르게 하는 Bot을 만들어보자.

```java
public interface Dictionary {

	void hello();

	void bye();

	boolean isFrom(Language language);
}

/*-----------------------------------------------------------------------------*/

@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class KoreanDictionary implements Dictionary {

	private static final KoreanDictionary INSTANCE = new KoreanDictionary();
	private static final String hello = "안녕하세요";
	private static final String bye = "안녕히 계세요";
	private static final Language language = Language.KOREAN;

	public static KoreanDictionary getInstance() {
		return INSTANCE;
	}

	@Override
	public void hello() {
		System.out.println(hello);
	}

	@Override
	public void bye() {
		System.out.println(bye);
	}

	@Override
	public boolean isFrom(Language language) {
		return KoreanDictionary.language.equals(language);
	}
}

/*-----------------------------------------------------------------------------*/

@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class EnglishDictionary implements Dictionary {

	private static final EnglishDictionary INSTANCE = new EnglishDictionary();
	private static final String hello = "Hello";
	private static final String bye = "Goodbye";
	private static final Language language = Language.ENGLISH;

	public static EnglishDictionary getInstance() {
		return INSTANCE;
	}

	@Override
	public void hello() {
		System.out.println(hello);
	}

	@Override
	public void bye() {
		System.out.println(bye);
	}

	@Override
	public boolean isFrom(Language language) {
		return EnglishDictionary.language.equals(language);
	}
}

/*-----------------------------------------------------------------------------*/

public class CreatableSpanishDictionary implements Dictionary {

	private static final String hello = "Hola";
	private static final String bye = "Adiós";
	private static final Language language = Language.SPANISH;

	@Override
	public void hello() {
		System.out.println(hello);
	}

	@Override
	public void bye() {
		System.out.println(bye);
	}

	@Override
	public boolean isFrom(Language language) {
		return CreatableSpanishDictionary.language.equals(language);
	}
}
```

한국어, 영어는 싱글톤, 스페인어는 생성 가능하게 해두었다.

- 생성자로 주입하기

```java
@RequiredArgsConstructor
public class ConstructorBot {
	private final Dictionary dictionary;

	public void sayHello() {
		dictionary.hello();
	}

	public void sayBye() {
		dictionary.bye();
	}
}
```

- 정적 팩터리로 자원 설정하기

```java
@RequiredArgsConstructor(access = AccessLevel.PRIVATE)
public class FactoryBot {
	private final Dictionary dictionary;

	public static FactoryBot of(Language language) {
		if (language.equals(Language.KOREAN)) {
			return new FactoryBot(KoreanDictionary.getInstance());
		}
		if (language.equals(Language.ENGLISH)) {
			return new FactoryBot(EnglishDictionary.getInstance());
		}
		if (language.equals(Language.SPANISH)) {
			return new FactoryBot(new CreatableSpanishDictionary());
		}
		throw new IllegalArgumentException("지원하지 않는 언어입니다.");
	}

	public void sayHello() {
		dictionary.hello();
	}

	public void sayBye() {
		dictionary.bye();
	}
}
```

- 빌더로 주입하기

```java
public class BuilderBot {
	private final Dictionary dictionary;

	@Builder(access = AccessLevel.PROTECTED)
	public BuilderBot(Dictionary dictionary) {
		this.dictionary = dictionary;
	}

	public void sayHello() {
		dictionary.hello();
	}

	public void sayBye() {
		dictionary.bye();
	}
}
```

생성자 주입이랑 별다른 점을 잘 모르겠다.

- 사용되는 자원의 팩터리(Provider)를 넘겨주는 방식

```java

@RequiredArgsConstructor
public class FactoryInjectedBot {

	private final DictionaryFactory dictionaryFactory;
	private Dictionary dictionary;

	public void changeDictionaryByFactory(Language language) {
		if (language.equals(Language.SPANISH)) {
			this.dictionary = dictionaryFactory.createSpanishDictionary(); // 얘만 새 인스턴스 만들어줌
			return;
		}
		if (language.equals(Language.KOREAN)) {
			this.dictionary = dictionaryFactory.getKoreanDictionary(); // 얘는 싱글톤임
			return;
		}
		if (language.equals(Language.ENGLISH)) {
			this.dictionary = dictionaryFactory.getEnglishDictionary();
			return;
		}
		throw new IllegalArgumentException("지원하지 않는 언어입니다.");
	}

	public void hello() {
		if (dictionary == null) {
			throw new IllegalStateException("사전을 먼저 선택해주세요.");
		}
		dictionary.hello();
	}

	public void bye() {
		if (dictionary == null) {
			throw new IllegalStateException("사전을 먼저 선택해주세요.");
		}
		dictionary.bye();
	}
}
```

필요한 `자원의 팩터리`를 주입받아서 쓴다는 얘기는 해당 자원이 가변적이라는 얘기일까? 하나의 자원의 하위 객체들이 너무너무 많아서 팩토리도 여러 개인 경우에 사용하는 방법이지 않을까..?

- static하게 모든 곳에서 사용하는 객체의 의존성을 갈아끼우는 방법은 어떨까?

```java
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class MutableBot {
	private static final MutableBot INSTANCE = new MutableBot();

	private static Dictionary dictionary;

	public static MutableBot getInstance() {
		return INSTANCE;
	}

	public static void changeDictionary(Dictionary dictionary) {
		MutableBot.dictionary = dictionary;
	}

	public void sayHello() {
		dictionary.hello();
	}

	public void sayBye() {
		dictionary.bye();
	}
}
```

일단 딱 봐도 싸늘하다.

싱글 스레드에서는 상관 없을 것 같은데, 멀티 스레딩 환경에서는 동시적으로 사용해야하는 흐름에서 그 자원이 변경되면 문제가 생길 것 같다 → 의존성 주입을 통해서 필요한 인스턴스를 그때그때 생성해서 사용하는 방식을 써야하는 이유이지 않을까?

---

```java
public class InjectDependencyTest {

	@Test
	void 의존성_주입() {
		/*--------------------------------생성자 주입---------------------------------------------*/

		ConstructorBot constructorKoreanBot = new ConstructorBot(KoreanDictionary.getInstance());
		ConstructorBot constructorEnglishBot = new ConstructorBot(EnglishDictionary.getInstance());
		ConstructorBot constructorBotSpanishBot = new ConstructorBot(new CreatableSpanishDictionary());

		constructorKoreanBot.sayHello();
		constructorEnglishBot.sayHello();
		constructorBotSpanishBot.sayHello();

		/*--------------------------------정적 팩터리---------------------------------------------*/

		FactoryBot factoryKoreanBot = FactoryBot.of(Language.KOREAN);
		FactoryBot factoryEnglishBot = FactoryBot.of(Language.ENGLISH);
		FactoryBot factoryBotSpanishBot = FactoryBot.of(Language.SPANISH);

		factoryKoreanBot.sayHello();
		factoryEnglishBot.sayHello();
		factoryBotSpanishBot.sayHello();

		/*--------------------------------빌더 팩터리---------------------------------------------*/

		BuilderBot builderKoreanBot = BuilderBot.builder().dictionary(KoreanDictionary.getInstance()).build();
		BuilderBot builderEnglishBot = BuilderBot.builder().dictionary(EnglishDictionary.getInstance()).build();
		BuilderBot builderBotSpanishBot = BuilderBot.builder().dictionary(new CreatableSpanishDictionary()).build();

		builderKoreanBot.sayHello();
		builderEnglishBot.sayHello();
		builderBotSpanishBot.sayHello();

		/*--------------------------------팩터리 주입---------------------------------------------*/

		FactoryInjectedBot factoryInjectedBot = new FactoryInjectedBot(DictionaryFactory.getInstance());

		assertThrows(IllegalStateException.class, factoryInjectedBot::hello); // 사전을 선택하지 않았음

		factoryInjectedBot.changeDictionaryByFactory(Language.KOREAN);
		factoryInjectedBot.hello();
		factoryInjectedBot.changeDictionaryByFactory(Language.ENGLISH);
		factoryInjectedBot.hello();
		factoryInjectedBot.changeDictionaryByFactory(Language.SPANISH);
		factoryInjectedBot.hello();

		/*--------------------------------가변 static---------------------------------------------*/

		MutableBot mutableBot = MutableBot.getInstance();

		assertThrows(IllegalStateException.class, mutableBot::sayHello); // 사전을 선택하지 않았음

		MutableBot.changeDictionary(KoreanDictionary.getInstance());
		mutableBot.sayHello();
		MutableBot.changeDictionary(EnglishDictionary.getInstance());
		mutableBot.sayHello();
		MutableBot.changeDictionary(new CreatableSpanishDictionary());
		mutableBot.sayHello();
	}
}
```
