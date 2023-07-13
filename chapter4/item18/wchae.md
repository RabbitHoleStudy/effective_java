- 상위 클래스와 하위 클래스를 모두 같은 프로그래머가 통제하는 패키지 안에서라면 상속도 안전한 방법이다.
- 확장할 목적으로 설계되었고 문서화도 잘 된 클래스도 안전하다

- 하지만 일반적인 구체 클래스를 패키지 경계를 넘어, 즉 다른 패키지의 구체클래스를 상속하는 일은 위험하다
- 클래스가 다른 클래스를 extends 하는 상속을 말한다
- 클래스가 인터페이스를 구현(Implements) 하는 경우를 얘기하는게 아니다

# 상속은 캡슐화를 깨뜨린다

상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.

상위 클래스는 릴리스마다 내부 구현이 달라질 수 있으며, 이러면 하위 클래스가 박살날 수 있다.

- 전체
    
    ```java
    public class InstrumentHashSetExample {
    	public class InstrumentedHashSet<E> extends HashSet<E> {
    		// 추가된 원소의 수
    		private int addCount = 0;
    
    		public InstrumentedHashSet() {
    
    		}
    
    		public InstrumentedHashSet(int initCap, float loadFactor) {
    			super(initCap, loadFactor);
    		}
    
    		@Override
    		public boolean add(E e) {
    			addCount++;
    			return super.add(e);
    		}
    
    		@Override
    		public boolean addAll(java.util.Collection<? extends E> c) {
    			addCount += c.size();
    			return super.addAll(c);
    		}
    
    		public int getAddCount() {
    			return addCount;
    		}
    	}
    
    }
    ```
    

### 예시코드

addAll 메소드는 
HashSet은  재정의 되어있지 않았고,
 HashSet의 super class인  AbstaractSet 에서 확인 할 수있다

```java
public abstract class AbstractCollection<E> implements Collection<E> {
	
	public boolean addAll(Collection<? extends E> c) {
	        boolean modified = false;
	        for (E e : c)
// 의도 : HashSet.add()
// 실제 : InstrumentHashSetExample.add()
	            **if (add(e))**
	                modified = true;
	        return modified;
	    }

// 얘는 abstract 라서 자식에서 구현이 안되어있어야 불림
public boolean add(E e) {
        throw new UnsupportedOperationException();
    }
}
```

```java
public class HashSet<E> extends AbstractSet<E> 
		implements Set<E>, Cloneable, java.io.Serializable
{
@java.io.Serial
    static final long serialVersionUID = -5024744406713321676L;
    private transient HashMap<E,Object> map;
    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
		
// addAll() 미구현 되어있다.

	public boolean add(E e) {
	        return map.put(e, PRESENT)==null;
		}
}
```

```java
public class InstrumentHashSetExample {
	public class InstrumentedHashSet<E> extends HashSet<E> {
		@Override
		public boolean add(E e) {
			addCount++;
			return super.add(e);
		}

		@Override
		public boolean addAll(java.util.Collection<? extends E> c) {
			addCount += c.size();
			return super.addAll(c);
		}
}
```

```java
	public static void main(String[] args) {
		InstrumentedHashSet<String> myHashSet = new InstrumentedHashSet<String>();
		myHashSet.addAll(List.of("Snap", "Crackle", "Pop"));
		System.out.println(myHashSet.getAddCount());
		//결과는 6
	}
```

의도

1. InstrumentHashSet.addAll()
    1. addCount++
2. super.addAll()
    1. 구현체의 add 호출
    2. AbstractCollection.add()
    3. HashSet.add()
    

실제 실행

1. InstrumentHashSet.addAll()
    1. addCount++
2. super.addAll()
    1. add( ) add 호출 ⇒ AbstractCollection.add()
    2. InstrumentHashSet.add()
        1. addCount++;
        2. super.add() ⇒ HashSet.add() 호출
        
- InstrumentHashSet 에서 addAll()을 구현하지 않으면 된다.
- 하지만 이런 해법은 
HashSet 의 addAll 은 자기자신의 add 메소드를 이용해서 구현했음을 가정해야한다
- 자기사용 여부(본인클래스의 메소드 사용)는 해당 클래스 내부 구현 방식에 해당하며, 
이런 세부 구현은 나중에 바뀔수도있다.
- 부모의 세부구현이 변경될경우 하위 클래스는 같이 깨진다

다른 해결방법으로는 원소당 add 하나씩 호출 하는 것이다

```java
public class InstrumentHashSetExample {
	public class InstrumentedHashSet<E> extends HashSet<E> {
		@Override
		public boolean add(E e) {
			addCount++;
			return super.add(e);
		}

		@Override
		public boolean addAll(java.util.Collection<? extends E> c) {
			for (E element : c){
				add(e)	
			}
		}

```

- 재정의를 피해가기 위해서 부모의 메소드 와 다른 이름의 메소드를 만든다?

⇒나중에 릴리스되면서 부모에 똑같은 함수가 생긴다면?  의도치않은 미래 재정의

# 해결책

## 래퍼클래스

- 기존 클래스를 확장(extend) 하지 않는다
- 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조 ⇒ 컴포지션(구성) (포함관계)
- 새 클래스의 인스턴스 메서드는 private 참조된 기존 클래스에 대응하는 메서드를 호출해 그 결과를 반환 
⇒ 전달 forwarding
    - 새 클래스의 메소드들 → 전달 메서드 ( forwarding method )

```java
class ForwardingSet<E> implements Set<E> {

	private final Set<E> s;

	public ForwardingSet(Set<E> s) {
		this.s = s;
	}

	public void clear() {
		s.clear();
	}

	public boolean contains(Object o) {
		return s.contains(o);
	}

	public boolean isEmpty() {
		return s.isEmpty();
	}

	public int size() {
		return s.size();
	}

	public Iterator<E> iterator() {
		return s.iterator();
	}

	public boolean add(E e) {
		return s.add(e);
	}

	public boolean remove(Object o) {
		return s.remove(o);
	}

	public boolean containsAll(Collection<?> c) {
		return s.containsAll(c);
	}

	public boolean addAll(Collection<? extends E> c) {
		return s.addAll(c);
	}

	public boolean removeAll(Collection<?> c) {
		return s.removeAll(c);
	}

	public boolean retainAll(Collection<?> c) {
		return s.retainAll(c);
	}

	public Object[] toArray() {
		return s.toArray();
	}

	public <T> T[] toArray(T[] a) {
		return s.toArray(a);
	}

	public boolean equals(Object o) {
		return s.equals(o);
	}

	public int hashCode() {
		return s.hashCode();
	}

	public String toString() {
		return s.toString();
	}
}
```

```java
public class InstrumentedSet<E> extends ForwardingSet<E> {

	private int addCount = 0;

	public InstrumentedSet(Set<E> s) {
		super(s);
	}

	@Override
	public boolean add(E e) {
		addCount++;
		return super.add(e);
	}

	@Override
	public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll(c);
	}

	public int getAddCount(){
		return addCount;
	}
}
```

- Instrumented 클래스를 WrapperClass 라고도 한다.
- 다른 Set 에 계측 기능을 덧씌운다는 뜻에서 데코레이터 패턴 ( Decorator pattern ) 이라 한다.
- 컴포지션과 전달의 조합은 넓은 의미로 위임(delegation) 이라고 부른다.
래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우만

### 단점

- 콜백 프레임워크와는 어울리지 않는다.

### 콜백 프레임워크?

자기 자신의 참조를 다른 객체에 넘겨서 다음 호출(콜백) 때 사용하도록 한다.

내부 객체는 자신을 감싸는 래퍼의 존재를 모르니, 래퍼클래스 대신 자신(this)의 참조를 넘기고, 

콜백 때는 래퍼가 아닌 내부 객체를 호출하게 된다.
⇒ SELF 문제

- 예시
    
    ### 예시
    ![Wrapper](https://github.com/TightJava/effective_java/assets/13278955/32177aa4-f0fd-42df-a0a0-bb26fa9af082)

    ```java
    final class SomeService {
    
    		void performAsync(SomethingWithCallback callback) {
    			new Thread(() -> {
    				perform();
    				// 여기서 들어온 전달인자 SomethingWithCallback 의 인스턴스는 WrappedObject
    				// 따라서 WrappedObject.call()
    				System.out.println("class = " + callback.getClass());
    				callback.call();
    			}).start();
    		}
    
    		void perform() {
    			System.out.println("Service is being performed.");
    		}
    }
    ```
    
    ```java
    class WrappedObject implements SomethingWithCallback {
    	private final SomeService service;
    
    	WrappedObject(SomeService service) {
    		this.service = service;
    	}
    
    	@Override
    	public void doSomething() {
    		service.performAsync(this);
    	}
    
    	@Override
    	public void call() {
    		System.out.println("WrappedObject callback!");
    	}
    }
    ```
    
    ```java
    class Wrapper implements SomethingWithCallback {
    	private final WrappedObject wrappedObject;
    
    	Wrapper(WrappedObject wrappedObject) {
    		this.wrappedObject = wrappedObject;
    	}
    
    	@Override
    	public void doSomething() {
    		wrappedObject.doSomething();
    	}
    
    	void doSomethingElse() {
    		System.out.println("We can do everything the wrapped object can, and more!");
    	}
    
    	@Override
    	public void call() {
    		System.out.println("Wrapper callback!");
    	}
    
    }
    ```
    
    ```java
    public class Main {
    	public static void main(String[] args) {
    		Main wrapperCallBack = new Main();
    		SomeService   service       = new SomeService();
    		WrappedObject wrappedObject = new WrappedObject(service);
    		Wrapper       wrapper       = new Wrapper(wrappedObject);
    
    		wrapper.doSomething();
    	}
    }
    ```
    
    [Wrapper Classes are not suited for callback frameworks](https://stackoverflow.com/questions/28254116/wrapper-classes-are-not-suited-for-callback-frameworks)
    
- 전달 메서드를 작성하는게 지루하다.
    - 재사용 가능한 Forwarding Class 를 인터페이스당 하나씩만 만들어두면 원하는 기능을 덧씌우는 전달 클래스들을 아주 손쉽게 구현할 수 있다.
    - forwarding class : 만약 래핑 클래스를 상속한다면, 구현해야하는 필수 메서드들

- 상속은 반드시 하위 클래스가 상위 클래스의 “진짜” 하위 타입인 상황에서만 쓰여야한다
⇒ Class B is-a ClassA
    - B 는 정말 A 인가? 생각해볼것
    - 자바 플랫폼 라이브러리에도 안좋은 예시가 있다.
        - Stack extends Vector
        - Properties extends HashTable
        
        ⇒ 개념적으로 봤을때 B is A  가 아니므로 컴포지션을 써야한다.
        
1. 내부구현을 불필요하게 노출하는 꼴이다.
2. API가 내부 구현에 묶이고, 클래스의 성능도 영원히 제한된다.
3. 클라이언트가 노출된 내부에 직접 접근 가능

- 사용자의 혼란 야기
    
    ```java
    class Person {
    	int age;
    	String name;
    
    	public Person(int age, String name) {
    		this.age = age;
    		this.name = name;
    	}
    
    	public int getAge() {
    		return age;
    	}
    
    	public String getName() {
    		return name;
    	}
    }
    public class PropertiesExample {
    
    	public static void main(String[] args) {
    		Properties properties = new Properties();
    		Person person = new Person(1234, "wchae");
    
    		properties.setProperty("name",person.getName());
    		///?
    		properties.put("name", person);
    
    		// hashtable 동작
    		System.out.println("properties.get(\"name\") = " + properties.get("name"));
    		// Properties 동작
    		System.out.println("properties.getProperty() = " + properties.getProperty("name"));
    
    //결과
    // properties.get("name") = test.item18.whynotcomposition.Person@a09ee92
    // properties.getProperty() = null
    
    	}
    }
    ```
    
- properties 는 키밸류를 String 으로 관리하려고 하는데, 
상위클래스의 .put 을 이용하면 Object가 들어간다
- getProperty() 와 상위 클래스의 get을 사용한 동작이 다르다.

## 상속을 쓰기 전에…
![했겠냐고](https://github.com/TightJava/effective_java/assets/13278955/994caf9f-0177-4592-93a6-46714f703187)

컴포지션 대신 상속을 사용하기로 결정하기 전, 질문해봐야한다.

- 상위 클래스의 API 에 아무런 결함이 없는가?
- 자식까지 결함이 있을때 전파되도 되는가?
