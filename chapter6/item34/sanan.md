## 핵심어

- 자바의 열거형(Enum) 그리고 특성
    - 열거 타입 자체는 하나의 클래스다.
    - 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static field로 공개한다(싱글턴).
    - 밖에서 열거타입을 생성하는 생성자를 공개하지 않으므로 사실상 final이다. ← 통제 인스턴스
    - 위 요소들 덕분에 컴파일 타임시의 타입 안전성을 제공한다.

---

## 요약

- 컴파일 타임에 다 알 수 있는 상수의 집합이라면 열거 타입을 사용하라.
- 일반 클래스(구 자바)에 상수를 선언하는 것은 좋지 않다.
→ 타입 안전 X, 표현력 구림
- 열거 타입에 메서드나 필드를 추가한다?
    
    결국 열거형도 하나의 클래스이므로, 별도의 메서드로 작성하는 것보다 열거형 자체에서 작성하는 것이 나을 수 있다.
    
- 추상 메서드를 선언하여 상수별 메서드를 재정의하게끔 만들 수 있다.
- toString이 반환하는 문자열을 그에 해당하는 열거 타입 상수로 변환하는 fromString 메서드도 제공하는 것을 고려해보자.
    
    ```java
    private static final Map String, Operation> stringToEnum =
    	Stream.of(values()).collect // this.values를 stream화 -> collect로 collection으로 변경
    	toMap (Object::toString, e -> e)); // Map으로 반환 -> K에는 각 상수의 toString, V에는 상수의 열거 타입을 엔트리로 지정
    
    //지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
    public static Optional<Operation> fromString (String symbol) {
    	return Optional.ofNullable(stringToEnum.get(symbol));
    // 해당 symbol(열거 -> toString)에 해당하는 열거 타입을 Optional로 반환
    }
    ```
    
- 상수별로 메서드가 다르게 동작해야한다면 상수별 메서드를 구현하든, 전략 열거 타입 패턴을 사용하든 하는 방법이 있다.

---

## 내 생각

실제로 이번에 알림 서비스를 구현하면서, 각 알림의 타입(상수)과 그에 따른 알림 포매팅을 작성하기 위해 이 아이템에서 제시하는 부분들에 대해 고민해본 적이 있었다. 프론트단에서 이 포맷을 가지게 되는 경우, 상수가 변경될 때 어플리케이션 특성상 ‘업데이트’를 모든 사용자가 하게 되므로 적절치 않다고 생각했고, 백엔드에서 변경사항을 가지고 있는게 좋다고 생각했다.

NoticeType이라는 열거형 클래스에 각 알림별 기본 제목, 내용에 대한 포매팅을 가지게 하여 특정 매개변수를 통해 프론트엔드에서 직접 딥링크(앱으로 이동하게끔 하는 링크)를 설정할 수 있도록 약속된 포맷으로 응답하도록 구현했었고, 직관적으로 나쁘지 않았던 코드였던 것 같다.

---

## 예제 코드

- 내가 작성했던 알림 타입 열거형 클래스

```java
@Getter
public enum NoticeType {
	ANNOUNCEMENT("%s", "%s"), // 공지사항의 경우 별도로 제목을 설정하여 사용합니다.
	DIARY_NOTE_FROM_TO("새 일기", "%s님이 %s에 일기를 남겼습니다."),
	DIARY_MEMBER_FROM_TO("새 멤버", "%s님이 %s에 가입하셨습니다."),
	NOTE_LIKE_FROM_TO("일기 좋아요", "%s님이 회원님의 %s를 좋아합니다."),
	FOLLOW_CREATE_FROM("새 팔로우", "%s님이 회원님을 팔로우했습니다."),
	;

	private final String title;
	private final String contentFormat;
	private final String navigatablePlaceholder = "{%s_%d}";
	private final String plainPlaceholder = "%s";

	NoticeType(String title, String contentFormat) {
		this.title = title;
		this.contentFormat = contentFormat;
	}

	/**
	 *
	 * @param fromName
	 * @param fromId
	 * @param toName
	 * @param toId
	 * @return
	 */

	public String createNavigatableNameFormattedContent(String fromName, Long fromId, String toName, Long toId) {
		ifTrue(this.equals(ANNOUNCEMENT), new DomainException(HttpStatus.BAD_REQUEST, "공지사항의 내용은 포매팅할 수 없습니다."));
		String fromPlaceholder = String.format(navigatablePlaceholder, fromName, fromId);
		String toPlaceholder = String.format(navigatablePlaceholder, toName, toId);
		return String.format(contentFormat, fromPlaceholder, toPlaceholder);
	}

	public String createPlainNameFormattedContent(String fromName, String toName) {
		ifTrue(this.equals(ANNOUNCEMENT), new DomainException(HttpStatus.BAD_REQUEST, "공지사항의 내용은 포매팅할 수 없습니다."));
		String fromPlaceholder = String.format(plainPlaceholder, fromName);
		String toPlaceholder = String.format(plainPlaceholder, toName);
		return String.format(contentFormat, fromPlaceholder, toPlaceholder);
	}
}
```
