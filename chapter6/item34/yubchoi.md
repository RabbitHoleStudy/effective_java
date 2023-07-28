## int enum pattern

자바에서 열거 타입을 지원하기 전, int enum pattern(정수 열거 패턴)을 사용했다.

```java
public static final int KOREA_SEOUL = 0;
public static final int KOREA_INCHEON = 1;
public static final int KOREA_BUSAN = 2;

public static final int JAPAN_TOKYO = 0;
public static final int JAPAN_OSAKA = 1;
public static final int JAPAN_OKINAWA = 2;
```

- **타입 안전성이 없다**. (KOREA_BUSAN == JAPAN_OKINAWA)가 true를 반환한다.
- 자바가 int enum pattern을 위한 별도 namespace를 지원하지 않기 때문에 접두어를 써서 **이름 충돌**을 방지한다. 예를 들어, 원소 수은과 행성 수성은 ELEMENT_MERCURY와 PLANET_MERCURY로 지어 구분한다.
- 평범한 상수를 나열한 거라 그 값이 클라이언트 파일에 그대로 새겨진다. **상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일해야 한다.**
- **디버깅이 어렵다**. 값을 출력하거나 디버거로 살펴보면 의미 없는 숫자로만 보인다.

## string enum pattern
정수 대신 문자열 상수를 사용하는 string enum pattern은 더 안 좋다.
**상수의 의미를 출력할 수 있다**는 장점이 있지만, 문자열을 하드코딩하다가 오타가 있으면 바로 **런타임 버그**가 생긴다. 문자열 비교에 따른 **성능 저하**도 있다.

## enum type
> 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입
> 

```java
public enum KOREA { SEOUL, INCHEON, BUSAN }
public enum JAPAN { TOKYO, OSAKA, OKINAWA}
```

**열거 타입은 클래스이다**. 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다. 하지만 외부에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final이다.

### 열거타입의 장점

- 인스턴스 통제된다.    
    열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재하기 때문이다.   
    싱글턴: 원소가 하나뿐인 열거 타입   
    => 열거타입: 싱글턴을 일반화한 형태   
- 컴파일타임 타입 안전성을 제공한다.   
    KOREA 열거 타입을 매개변수로 받는 메서드에 JAPAN 열거 타입 변수를 넘겨주면 컴파일 오류가 난다.    
- 각자의 namespace가 있다.   
    이름이 같은 상수도 접두어 없이 공존할 수 있다.   
- 열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 된다.   
    공개되는 것이 오직 필드의 이름뿐이라, 정수 열거 패턴과 달리 상수 값이 클라이언트로 컴파일되어 각인되지 않기 때문이다.   
- 열거 타입의 `toString` 메서드는 출력하기에 적합한 문자열인 상수 이름을 반환한다.
    - 예제 코드
        
        ```java
        public class EnumType {
        
        	public enum Country {
        		KOREA, JAPAN, USA, CANADA
        	}
        
        	public static void main(String[] args) {
        		System.out.println(Country.USA);
        
        //		System.out.println(Country2.CHINA);   // 컴파일 오류
        
        		Country newCountry = Country.CANADA;    // 새로운 상수를 추가하거나 순서를 바꿔도 재컴파일할 필요가 없음
        		System.out.println(newCountry);         // // toString 메서드가 자동으로 생성되어 "CANADA" 출력
        	}
        }
        ```
        
- 열거 타입 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드 `values`를 제공한다.   
    값들은 선언된 순서로 저장된다.  
- 열거 타입에 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다.   
    각 상수와 연관된 데이터를 해당 상수 자체에 내재시키고 싶을 경우 도움이 된다. 예를 들어 Apple과 Orange 타입에 과일의 색을 알려주거나 과일의 이미지를 반환하는 메서드를 추가할 수 있다.
    열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.   
    열거 타입을 선언한 클래스 혹은 그 패키지에서만 유용한 기능은 `private`이나 `package-private` 메서드로 구현한다.
    - 데이터와 메서드를 갖는 열거 타입 예제 코드
        
        ```java
        public enum Planet {
        	MERCURY(3.302e+23, 2.439e6), // 괄호 안은 생성자에 넘겨지는 매개변수이다.
        	VENUS(4.869e+24, 6.052e6),
        	EARTH(5.975e+24, 6.378e6),
        	MARS(6.419e+23, 3.393e6);
        
        	// 열거 타입은 근본적으로 불변이라 모든 필드는 final
        	// 필드를 public으로 선언해도 되지만, private으로 두고 별도의 public 접근자 메서드를 두는 게 낫다.
        	private static final double G = 6.67300E-11;
        	private final double mass;    // 질량
        	private final double radius;    // 반지름
        	private final double surfaceGravity;    // 표면중력
        
        	Planet(double mass, double radius) {
        		this.mass = mass;
        		this.radius = radius;
        		this.surfaceGravity = G * mass / (radius * radius);
        	}
        
        	public double surfaceWeight(double mass) {
        		return this.mass * this.surfaceGravity;
        	}
        }
        ```
        

### constant-specific method implementation

상수별로 다르게 동작하는 코드의 경우 switch문이 아닌 상수별 메서드 구현(constant-specific method implementation)을 사용하자.   
switch문의 경우 깨지기 쉬운 코드가 된다. 새로운 상수를 추가하면 해당 case문도 추가해야 하고 혹시 깜빡한다면 런타임 오류가 발생한다.    
다음은 상수별 클래스 body와 데이터를 사용한 열거타입이다.

```java
public enum Operation {
	PLUS("+") {
		public double apply(double x, double y) { return x + y; }
	},
	MINUS("-") {
		public double apply(double x, double y) { return x - y; }
	},
	TIMES("*") {
		public double apply(double x, double y) { return x * y; }
	},
	DIVIDE("/") {
		public double apply(double x, double y) { return x / y; }
	};

	private final String symbol;

	Operation(String symbol) {
		this.symbol = symbol;
	}

	@Override
	public String toString() {
		return symbol;
	}

	public abstract double apply(double x, double y);
}

/* 실행문 */
public class OperationTest {
	public static void main(String[] args) {
		double x = 4.0;
		double y = 2.0;
		for (Operation op : Operation.values()) {
			System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
		}
	}
}

/* 결과 */
4.000000 + 2.000000 = 6.000000
4.000000 - 2.000000 = 2.000000
4.000000 * 2.000000 = 8.000000
4.000000 / 2.000000 = 2.000000
```

- 열거 타입에 apply라는 추상 메서드를 선언하고 각 상수별 클래스 body에서 자신에 맞게 재정의한다.
- symbol이라는 상수별 데이터를 추가하고, toString을 재정의해 출력하기에 적합한 연산 기호를 반환한다.

toString을 재정의할 경우, toString이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드도 함께 제공해보자.

- 열거 타입용 fromString 메서드 구현
    
    ```java
    private static final Map<String, Operation> stringToEnum
    			= Stream.of(values()).collect(toMap(Object::toString, e -> e));
    
    // 지정한 문자열에 해당하는 Operation이 존재할 경우 반환한다.
    public static Operation fromString(String symbol) {
    	return stringToEnum.get(symbol);
    }
    ```
    
    - Operation 상수가 stringToEnum 맵에 추가되는 시점은 열거 타입 상수 생성 후 정적 필드가 초기화될 때다.

### 전략 열거 타입
상수별 메서드 구현의 경우, 열거 타입 상수끼리 코드를 공유하기 어렵다. 그럴 땐 전략 열거 타입을 사용하자.   
- 요금 계산 전략이 다르게 적용되는 피크 시간과 비피크 시간을 나타내는 FareType 전략 열거 타입
    
    ```java
    public enum TransportationMode {
    	BUS(FareType.REGULAR),
    	TRAIN(FareType.REGULAR),
    	TAXI(FareType.PREMIUM);
    
    	private final FareType fareType;
    
    	TransportationMode(FareType fareType) { this.fareType = fareType; }
    
    	int calculateFare(int distanceTravelled) {
    		return fareType.calculateFare(distanceTravelled);
    	}
    
    	// 전략 열거 타입
    	private enum FareType {
    		**REGULAR {
    			int extraCharge(int distance) {
    				return distance <= STANDARD_DISTANCE ? 0 :
    						(distance - STANDARD_DISTANCE) * 2;
    			}
    		},
    		PREMIUM {
    			int extraCharge(int distance) {
    				return distance * 3;
    			}
    		};**
    
    		abstract int extraCharge(int distance);
    		private static final int STANDARD_DISTANCE = 5;
    
    		int calculateFare(int distance) {
    			int baseFare = distance * 10;
    			return baseFare + extraCharge(distance);
    		}
    	}
    
    	public static void main(String[] args) {
    		int distanceTravelled = 10;
    
    		// 각 교통수단 별로 요금 계산
    		for (TransportationMode mode : TransportationMode.values()) {
    			System.out.printf("%dkm를 이동하는데 필요한 %s의 요금은 %d입니다.\n",
    					distanceTravelled, mode, mode.calculateFare(distanceTravelled));
    		}
    	}
    }
    
    /* 결과 */
    10km를 이동하는데 필요한 BUS의 요금은 110입니다.
    10km를 이동하는데 필요한 TRAIN의 요금은 110입니다.
    10km를 이동하는데 필요한 TAXI의 요금은 130입니다.
    ```
    

## switch문이 적합한 경우: 기존 열거 타입에 상수별 동작을 혼합해 넣을 때
위에서 언급했듯이 switch문은 열거 타입의 상수별 동작을 구현하는 데 적합하지 않다. 하지만 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch문이 적합할 수 있다.
- 서드파티에서 가져온 Operation 열거 타입이 있는데, 각 연산의 반대 연산을 반환하는 메서드가 필요한 경우
    
    ```java
    public static Operation inverse(Operation op) {
    	switch (op) {
    		case PLUS: return Operation.MINUS;
    		case MINUS: return Operation.PLUS;
    		case TIMES: return Operation.DIVIDE;
    		case DIVIDE: return Operation.TIMES;
    		default: throw new AssertionError("알 수 없는 연산: " + op);
    	}
    }
    ```
    
**💡 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.**

## 정리

- 자바의 int enum pattern과 string enum pattern에 비해 enum type은 타입 안전성을 제공하며, 상수 값의 변경에 따른 재컴파일이 필요 없고, 각 상수는 고유의 namespace를 가진다.
- 상수별로 동작이 다르다면 상수별 메서드 구현을 사용하자.
- 상수 일부가 같은 코드를 공유한다면 전략 열거 타입 패턴을 사용하자.
- 기존 열거 타입에 상수별 동작을 추가해야 할 때는 switch문이 좋을 수 있다.
- 컴파일타임에 알 수 있는 상수 집합은 열거 타입을 사용하자.
