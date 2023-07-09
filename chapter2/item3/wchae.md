[item3] private 생성자나 열거 타입으로 싱글턴임을 보장하라
- 인스턴스를 오직 하나만 생성하는 클래스
- 클라이언트를 테스트 하기 어려워질 수 있다? → 인터페이스로 정의하면, 인터페이스를 구현해서 만든 싱글턴이 아니면 mock 할수 없다.
    - 우리가 만든 Service 클래스 테스트 작성할때, ServiceImpl 을 mock 해야하는 상황을 얘기하는듯???
    

# 싱글턴을 만드는 방법

## 공통

- 생성자를 private로 감추고, 유일한 인스턴스 접근수단으로 public static 멤버를 둔다

## final 방식

```java
public class Unity {
	public static final Unity INSTANCE = new Unity();
	private Unity() {}

}
```

- AccessibleObject 라는걸 통해서, private 생성자를 호출할수 있다고함
- API에 명백히 드러난다
- final 이라서 다른객체 참조 불가

## static factory

```java
public class Unity {
	private static final Unity NORTHANDSOUTH = new Unity();
	
	public static Unity getInstance() {
		return NORTHANDSOUTH;
	}

}
```

- API 를 바꾸지 않고 싱글턴 → 일반으로 변경이 가능 
getInstance 함수의 구현만 바꾸면 됨
- 제네릭 싱글턴
    
    ```java
    public class SingletonFactory<T> {
    private static SingletonFactory instance;
    	private T instanceValue;
    
    	private SingletonFactory() {}
    
    	public static synchronized <T> SingletonFactory<T> getInstance() {
    		if (instance == null) {
    			instance = new SingletonFactory<T>();
    		}
    		return instance;
    	}
    
    	public T getInstanceValue() {
    		return instanceValue;
    	}
    
    	public void setInstanceValue(T instanceValue) {
    		this.instanceValue = instanceValue;
    	}
    }
    public class Main {
    public static void main(String[] args) {
    		SingletonFactory<String> stringfact1 = SingletonFactory.getInstance();
    		stringfact1.setInstanceValue("ANG, KIMOCCCHI");
    		SingletonFactory<String> stringfact2 = SingletonFactory.getInstance();
    		System.out.println("stringfact2 = " + stringfact2);
    		System.out.println("stringfact2.getInstanceValue() = " + stringfact2.getInstanceValue());
    		System.out.println("stringfact1 = " + stringfact1);
    		System.out.println("stringfact1.getInstanceValue() = " + stringfact1.getInstanceValue());
    
    		SingletonFactory<Integer> intFact1 = SingletonFactory.getInstance();
    		intFact1.setInstanceValue(42);
    		SingletonFactory<Integer> intFact2 = SingletonFactory.getInstance();
    		System.out.println(intFact2.getInstanceValue());  // 42
    		System.out.println("intFact2 = " + intFact2);
    		System.out.println("intFact2 = " + intFact2.getInstanceValue());
    		System.out.println("intFact1 = " + intFact1);
    		System.out.println("intFact1 = " + intFact1.getInstanceValue());
    
    //결과
    stringfact2 = test.singletons.SingletonFactory@a09ee92
    stringfact2.getInstanceValue() = ANG, KIMOCCCHI
    stringfact1 = test.singletons.SingletonFactory@a09ee92
    stringfact1.getInstanceValue() = ANG, KIMOCCCHI
    42
    intFact2 = test.singletons.SingletonFactory@a09ee92
    intFact2 = 42
    intFact1 = test.singletons.SingletonFactory@a09ee92
    intFact1 = 42
    	}
    }
    ```
    
- supply 사용할 수 있다.
    
    ```java
    public static void main(String[] args) {
            // Example usage of the generic singleton factory and method reference as a supplier
            Supplier<Unity>  unitySupplier= Unity::getInstance;
            Unity unity = unitySupplier.get();
        }
    ```
    

## 열거타입
