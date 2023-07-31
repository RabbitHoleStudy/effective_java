# 메소드 시그니처를 신중히 설계하라

아이템 한 줄 요약: 잡다한 API 설계 꿀팁 모아놨다. 잘 써라.

### **메소드 이름을 신중히**

- **자바 메소드 명명 규칙(일부)**
    
    ```java
    Packages: 전체 소문자로 작성하며, 고유의 레벨이 있는 경우 점 (.)을 이용해 구분합니다. 대개 도메인 이름을 거꾸로 문자열로 사용합니다. 예: com.example.myapp
    Classes and Interfaces: 카멜 케이스(CamelCase)를 사용하며, 각 단어의 첫 글자는 대문자입니다. 예: String, MyClass, UserInterface.
    Methods and Variables: 카멜 케이스를 사용하되, 첫 글자는 소문자로 시작합니다. 동사가 앞에 오고, 명사가 뒤에 오는 것이 일반적입니다. 예: getAccount, processData, index.
    Constants: 모든 글자를 대문자로 작성하고, 각 단어 사이는 언더바로 구분합니다. 예: MAX_VALUE, USER_HOME_DIRECTORY.
    Generics: 한 글자 변수는 대문자를 사용합니다. T는 타입을, E는 요소를, K는 키를, V는 값을, S, U, V 등은 2번째, 3번째, 4번째 타입을 뜻합니다.
    ```
    

사실 어느 언어나 거의 비슷하다고 생각한다. 

아무튼간에 같은 패키지에 속한 다른 이름들과 일관성있게 작성해야 하는 것이 핵심이다.

### 편의 메소드는 적당히만

제목이 곧 내용이다. 편의 메소드를 너무 많이 만들면 이를 익히고 사용하고, 문서화하고 테스트하고 유지보수하는데 너무 많은 비용이 들 수 있다.

그렇기 때문에 확실한 이유가 있지 않다면 가급적 사용하지 않는 것이 좋다.

### 매개변수 목록은 짧게

매개변수는 4개 이하 정도로 짧게 유지하는 것이 좋다.

너무 당연한 얘기이다. 4개를 넘어가면 매개변수를 전부 기억하기가 어렵고, 같은 타입의 매개변수가 연달아 나오는 경우 실수가 나오기 쉽다.

- **매개변수 목록을 짧게 줄여주는 기술 세가지**
    1. 여러 메소드로 쪼갠다.
        
        잘못하면 메소드 수가 많아질 수 있지만, 직교성이 높아져 오히려 역설적으로 메소드 수를 줄여주는 방향이 될 수 있다.
        
        - **직교성이란?**
            
            수학에서 직교(orthogonal)은 두 벡터의 내적이 0이라는 의미이고, 곧 서로 영향을 주는 성분이 없다는 의미이다. (독립)
            
            소프트웨어공학에서는 이를 기능을 원자적으로 쪼개어 제공해준다는 의미로 사용한다.
            
        
    2. 매개변수 여러 개를 묶어주는 헬퍼 클래스를 만든다.
        - **헬퍼클래스**란?
            
            **헬퍼 클래스**는 이름에서 짐작할 수 있듯이 "도우미"의 역할을 합니다. 대개는 상태 없이 정적 메서드만 가지며, 커몬 (공통적으로) 사용되는 로직을 캡슐화하는데 사용됩니다. 예를 들어, 정렬이나 수치 계산, 파일 I/O와 같은 작업을 수행하는 메서드들이 있는 클래스가 헬퍼 클래스가 될 수 있습니다.
            
            ```java
            public class CardGame {
            
                public static class Card {
                    private final int rank;
                    private final String suit;
            
                    public Card(int rank, String suit) {
                        this.rank = rank;
                        this.suit = suit;
                    }
            
                    // getters
                    public int getRank() {
                        return this.rank;
                    }
            
                    public String getSuit() {
                        return this.suit;
                    }
                }
            
                public void playCard(Card card) {
                    // Card playing logic
                    System.out.println("Playing card: " + card.getRank() + " of " + card.getSuit());
                }
            
                public static void main(String[] args) {
                    CardGame game = new CardGame();
            
                    // Using Card helper class, we only need to pass one argument instead of two
                    game.playCard(new Card(10, "Hearts"));
                }
            }
            ```
            
        
    3. 1, 2번을 혼합한 빌더 패턴
        
        매개변수가 많고, 일부는 객체 생성 시점에 꼭 필요하지 않은 경우일 때 사용하기 좋다.
        
        - **빌더 패턴 예제**
            
            ```java
            public class Pizza {
                private String cheese;
                private String topping;
                private String sauce;
            
                // Constructor
                public Pizza() {}
            
                // Setter Methods
                public Pizza setCheese(String cheese) {
                    this.cheese = cheese;
                    return this;
                }
            
                public Pizza setTopping(String topping) {
                    this.topping = topping;
                    return this;
                }
            
                public Pizza setSauce(String sauce) {
                    this.sauce = sauce;
                    return this;
                }
            
                public void makePizza() {
                    // Assuming here it makes pizza and validates the parameters.
                    System.out.println("Making pizza with " + this.cheese + ", " + this.topping + " and " + this.sauce);
                }
            
            	public static void main(String[] args) {
            	    Pizza pizza = new Pizza();
            	    pizza.setCheese("Mozzarella")
            	        .setTopping("Olives")
            	        .setSauce("Tomato")
            	        .makePizza();
            	}
            }
            ```
            
        

### 매개변수 타입은 클래스보다는 인터페이스로

인터페이스 대신 클래스를 매개변수로 사용하게 되면, 클라이언트에게 특정 구현체만 사용하도록 제한하게 된다.

클라이언트에서 사용하는 객체가 우리가 작성한 메소드의 매개변수와 조금이라도 다르다면, 명시한 특정 구현체로 객체를 옮겨담는 비용을 치뤄야하므로 비효율적일 수 있다.

그래서 자바 관련 라이브러리를 보더라도 **HashMap**이나 **ArrayList**를 직접 넘기는 경우는 절대 없다. **Map**과 **List** 인터페이스를 매개변수로 사용한다.

### boolean보다는 원소 2개짜리 enum을

물론 메소드 이름상 boolean을 받아야 의미가 더 명확해지는 경우는 예외이다.

(isValid()같은 메소드에서 boolean이 아닌 enum을 반환하는 건 말이 안 되죵?)

하지만 그 외의 경우에는 열거타입을 사용해야 코드가 읽기 쉬어지고, 나중에 선택지를 추가하기도 쉬워진다.

```java
public enum TemperatureScale { FAHRENHEIT, CELSIUS }
```

이런식으로 enum을 정의한다면 

`Thermometer.newlnstance(true)`

보다

`Thermometer.newlnstance(TemperatureScale.CELSIUS)`

이게 훨씬 의미가 명확하게 전달될 것이다.

또한 나중에 캘빈온도를 지원해야한다면 추가로 다른 메소드를 만들필요 없이 enum에 KELVIN만 추가하면 되므로 확장성이 좋아진다.
