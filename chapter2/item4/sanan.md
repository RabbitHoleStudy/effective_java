## 핵심어

---

## 요약

인스턴스화가 필요 없는 클래스 등은 생성자를 private으로 설정해서 인스턴스화를 막아라.

추상 클래스여도 결국 하위 클래스에서 생성이 가능하므로 잘 막아라.

---

## 내 생각

맞는 말이다. 전에 작성해놓은 static한 클래스들에 생성자가 작동하는지 찾아봐야겠다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d7b08c68-0498-4662-8ddc-0f85641efecf/Untitled.png)

까비 프로젝트에서 사용하는 ExceptionUtil이 인스턴스화가 된다..😱 막아놔야겠다.

---

## 예제 코드

```java
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class PrivateConstructorExample {
	private static final String name = "개인정보입니다만?";
	private static final Integer age = 7;

	public static String getName() {
		return name;
	}
}

/*--------------------------------------------------------*/

@Test
	void 생성자_막기() {
		// PrivateConstructorExample privateConstructorExample = new PrivateConstructorExample(); <- 안 됨
		System.out.println("PrivateConstructorExample.getName() = " + PrivateConstructorExample.getName());
		// "개인정보입니다만?"
	}
```
