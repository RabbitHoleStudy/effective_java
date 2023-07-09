# Singleton 싱글톤이란?

- 인스턴스를 오직 하나만 생성하는 클래스

## 싱글톤의 단점

- 클라이언트를 테스트 하기 어려워질 수 있다?
    
    → 인터페이스로 정의하면, 인터페이스를 구현해서 만든 싱글턴이 아니면 mock 할수 없다.
    
    - 우리가 만든 Service 클래스 테스트 작성할때, ServiceImpl 을 mock 해야하는 상황을 얘기하는듯???

# 싱글턴을 만드는 방법

---

공통

- 생성자를 private로 감추고, 
유일한 인스턴스 접근수단으로 public static 멤버를 둔다

---

### final 방식

```java
public class Unity {
	public static final Unity INSTANCE = new Unity();
	private Unity() {}

}
```

- AccessibleObject 라는걸 통해서, private 생성자를 호출할수 있다고함
- API에 명백히 드러난다 (쉽게 알아볼 수 있다)
- final 이라서 다른객체 참조 불가

### static factory

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

---

# Singleton 활용 예시

## 제네릭 싱글턴

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
```

```java
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
		
		/*
	System.out.println(stringfact1.getInstanceValue());
=>	Exception in thread "main" java.lang.ClassCastException
	현재 싱글톤 객체는 Integer 라서, string 으로는 더이상 사용할 수 없다.
	
			*/
	}

}

```

- 결과

```java

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

//	System.out.println(stringfact1.getInstanceValue());

```

- `SingletonFactory<String> stringfact1` 를 하여 `String` 싱글톤을 생성후 → `SingletonFactory<Integer> intFact1` 로 `Integer` 싱글톤을 생성
- 그리고 `stringfact1` 을 출력하려 하면 `ClassCastException` 이 발생한다.

```java
public class SingletonFactory<T> {
private static SingletonFactory instance;
	private T instanceValue;
... 생략
```

- 왜냐하면 싱글톤 이니까
    
    ![(끄덕)](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/441b88d4-dbac-4250-93b6-93c7eb51a908/1592390732650.gif)
    
    (끄덕)
    

## supply 로 간결하게 사용할 수도 있다.

```java
public static void main(String[] args) {
        // Example usage of the generic singleton factory and method reference as a supplier
        Supplier<Unity>  unitySupplier= Unity::getInstance;
        Unity unity = unitySupplier.get();
    }
```

---

# Serialization 직렬화

뒤에 아이템에서 나오기때문에, 블로그글로 이론을 정리하고 나중에 다시 보자

[자바 직렬화, 그것이 알고싶다. 훑어보기편 | 우아한형제들 기술블로그](https://techblog.woowahan.com/2550/)

**`public interface Serializable**{}`

- 생성한 객체를 파일로 저장할 때
- 파일로 저장한 객체를 프로그램에서 읽을 때
- 다른 서버에서 생성한 객체를 전송받을 때

⇒ Class 를 외부로 Export 할때 Byte화 시켰다가 다시 해독하고 하는 과정을 직렬화 ↔ 역직렬화(Deserialize) 라고 한다.

### 싱글턴 클래스의 직렬화 & 역직렬화시 깨짐 현상

[☕ 싱글톤 객체가 깨져버리는 경우 (역직렬화 / 리플렉션)](https://inpa.tistory.com/entry/JAVA-☕-싱글톤-객체-깨뜨리는-방법-역직렬화-리플렉션)

위 블로그에 잘 정리되어 있다.

- 요약
    - Object → writeObject (직렬화) → Byte → readObject(역직렬화) → Object
    - readObject(역직렬화) 과정에서 JVM 은 새로운 객체라고 인식하여, 새로운 Object 로 만들기때문에 싱글톤이 깨지게된다.
- 해결방법
    - Object → writeObject (직렬화) → Byte → readObject(역직렬화) → `readResolve(역직렬화된 결과를 조정)` → Object
    - readResolve 를 넣어서, GC가 새로만든 Object 는 제거하고, 기존의 Object와 연결해주도록 한다.
        
        ```java
        class Singleton implements Serializable {
            transient String str = "";
            transient ArrayList lists = new ArrayList();
            transient Integer[] integers;
            
            private Singleton() {}
        
            private static class SettingsHolder {
                private static final Singleton INSTANCE = new Singleton();
            }
        
            public static Singleton getInstance() {
                return SettingsHolder.INSTANCE;
            }
        
            private Object readResolve() {
                return SettingsHolder.INSTANCE;
            }
        }
        ```
        

# 리플렉션

[☕ 누구나 쉽게 배우는 Reflection API 사용법](https://inpa.tistory.com/entry/JAVA-☕-누구나-쉽게-배우는-Reflection-API-사용법)

- 리플렉션은 객체를 통해서 클래스의 정보를 분석하고 런타임에 클래스의 동작을 조작하는 프로그램 기법이다.
- Spring, Lombok 같은 프레임워크에서 사용된다.
- .class 파일에서 정보를 가져오거나, 인스턴스화 할 수 있고, 메서드를 호출할 수 있다 ⇒ Reflection API 라고 부른다

## 문제

리플렉션으로 싱글톤 객체를 생성하면, 다른 객체를 반환해서 싱글톤이 깨진다.

readResolve() 로도 해결 할 수 없다.

# enum 싱글톤

C, C++ 같은경우 Enum 은 단순히 정수형타입이지만,

JAVA 에서의 Enum은 일종의 클래스로 취급된다.

- Enum은 멤버를 만들때 private 로 만들고, 한번만 초기하 하므로 Thread-safe 하다
- 변수나 메서드를 선언해 사용이 가능하므로, 독립된 싱글톤 클래스 처럼 사용이 가능
- 내부적으로 serializable 인터페이스를 구현하고 있기때문에 직렬화도 가능하다.

### 한계

enum 이 외의 클래스 상속은 불가능하므로 일반적인 클래스로만 사용이 가능하다.

### 내 생각

결국엔 내부적으로 enum 또한 열거형으로 연결되어 있기 때문에, 리터럴로 매핑되니까 인스턴스를 하나만 만들도록 내부적으로 구현되어있는게 아닐까?

하지만 object 들을 사용할수 없다면 무슨 활용이 되는지 잘 모르겠다.

### 예시

```java
public enum EnumSingleton {
	INSTANCE;
	private int val = 42;

	public int getVal() {
		return val;
	}
}
```

```java
public static void main(String[] args) {
		EnumSingleton enumSingleton1 = EnumSingleton.INSTANCE;
		EnumSingleton enumSingleton2 = EnumSingleton.INSTANCE;

		System.out.println("enumSingleton1.equals() = " + enumSingleton1.equals(enumSingleton2));

		System.out.println("enumSingleton1.hashCode() = " + enumSingleton1.hashCode());
		System.out.println("enumSingleton2.hashCode() = " + enumSingleton2.hashCode());
}
/**
enumSingleton1.equals() = true
enumSingleton1.hashCode() = 1067040082
enumSingleton2.hashCode() = 1067040082

**/
```
