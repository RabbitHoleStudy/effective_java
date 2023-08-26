
# 요약

- 숙련된 프로그래머는 더 많은 코드를 재사용한다.
- 웬만하면 Exception들도, 재사용하자, 아래에서 잘 쓰이는 6가지를 소개한다.
- 재사용하지 말아야할 것도 있다.

---

# 재사용되는 예외 TOP6

## IllegalArgumentException

- 전달인자로 부적절한 값을 받을 때 던지는 예외
- 반복횟수를 지정하는 매개변수에 음수가 나올때

## IllegalStateException

- 대상 객체의 상태가 호출된 메서드를 수행하기 적합하지 않을 경우
- 초기화가 되지 않은 객체를 사용하려 할 때

## NullPointerException

- null 값을 허용하지 않는 메서드에 null 을 건네면 관례상 이것을 쓴다
- IllegalStateException 이 아닌 NPE

## IndexOutOfBoundsException

- 시퀀스의 허용 범위를 넘을때
- IllegalArguemtException이 아닌 IndexOutOfBoundsException

## ConcurrentModificationException

- 단일 스레드에서 사용하려고 설계한 객체를 여러 스레드가 동시에 수정하려 할 때
- 동시 수정을 확실히 검출할 방법은 없으나, 가능성을 알려주는 명세 정도로 사용

## UnsupportedOperationException

- 클라이언트가 요청한 동작을 대상 객체가 지원하지 않을경우
- 보통은 구현하려는 인터페이스의 메서드 일부를 구현할 수 없을 때 사용
- 원소를 넣을 수 만 있는 List 구현체에 remove를 호출할 때

---

# 재사용하지 말아야 할 클래스

- Exception, RuntimeException, Throwable, Error
- 추상클래스라고 생각하자
- 다른 예외들의 상위 클래스이므로 안정적인 테스트가 불가능

⇒ Object를 직접 쓰는 꼴

- 웬만하면 나만의 예외를 새롭게 만들지 않는게 좋다.

---

# 기준이 애매할 때

# 예시

```java
static class Decks {
		List<Integer> cards = new ArrayList<>();

		public Decks() {
			for (int i = 1; i < 53; i++) {
				cards.add(i);
			}
		}

		public List<Integer> pickCards(int amount) throws IllegalArgumentException {
			if (amount < 0) {
				throw new IllegalArgumentException("잘 보고 넣어라잉");
			}

			if (amount > cards.size()) {
				throw new IllegalStateException("똑바로 넣어라잉");
			}

			List<Integer> pickedCards = new ArrayList<>();
			for (int i = 0; i < amount; i++) {
				pickedCards.add(cards.remove(0));
			}
			return pickedCards;
		}
	}
```

- pickCards 는 amount 만큼 Decks 에서 빼서, 뺀 카드들을 리턴

```java
if (amount == null) {
				throw new NullPointerException("amount is null");
}

if (amount < 0) {
				throw new IllegalArgumentException("ㅗ");
}

if (amount > cards.size()) {
				throw new IllegalStateException("ㅗㅗ");

}
```

- amount가 음수 중 어떤 수 였든 어차피 실패 ⇒ IllegalState
- amount가 cards.size() 보다 큰 경우에만 발생 ⇒ IllegalArgument
