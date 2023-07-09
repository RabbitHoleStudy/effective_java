# 정적 팩터리 메소드 (static factory method)

---

# 요약

객체를 새로 생성해서 반환하거나,

기존에 있는 객체를 반환하도록 하는 메소드

메소드에 static 을 붙여서 사용하는 형태

- 생성자 처럼 사용할 수 있다.
- 객체를 생성하지 않고 함수를 호출 할 수 있다.

---

# 장점 & 설명

## 1. 생성자처럼 쓰면서 이름을 가질 수 있다.

- 이름을 가질 수 있다는 말은 객체의 특성을 설명할 수 있다는걸 뜻한다.

### 예시 BigInteger

```java
//BigInteger Class 

public BigInteger(int numBits, Random rnd) { ... }
public static BigInteger probablePrime(int bitLength, Random rnd) {...}
```

- BigInteger() 생성자 와 BigInteger.probablePrime()
    - [생성자] BigInteger(int numBits, Random rnd)
    - 함수에 대한 설명
        - numBits 로 비트 수를 입력 받고, Random  객체를 입력받아서 랜덤한 numBits에 해당하는 수 중, 10진수를 리턴한다
        - 예를들어 numBits = 3 일때 출력값은 3 비트 중 랜덤으로 나온다
        `000(0), 001(1), 010(2), 011(3), 111(4)`
        - [Static Factory method] BigInteger.probablePrime()
            - bitLength 로 비트 수를 입력받고, Random 객체를 입력받아서 랜덤한 bitLength 에 해당하는 수들 중 소수값을 랜덤으로 10진수 리턴한다.
            - 예를들어 bitLength = 5 일때
            `00011(3),00101(5)` 중 하나가 랜덤으로 나온다.
    - BigInteger.probablePrime() 상세 구현 설명
        
        ```java
        BigInteger(int, int, Random)
        
        //정적 메소드
        public static BigInteger probablePrime(int bitLength, Random rnd) {
                if (bitLength < 2)
                    throw new ArithmeticException("bitLength < 2");
        
                return (bitLength < SMALL_PRIME_THRESHOLD ?
                        smallPrime(bitLength, DEFAULT_PRIME_CERTAINTY, rnd) :
                        largePrime(bitLength, DEFAULT_PRIME_CERTAINTY, rnd));
            }
        
        // 생성자
        public BigInteger(int numBits, Random rnd) {
                byte[] magnitude = randomBits(numBits, rnd);
        
                try {
                    // stripLeadingZeroBytes() returns a zero length array if len == 0
                    this.mag = stripLeadingZeroBytes(magnitude, 0, magnitude.length);
        
                    if (this.mag.length == 0) {
                        this.signum = 0;
                    } else {
                        this.signum = 1;
                    }
                    if (mag.length >= MAX_MAG_LENGTH) {
                        checkRange();
                    }
                } finally {
                    Arrays.fill(magnitude, (byte)0);
                }
            }
        ```
        
        잔달받은 비트수에 해당하는, 랜덤한 소수를 생성하는 정적 팩터리 메소드 이다.
        
        - 함수 설명
            
            bitLength 를 입력하여 해당 비트에 맞는 랜덤한 소수값을 리턴해준다.
            ex) bitLength=3 이면, 5나 7이 랜덤으로 나온다 2^3 승에서 나올 수 있는 소수 5 혹은 7
            
            2보다 작을경우 Exception
            
            `bitLength` 가 
            `DEFAULT_PRIME_CERTAINTY` (100) 
            보다 작으면 smallPrime 
            크면 largePrime 함수에서 랜덤한 소수를 생성한다.
            
            그렇다면 이 소수는 어떻게 만들어지는걸까?
            
        - 그렇다면 largePrime 함수는?
            
            ![그만알아보자](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f24a3bfa-3228-4a1d-b930-3f350e114b0a/Untitled.png)
            
            그만알아보자
            
        
- 한 클래스에 시그너처(전달인자) 가 같은 생성자가 여러개 필요할 경우에 사용 가능
    - `BigInteger.probablePrime(int, Random)`  는 랜덤 소수 생성
    - `BigInteger(int, Random)` 는 랜덤한 10진수 반환 생성자

## 2. 호출 때마다 인스턴스를 새로 생성하지 않아도 된다.

### Boolean.valueOf()

```java
@IntrinsicCandidate // JIT 컴파일러 내장 후보로 등록 -> 하드웨어 아키텍쳐 명령어로 실행한다는 뜻 -> 대충 최적화
    public static Boolean valueOf(boolean b) { 
        return (b ? TRUE : FALSE);
    }

public final class Boolean implements java.io.Serializable,
                                      Comparable<Boolean>, Constable
{
    public static final Boolean TRUE = new Boolean(true);
    public static final Boolean FALSE = new Boolean(false);
... 생략
}
```

- Primitive Type → Wrapper class 로 반환해주는 클래스이다.
    - 이때 TRUE 와 FALSE 는 시스템상 인스턴스가 하나만 존재하도록 위처럼 구현되어있다.
- 위와 같은 형태를 인스턴스 통제(instance-controlled) 클래스 라고도 한다.
- 싱글턴 테크트리 혹은 인스턴스화 불가 테크트리로 갈 수 있다.
- 인스턴스가 하나이기때문에, 객체비교인 a==b 와 a.equals(b) 가 성립된다.
    - JAVA 에서 일반적으로 equals 는 객체의 값 비교
    - == 는 두 개체의 참조 비교 (메모리 위치 비교)

[정적 팩토리 객체 재사용(캐시) 예시코드](https://www.notion.so/fa367fcf580b4c6e813b6d486bec6b9c?pvs=21) 

## 3. 반환 타입의 하위 타입 객체를 반환할 수 있다

- 아래 예시 참조

[하위타입 반환 예제](https://www.notion.so/be87037df688497898b045f61e544a7d?pvs=21) 

## 4.입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다

```java
public class EnumSetExample {
	enum Color { RED, BLUE, GREEN}

	public static void main(String[] args) {
		EnumSet<Color> smallSet = EnumSet.of(Color.RED, Color.BLUE); // RegularEnumSet 반환
		//EnumSet<Color> largeSet = EnumSet.allOf(Color.class); // RegularEnumSet 반환 
//65개이상 JumboEnumSet - LONG

		System.out.println(smallSet.getClass().getSimpleName()); // RegularEnumSet 출력
	//	System.out.println(largeSet.getClass().getSimpleName()); // RegularEnumSet 출력
	}
}
public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2) {
        EnumSet<E> result = noneOf(e1.getDeclaringClass());
        result.add(e1);
        result.add(e2);
        return result;
    }
/*

	public static <E extends Enum<E>> EnumSet<E> allOf(Class<E> elementType) {
	        EnumSet<E> result = noneOf(elementType);
	        result.addAll();
	        return result;
	    }

*/
		public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
		        Enum<?>[] universe = getUniverse(elementType);
		        if (universe == null)
		            throw new ClassCastException(elementType + " not an enum");
		
		        if (universe.length <= 64)
		            return new RegularEnumSet<>(elementType, universe);
		        else
		            return new JumboEnumSet<>(elementType, universe);
		    }
		
		
```

- EnumSet
    - EnumSet.of → nonOf
    - EnumSet 클래스는 Enum 이 정의된게 64개 이상일경우 JumboEnumSet ( long type) 으로 생성
    - 64개 미만일경우 RegularEnumSet (int type) 으로 생성된다.
- 위와 같이 세부 구현사항을 몰라도 우리는 그냥 EnumSet 을 만들어주는대로 사용할 수 있다.
- 만약 세부구현이 바뀐다고해도, 클라이언트 코드는 그대로 EnumSet.of 로 사용할 수 있다.

## 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

## 단점

### 1. 상속을 하려면 public이나 protected 생성자 필요 
따라서 정적 팩터리 메서드만 있다면 상속 / 구현 불가능

### 2. 찾기 어려움 - 규약 지켜야함

- From : 매개변수 받아서 형변환 인스턴스 반환
- of : 매개변수 받아서 인스턴스 반환
- valueOf: from + of
- getInstance, instance : 매개변수로 명시한 인스턴스 반환 `같은 인스턴스임을 보장하지 않음`
- create, newInstance : 매번 새로운 인스턴스를 생성
- getType  : 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할때
    - FileStore fs = Files.getFileStore(path)
    - path 전달인자로 abstract FileStore 객체리턴 하지만 Files 라는  관리 객체에서 리턴
    ⇒ 더큰 개념에서 하위 개념이지만 포함은 아닐때 사용하는듯
    
    ```java
    public static FileStore getFileStore(Path path) throws IOException {
            return provider(path).getFileStore(path);
        }
    
    //
    public abstract class FileStore {
    
        /**
         * Initializes a new instance of this class.
         */
        protected FileStore() {
        }
    ```
    
- newType : 다른클래스에서 새로운 인스턴스 반환
- type : getType과 newType의 간결한 버전
    - List<Complaint> litany = Collections.list(legacyLitany);
    
    ```java
    public class Collections {
        // Suppresses default constructor, ensuring non-instantiability.
        private Collections() {
        }
    ...생략
    	public static <T> ArrayList<T> list(Enumeration<T> e) {
    	        ArrayList<T> l = new ArrayList<>();
    	        while (e.hasMoreElements())
    	            l.add(e.nextElement());
    	        return l;
    	    }
    ```
    

## 정적 팩토리

## 정적 팩토리 객체 재사용(캐시) 예시코드

```java
import java.util.HashMap;
import java.util.Map;

public class Effective {
	class User {
		private String name;

		public User(String name) {
			this.name = name;
		}
		@Override
		public boolean equals(Object obj) {
			if (obj instanceof User) {
				return ((User) obj).name.equals(this.name);
			}
			return false;
		}
	}
	class UserFactory {
		private static Map<String, User> userMap = new HashMap<>();

		public static User getUser(String name) {
			User user = userMap.get(name);
			if (user == null) {
				user = new Effective().new User(name);
				userMap.put(name, user);
			}
			return user;
		}
	}
	public static void main(String[] args) {
		User user1 = new Effective().new User("user");
		User user2 = new Effective().new User("user");
		System.out.println("user1.equals(user2) = " + user1.equals(user2));
		System.out.println("user1 == user2 = " + (user1 == user2));

		System.out.println("====================================================");

		User user3 = UserFactory.getUser("user");
		User user4 = UserFactory.getUser("user");
		System.out.println("user3.equals(user4) = " + user3.equals(user4));
		System.out.println("(user3 == user4)  = " + (user3 == user4));
	}

}
```

## 하위타입 반환 예제

```java

/**
INTERFACE
*/
public interface Seoul42Human {
	void evaluate(Seoul42Human target);
	String getName();
}
/**
CADET
**/

public class Cadet implements Seoul42Human {

	private String name;
	private int remainBlackhole;

	private Cadet(String name, int remainBlackhole) {
		this.name = name;
		this.remainBlackhole = remainBlackhole;
	}

	public static Cadet create(String name, int remainBlackhole) {
		return new Cadet(name, remainBlackhole);
	}

	@Override
	public void evaluate(Seoul42Human target) {
		System.out.println("target " + target.getName() + " evaluated by Cadet " + this.name);
	}

	@Override
	public String getName() {
		return this.name;
	}
}
/**
	MEMBER
**/
public class Member implements Seoul42Human{

	private String name;

	private Member(String name) {
		this.name = name;
	}
	public static Member create(String name) {
		return new Member(name);
	}

	@Override
	public String getName() {
		return this.name;
	}
	@Override
	public void evaluate(Seoul42Human target) {
		System.out.println("target " + target.getName() + " evaluated by Member " + this.name);
	}

}
/**
MAIN
*/

public class GaepoCluster {

	public static void main(String[] args) {
		Seoul42Human cadet1 = Cadet.create("daewoole", 123);
		Seoul42Human member1 = Member.create("jpark2");

		member1.evaluate(cadet1);
		cadet1.evaluate(member1);
	}
}

//target daewoole evaluated by Member jpark2
//target jpark2 evaluated by Cadet daewoole
```
