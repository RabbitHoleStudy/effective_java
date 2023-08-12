# 아이템 58 - 전통적인 for 문 보다는 for-each 문을 사용하라
# 요약

- 가능한 곳에선 무조건 for-each문
- 원소의 삭제, 변형, 병렬반복인 경우에는 제외하자

# for-each 문

```java
List<String> list2 = List.of("z","y", "x", "w", "v", "u", "t", "s", "r");

for(Iterator<String> i = list.iterator(); i.hasNext();) {
			System.out.println("next = " + i.next());
	}
```

```java
for (String s : list) {
			System.out.println("next = " + s);
		}
```

---

## for 문에서 헷갈릴 수 있는 예시

### 예시1

```java
enum Suit {CLUB, DIAMOND, HEART, SPADE}
	enum Rank {ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING}

	static class Card {
		Suit suit;
		Rank rank;
		Card(Suit suit, Rank rank){
			this.suit = suit;
			this.rank = rank;
		}

		@Override
		public String toString() {
			return "Card{" +
					"suit=" + suit +
					", rank=" + rank +
					'}';
		}
	}
```

```java
List<Card> deck = new ArrayList<>();
		Collection<Suit> suits = Arrays.asList(Suit.values());
		Collection<Rank> ranks = Arrays.asList(Rank.values());

		for (Iterator<Suit> i = suits.iterator(); i.hasNext();)
			for (Iterator<Rank> j = ranks.iterator(); j.hasNext();)
				deck.add(new Card(i.next(), j.next()));

		for (Card card : deck) {
			System.out.println(card);
		}

//Exception in thread "main" java.util.NoSuchElementException
```

⇒ 무엇이 문제일까요?

- 정답
    - i.next( ) 가 너무 많이 불립니다.
    - new Card( i++, j++)
    
- 위와같이 헷갈릴 수 있다.

### 개선

- 개선 - for
    - suit 상태를 저장
    
    ```java
    for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
    			Suit suit = i.next();
    			for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); ) {
    				deck.add(new Card(suit, j.next()));
    			}
    		}
    ```
    
- 개선 - for-each
    
    ```java
    for (Suit suit :suits)
    			for (Rank rank : ranks)
    				deck.add(new Card(suit, rank));
    ```
    

---

### 예시 2

```java
enum Face {ONE, TWO, THREE, FOUR, FIVE, SIX}

Collection<Face> faces = EnumSet.allOf(Face.class);
		for (Iterator<Face> i = faces.iterator(); i.hasNext();)
			for (Iterator<Face> j = faces.iterator(); j.hasNext(); ) {
				System.out.println(i.next() + " " + j.next());
	}
```

- 정답
    
    ```java
    ONE ONE
    TWO TWO
    THREE THREE
    FOUR FOUR
    FIVE FIVE
    SIX SIX
    ```
    

### 개선

- 개선
    
    ```java
    for (Face face : faces) 
    			for (Face face2 : faces)
    				System.out.println(face + " " + face2);
    ```
    

---

# for-each 를 사용할 수 없는경우

### 파괴적인 필터링 (destructive filtering)

- 컬렉션을 순회하면서 선택된 원소를 제거해야 하는경우
    
    ```java
    for (Integer number : numbers) {
    			if (number % 2 == 0 )
    				numbers.remove(number);
    		}
    
    Exception in thread "main" java.util.ConcurrentModificationException
    ```
    
    - 개선 1 - for
        
        ```java
        for (Iterator<Integer> i = numbers.iterator(); i.hasNext();) {
        			Integer number = i.next();
        			if (number % 2 == 0)
        				i.remove();
        		}
        ```
        

- java 8 이후로는 Collection removeIf 로 명시적 순회 파-괴 가능
    - 개선 2 forEach + stream
        
        ```java
        numbers.removeIf(i -> i % 2 == 0);
        ```
        

### 변형(transforming)

- 리스트나 배열을 순회하면서 그 원소의 값의 일부/전체 를 교체해야하는 경우
    
    ```java
    List<String> fruits = new ArrayList<>();
    		fruits.add("Apple");
    		fruits.add("Banana");
    		fruits.add("Cherry");
    		
    		
    		for (String fruit : fruits) {
    			fruit +="s";
    		}
    		// 바뀌지 않는다.
    
    		for (ListIterator<String> i = fruits.listIterator(); i.hasNext();) {
    			i.set(i.next() + "s");
    		}
    ```
    

### 병렬반복(parallel iteration)

- 여러 컬렉션을 병렬로 순회할때
    
    ```java
    private static void parallelForEach() {
    		List<Integer> numbers = new ArrayList<>();
    		numbers.add(1);
    		numbers.add(2);
    		numbers.add(3);
    
    		// 이전 요소에 의존하는 처리
    		int previousValue = 0;
    		for (int number : numbers) {
    			int newValue = number + previousValue;
    			System.out.println("Number: " + number + " | New Value: " + newValue);
    			previousValue = newValue;
    		}
    	}
    ```
    
    ⇒ 가능은 한데, 차라리 인덱스를 쓰는게 나은경우
    
- 스레드를 쓰는경우
    
    ```java
    private static void parallelForEachThread(){
    		List<Integer> numbers = new ArrayList<>();
    		numbers.add(1);
    		numbers.add(2);
    		numbers.add(3);
    
    		Runnable task = () -> {
    			for (int i = 0; i < numbers.size(); i++) {
    				int newValue = numbers.get(i) * 2;
    				numbers.set(i, newValue);
    				System.out.println("Thread " + Thread.currentThread().getId() +
    						" | Element at index " + i + " updated to " + newValue);
    			}
    		};
    
    		Thread thread1 = new Thread(task);
    		Thread thread2 = new Thread(task);
    
    		thread1.start();
    		thread2.start();
    	}
    ```
    
    ⇒ 실행마다 최종값이 달라짐
