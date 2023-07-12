## 핵심어

---

## 요약

- 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지(자기사용) 문서로 남겨야 한다.
- 꼭 필요한 protected 멤버를 놓쳤다면 하위 클래스를 작성할 때 그 빈 자리가 확연히 드러난다. 거꾸로, 하위 클래스를 여러 개 만들 때까지 전혀 쓰이지 않는 protected 멤버는 사실 private이었어야 할 가능성이 크다.
- 상속용으로 설계한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증해야 한다.
- 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서 는 안 된다.
→ 상위 클래스의 생성자는 하위 클래스의 생성자가 인스턴스 필드를 초기화하기도 전에 해당 재정의 메서드를 호출하기 때문
- clone과 readObject 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.
→ clone과 readObject 메서드는 생성자와 비슷한 효과를 낸다(새로운 객체를 만든다).

---

## 내 생각

- private에 대해 항상 염두하자.
- extends 상속은 제약사항이나 고려해야할 점이 너무 많으니, 최대한 지양해야겠다(명확한 is-a 관계가 아니라면).
- 생성자에 쓸 데 없는 짓하지 말자.

---

## 예제 코드

- 상위 클래스의 생성자에서 재정의 가능한 메서드를 호출하고 ,이를 하위 클래스가 재정의 하는 경우 - 문제점

```java
@Getter
public class SuperClass {
	private final String name;

	public SuperClass(String name) {
		this.name = name;
		this.introduce();
	}

	public void introduce() {
		System.out.println("안녕하세요. 저는 " + name + "입니다.");
	}
}
/*-----------------------------------------------------------------------------*/
public class InferClass extends SuperClass {
	private final Integer age;

	public InferClass(String name, Integer age) {
		super(name);
		this.age = age;
	}

	@Override
	public void introduce() {
		System.out.println("안녕하세요. 저는 " + getName() + "인데요, 제 나이는 " + age + "입니다.");
	}
}
/*-----------------------------------------------------------------------------*/
@Test
	void 오버라이드_테스트() {
		InferClass inferClass = new InferClass("삼천갑자 동방삭", 180000);
		// 예상 : 안녕하세요. 저는 삼천갑자 동방삭인데요, 제 나이는 180000입니다.
		// 실제 : 안녕하세요. 저는 삼천갑자 동방삭인데요, 제 나이는 null입니다.
	}
```
