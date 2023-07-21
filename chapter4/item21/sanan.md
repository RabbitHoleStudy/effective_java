## 핵심어

- **디폴트 메서드**
    
    자바 8 이후부터 인터페이스는 **default 키워드**를 사용해 디폴트 메서드를 포함할 수 있게 되었다. 이는 기존 인터페이스를 변경하지 않고도 새로운 메서드를 추가할 수 있게 해주는 특징인데, **인터페이스를 구현하는 클래스에서 오버라이드할 수 있지만, 그렇지 않은 경우에는 디폴트 메서드의 구현이 사용**된다.
    

---

## 요약

- 불변식을 해치지 않는 디폴트 메서드를 작성하기는 어렵다.
- 컴파일이 되더라도 (디폴트 메서드로 인해) 런타임에서 오류가 날 수 있다.
- 그러니 세심하게 주의를 기울이고, 배포 전에 인터페이스의 결함을 찾아내도록 노력하라.

---

## 내 생각

결국 추상 클래스의 일반 메서드든, 디폴트 메서드든 갖게 되는 그 문제점이 유사할 것 같다. 당연히 주의해야하는 부분이지만 굳이 챕터를 두어서 얘기를 한 것을 생각해보면, 내 생각에는 **‘디폴트 메서드를 막 작성하지 말고, 만약 작성하게 된다면 그게 정말 default한 것인지 고려해봐라 + 배포 전에 디폴트 메서드가 들어가 있는 인터페이스에 대해 이리저리 구현체를 작성해봐라’**의 뜻인 것 같다.

---

## 예제 코드

- 디폴트 메서드를 갖는 인터페이스와 여러 구현

```java
public interface Walking {

	default void lookAhead() {
		System.out.println("앞을 보기!");
	}

	default void balance() {
		System.out.println("중심 잡기!");
	}

	void walk();
}
/*-----------------------------------------------------------*/
public class StraightWalking implements Walking {
	@Override
	public void walk() {
		balance();
		lookAhead();
		moveForward();
		balance();
		moveForward();
	}

	private void moveForward() {
		System.out.println("앞으로 걷기!");
	}
}
/*-----------------------------------------------------------*/
public class PalzaWalking implements Walking {
	@Override
	public void walk() {
		balance();
		lookAhead();
		moveLeft();
		balance();
		moveRight();
	}

	private void moveLeft() {
		System.out.println("왼쪽으로 갔다가 ~ ");
	}

	private void moveRight() {
		System.out.println("오른쪽으로 갔다가 ~ ");
	}
}
```
