## 핵심어

- 캡슐화(은닉)
캡슐화는 데이터와 해당 데이터에서 작동하는 메서드를 클래스처럼 하나의 단위로 묶는 것을 말한다. 우리는 종종 객체의 내부 표현이나 상태를 외부에 대해 숨길 때 이 개념을 사용하는데, 이를 정보 은닉이라고 한다.
- 방어적 복사
내부 프로퍼티의 값을 복사해서 새 인스턴스로 반환하는 방법.
기준이 되는 인스턴스의 값의 변화를 방어하기 위한 복사라는 의미에서 방어적이라고 하는 것 같다.

---

## 요약

- Dimension 공개 라이브러리 API의 예시(캡슐화가 깨진 가변필드)
→ 해당 객체의 프로퍼티를 참조하려할 때, 방어적 복사(가변필드이므로)가 일어나게 되고, 이에 따른 비용이 발생할 수 있다.
- **public 클래스는 절대 가변 필드를 노출해서는 안 된다 + 불변 필드여도 노출하지 마라.**

---

## 내 생각

- **getter를 사용하는 의미?**
결국 ‘의미를 담은 사용’이라는 측면이 지배적인 것 같다. 단순한 인스턴스의 프로퍼티에 대한 참조가 아닌, getter라는 메서드를 통한 일종의 래핑(원하는 값으로 반환하게끔 하는 것도 가능하니)을 통해, 내부의 프로퍼티를 은닉하고 외부에는 원하는 값에 대해서 호출할 수 있도록 추상화가 가능한 것 같다.
- **setter를 지양하고 별도의 이름으로 두고자 하는 의미와 getter의 상관성?**
setter를 지양하라는 의미는 오히려 관습처럼 get을 찾듯 set을 사용하게 되는 경우에 있어서 set을 남발하는 프로그래밍 습관을 제한하기 위함이 아닌가 싶다. 또한 getter와 setter가 공존하는 한 엄밀한 캡슐화는 깨지기 때문이다. 필요한 경우에 알맞은 의미로 사용될 수 있도록 setter의 이름을 명시하는 방식으로 어느 정도의 제약과 가독성을 챙길 수 있을 것 같다.

---

## 예제 코드

- Setter의 이름 명시

```java
// 까비의 Cabinet Domain의 세터 일부분
public void specifyLentType(LentType lentType) {
		this.lentType = lentType;
		ExceptionUtil.throwIfFalse(this.isValid(), new DomainException(INVALID_STATUS));
	}

	public void writeTitle(String title) {
		this.title = title;
		ExceptionUtil.throwIfFalse(this.isValid(), new DomainException(INVALID_STATUS));
	}

	public void coordinateGrid(Grid grid) {
		this.grid = grid;
		ExceptionUtil.throwIfFalse(this.isValid(), new DomainException(INVALID_STATUS));
	}
```

단순한 이름의 setter를 지양하고, 명확한 이름을 사용하려고 했었다. (~~DDD의 잔재..)~~
