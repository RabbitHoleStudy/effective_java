## 핵심어

- 태그 클래스(Tagged Class)
    
    내부에 있는 태그에 따라 가변적으로 대응하는 클래스.
    
- 클래스 계층구조(Class Hierarchy)
    
    클래스 간의 관계를 나타낼 때 사용하며, 특히 상속 관계를 표현한다.
    클래스 계층구조에서는 상위 클래스(또는 부모 클래스, 슈퍼 클래스라고도 함)가 하위 클래스(또는 자식 클래스, 서브 클래스라고도 함)에게 일부 특성과 메서드를 상속하게 된다.
    

---

## 요약

- 여러 상태를 담고자 하는 클래스(태그 클래스)는 그냥 구린 것이니 무조건 지양하라.
→ 대신 클래스의 계층 구조를 두어서 구현해보자.

---

## 내 생각

- 가장 상위의 추상 클래스 혹은 인터페이스로 하위 클래스로 분기점을 없애는 것이 포인트인 것 같다.

---

## 예제 코드

- (안티패턴) 구린 태그 클래스 구현하기

```java
	public class CellPhone {

	private final DeviceType deviceType;
	private final String operatingSystem;
	private Integer foldingLifeSpan;

	public CellPhone() {
		deviceType = DeviceType.FLAT;
		this.operatingSystem = "iOS";
	}

	public CellPhone(Integer foldingLifeSpan) {
		deviceType = DeviceType.FOLDABLE;
		this.operatingSystem = "Samsung";
		this.foldingLifeSpan = foldingLifeSpan;
	}

	public void closePhone() {
		switch (deviceType) {
			case FOLDABLE:
				System.out.println("폴딩폰을 접습니다.");
				foldingLifeSpan--;
				if (foldingLifeSpan == 0) {
					System.out.println("뽀각");
				}
				break;
			case FLAT:
				System.out.println("터치폰을 끕니다.");
				break;
		}
	}

	public void forcePowerOff() throws InterruptedException {
		switch (operatingSystem) {
			case "Samsung":
				System.out.println("전원 버튼을 꾹 누릅니다.");
				Thread.sleep(1000 * 10);
				System.out.println("전원이 꺼집니다.");
				break;
			case "iOS":
				System.out.println("전원 버튼과 볼륨 버튼을 꾹 누릅니다.");
				Thread.sleep(1000 * 20);
				System.out.println("전원이 꺼집니다.");
				break;
		}
	}
}
```

- 태그 클래스를 추상 클래스를 상속하게하기

```java
public abstract class AbstractCellPhone {
	protected String operatingSystem;

	public abstract void closePhone();

	public abstract void forcePowerOff() throws InterruptedException;
}

public class FlatPhone extends AbstractCellPhone {
	public FlatPhone() {
		this.operatingSystem = "iOS";
	}

	@Override
	public void closePhone() {
		System.out.println("터치폰을 끕니다.");
	}

	@Override
	public void forcePowerOff() throws InterruptedException {
		System.out.println("전원 버튼과 볼륨 버튼을 꾹 누릅니다.");
		Thread.sleep(1000 * 20);
		System.out.println("전원이 꺼집니다.");
	}
}

public class FoldablePhone extends AbstractCellPhone {
	private Integer foldingLifeSpan;

	public FoldablePhone(Integer foldingLifeSpan) {
		this.operatingSystem = "Samsung";
		this.foldingLifeSpan = foldingLifeSpan;
	}

	@Override
	public void closePhone() {
		System.out.println("폴딩폰을 접습니다.");
		foldingLifeSpan--;
		if (foldingLifeSpan == 0) {
			System.out.println("뽀각");
		}
	}

	@Override
	public void forcePowerOff() throws InterruptedException {
		System.out.println("전원 버튼을 꾹 누릅니다.");
		Thread.sleep(1000 * 10);
		System.out.println("전원이 꺼집니다.");
	}

	public void alwaysOnDisplay() {
		System.out.println("AOD로 시간이 보인다");
	}
}
```
