### ordinal

> 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 메서드.
> 

대부분의 열거 타입 상수는 하나의 정숫값에 대응되기 때문에 편리한 메서드처럼 보일 수 있다.   
하지만 ordinal을 사용하면 상수 선언 순서를 바꾸기 힘들어진다. 또한, 값을 중간에 비워둘 수도 없어서 dummy 상수를 추가해야 할 수도 있다. 따라서 코드의 가독성와 실용성이 떨어지면서 유지보수가 힘들어진다.

**💡 열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고, 인스턴스 필드에 저장하자**

```java
public enum Ensemble {
	SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
	SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8);

//	public int numberOfMusicians() {
//		return ordinal() + 1; // ordinal은 0부터 시작
//	}
	private final int numberOfMusicians;

	Ensemble(int size) {
		this.numberOfMusicians = size;
	}

	public int numberOfMusicians() {
		return numberOfMusicians;
	}
}
```

<img width="872" alt="ordinal" src="https://github.com/TightJava/effective_java/assets/65652094/591a4552-efca-4bb1-b2c6-09d2b9b6caaa">

> ordinal 메서드는 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계된 메서드이고 대부분 프로그래머는 이 메서드를 쓸 일이 없다.
*- Enum API 문서*
> 

### 정리

Enum 또한 클래스이니 유지보수 힘들게 하는 ordinal 쓰지 말고 인스턴스 필드를 사용하자~
