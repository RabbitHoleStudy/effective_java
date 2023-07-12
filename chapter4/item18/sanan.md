## 핵심어

- Wrapper Class
- Decorator Pattern
- Delegation
- has-a, is-a 관계
구성요소로서 갖느냐(has-a),
하위 범주로서 해당 상위 클래스에 속하느냐(is-a)에 따라 다르다. 예로는 Car-Engine(has-a), Animal-Cat(is-a)라고 생각해볼 수 있다.

---

## 요약

- 다른 패키지의 구체 클래스를 상속하는 일은 위험하다 - 같은 프로그래머, 같은 패키지 내의 상속은 괜찮다.
→ 문서화가 잘 된 클래스라면 괜찮다.
- 상속은 캡슐화를 깨뜨린다.
→ 상위 클래스의 구현에 하위 클래스가 영향을 받고, 하위 클래스의 구현(특히 재정의)가 상위 클래스의 캡슐화를 망가뜨릴 수 있다.
- 상속 대신 새로운 클래스를 만들고 private으로 기존 클래스의 인스턴스를 참조하게끔 하는 컴포지션을 사용하라.
- 상속은 is-a 관계일 때만 하라.

---

## 내 생각

- 똑같은 기능(행동)으로 묶여야만하고, 그 변화가 상위 클래스에 일어나고, 그에 따라 마땅히 하위 클래스가 변해야 하는 관계에서만 상속을 써야 한다고 생각하면 될 것 같다.
- 캡슐화는 은닉화가 아니다.
→ 캡슐화는 프로그래머의 의도가 담긴 객체의 상태를 나타내는 속성(프로퍼티)과 행위를 수행하는 메서드로 구성된 하나의 논리적 묶음으로서의 특징이다.
→ 은닉은 캡슐화와 접근 제어자를 통해 따라오는 것이다.
- 상속이 캡슐화를 깬다는 말의 의미는 뭘까?
상속 자체가 캡슐화를 깬다기 보다는 망가뜨릴 수 있는 가능성을 제공한다.
→ 하위 클래스에서 상위 클래스의 메서드를 재정의 하는 경우에, 무슨 짓이든 할 수 있다. 부모에서 private으로 지정해놓은 프로퍼티들을 냅다 print로 다 찍어버린다든지.. 기존의 상위 클래스에서 의도한 바(속성과 행위의 캡슐화)를 망가뜨릴 수 있다는 것이다.
자바를 만든 제임스 고슬링은 extends 상속은 문제가 있으니 지양하고, implements 상속을 고려하라고 얘기했다.
참고 : https://unluckyjung.github.io/oop/2021/03/17/Inheritance-and-Encapsulation/
- Composition 자체가 합성이라는 뜻인데, 클래스를 포함관계(has-a)로 만들어서 사용한다는 것 자체가 의존성 주입과 같은 얘기라고 생각한다.

---

## 예제 코드

- has-a와 is-a

```java
@AllArgsConstructor
public class EngineBelongsToCar {
	public void start() {
		System.out.println("부릉부릉");
	}
}

@AllArgsConstructor
public class CarHasEngine {
	private EngineBelongsToCar engine;

	public void start() {
		engine.start();
	}
}
/*-----------------------------------------------------------------------------*/
@AllArgsConstructor
public class Animal {
	public void makeSound() {
		System.out.println("[뭔가 소리를 내고 있음]");
	}
}

@AllArgsConstructor
public class CatIsAnimal extends Animal {
	@Override
	public void makeSound() {
		System.out.println("응애옹");
	}
}
```

- extends 상속과 구현에 따른 캡슐화 깨짐

```java
@AllArgsConstructor
public class ParentCapsule {
	private final String mySecretCanBeToldOnlyToParents;
	private final Integer weightCanBeToldOnlyToMyself;
	private final String nameCanBeTold;

	public String getName() {
		return nameCanBeTold;
	}

	protected String getMySecretCanBeToldOnlyToParents() {
		return mySecretCanBeToldOnlyToParents;
	}

	protected Integer getWeightCanBeToldOnlyToMyself() {
		return weightCanBeToldOnlyToMyself;
	}

	public void introduce() {
		System.out.println("안녕하세요. 저는 " + nameCanBeTold + "입니다.");
	}
}
/*-----------------------------------------------------------------------------*/
public class ChildCapsule extends ParentCapsule {

	public ChildCapsule(String mySecretCanBeToldOnlyToParents, Integer weightCanBeToldOnlyToMyself, String nameCanBeTold) {
		super(mySecretCanBeToldOnlyToParents, weightCanBeToldOnlyToMyself, nameCanBeTold);
	}

	@Override
	public void introduce() {
		System.out.println("안녕하세요. 저는 " + getName() + "인데요,");
		System.out.println("저가 부모님한테만 말하는 비밀은 " + getMySecretCanBeToldOnlyToParents() + "이구요,");
		System.out.println("저만 알고 있는 제 몸무게는 " + getWeightCanBeToldOnlyToMyself() + "입니다.");
	}
}
/*-----------------------------------------------------------------------------*/
@Test
	void 캡슐화_테스트() {
		ParentCapsule parentCapsule = new ParentCapsule("티슈 세장씩 뽑아 씀", 63, "몽키 D 루피");
		parentCapsule.introduce(); // 안녕하세요. 저는 몽키 D 루피입니다.

		ChildCapsule childCapsule = new ChildCapsule("코노에서 노래부르다가 이빨 깨졌던 일", 130, "부모님 속도 모르는 자식");
		childCapsule.introduce(); //안녕하세요. 저는 부모님 속도 모르는 자식인데요, 저가 부모님한테만 말하는 비밀은 코노에서 노래부르다가 이빨 깨졌던 일이구요, 저만 알고 있는 제 몸무게는 130입니다.
	}
/*-----------------------------------------------------------------------------*/
@Test
	void 컴포지션_테스트() {
		CarHasEngine car = new CarHasEngine(new EngineBelongsToCar());
		car.start(); // 부릉부릉 from Engine
	}
```
