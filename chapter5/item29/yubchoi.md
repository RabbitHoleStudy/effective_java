### 제네릭 타입을 새로 만들어보자

- Object 기반 스택 코드이다.
    
    ```java
    public class Stack {
    	private Object[] elements;
    	private int size = 0;
    	private static final int DEFAILT_INITIAL_CAPACITY = 16;
    
    	public Stack() {
    		elements = new Object[DEFAULT_INITIAL_CAPACITY];
    	}
    
    	public void push(Object e) {
    		ensureCapacity();
    		elements[size++] = e;
    	}
    
    	public Object pop() {
    		if (size == 0)
    			throw new EmptyStackException();
    		Object result = elements[--size];
    		elements[size] = null;  // 다 쓴 참조 해제
    		return result;
    	}
    
    	public boolean isEmpty() {
    		return size == 0;
    	}
    
    	private void ensureCapacity() {
    		if (elements.length == size)
    			elements = Arrays.copyOf(elements, 2 * size + 1);
    	}
    }
    ```
    
    스택에서 꺼낸 객체를 형변환하면서 런타임 오류가 날 위험이 있다. 제네릭으로 만들어보자!
    

### 일반 클래스를 제네릭 클래스로 만드는 방법

클래스 선언에 하나 이상의 타입 매개변수를 추가한다. 이 때 타입의 이름으로 보통 E를 사용한다. E와 같은 실체화 불가 타입으로는 배열을 만들 수 없다.

제네릭 스택으로 바꿔봤지만 컴파일 오류가 난다.

```java
public class Stack<E> {
	private E[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16;

	public Stack() {
		elements = new E[DEFAULT_INITIAL_CAPACITY]; // Type parameter 'E' cannot be instantiated directly.
	}

	public void push(E e) {
		ensureCapacity();
		elements[size++] = e;
	}

	public E pop() {
		if (size == 0)
			throw new EmptyStackException();
		E result = elements[--size];
		elements[size] = null;  //  다 쓴 참조 해제
		return result;
	}
	... // isEmpty와 ensureCapacity 메서드는 그대로
}
```

### 제네릭 배열을 만드는 방법 2가지
1. Object 배열을 생성한 다음 제네릭 배열로 형변환한다.
    
      ```java
      // 배열 elements는 Push(E)로 넘어온 E 인스턴스만 담는다. => 타입 안정성 보장
      // 이 배열의 런타임 타입은 E[]가 아닌 Object[]이다.
      @SuppressWarnings("unchecked")
      public Stack() {
      	elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
      }
      ```
    
      - 제네릭 배열 생성 금지 제약을 우회하는 방법으로, 컴파일러 오류 대신 경고가 뜬다.
      - 컴파일러는 하지 못하는 typesafe 증명을 우리가 해보자.
          
          배열 elements는 private 필드에 저장되고, 클라이언트로 반환되거나 다른 메서드에 전달되는 일이 전혀 없다. 또한, push 메서드를 통해 배열에 저장되는 원소의 타입은 항상 E다. 따라서 이 비검사 형변환은 확실히 안전하다.
          
      - 비검사 형변환이 안전하므로 범위를 최소로 좁혀 `@SuppressWarnings` 어노테이션으로 해당 경고를 숨긴다. 어노테이션을 달면 Stack은 깔끔히 컴파일되고, 명시적으로 형변환하지 않아도 `ClassCastException` 걱정 없이 사용할 수 있다.
  2. elements 필드의 타입을 `E[]`에서 `Object[]`로 바꾼다.
    
      ```java
      public class Stack<E> {
      	**private Object[] elements;**
      	private int size = 0;
      	private static final int DEFAULT_INITIAL_CAPACITY = 16;
      
      	public Stack() {
      		**elements = new Object[DEFAULT_INITIAL_CAPACITY];**
      	}
      
      	// 비검사 형변환을 수행하는 할당문에서만 경고를 숨김
      	public E pop() {
      		if (size == 0)
      			throw new EmptyStackException();
      		// push에서 E 타입만 허용하므로 이 형변환은 안전하다.
      		**@SuppressWarnings("uncheked")**
      		E result = **(E)** elements[--size];
      		elements[size] = null;  //  다 쓴 참조 해제
      		return result;
      	}
      	... // 나머지 메서드는 그대로
      }
      ```
      
      - 배열이 반환한 원소를 E로 형변환하면 오류 대신 경고가 뜬다.
- 방법 2와 비교했을 때 방법 1의 장단점
    - 장점
        - 가독성이 더 좋다. 배열의 타입을 E[]로 선언하여 오직 E 타입 인스턴스만 받는다는 것을 확실하게 알 수 있다.
        - 코드도 더 짧다.
        - 방법 1에서는 형 변환을 배열 생성 시 한 번만 해주면 되지만, 방법 2에서는 배열에서 원소를 읽을 때마다 해줘야 한다.
    - 단점
        - E가 Object가 아닌 한, 배열의 런타임 타입이 컴파일타임의 타입과 달라 heap pollution을 일으킨다. 현재 예시에서는 문제가 되지 않는다.
- 현업에서는 주로 방법 1을 선호하지만 heap pollution 문제 때문에 방법 2를 사용하기도 한다.
- heap pollution은 뒷장에서 나오니 지금은 간단하게 알아보자.
    - 매개변수화 타입이 매개변수화 타입이 아닌 것을 참조할 때 발생하는 현상.
    - 매개변수화 타입은 타입 소거가 이루어진다. 따라서 런타임 시점에는 타입 정보를 모른다.
    - 하지만 매개변수화 타입이 아닌 배열은 런타임 시점에 타입 정보를 안다.
    - 컴파일타임과 런타임의 정보가 달라서, `UncheckedWarning`과 `ClassCastException`이 발생.

- 방법 1과 2 모두 클라이언트에서 elements를 받아올때 형변환하지 않아도 문제가 발생하지 않는다. 컴파일러에 의해 자동 생성된 형변환이 항상 성공함을 보장한다.
    
    ```java
    public static void main(String[] args) {
    	Stack<String> stack = new Stack<>();
    
    	for (String arg : args)
    		stack.push(arg);
    	while (!stack.isEmpty())
    		System.out.println(stack.pop().toUpperCase());
    }
    ```
    

- 아이템 28에서는 배열보다 리스트를 우선하라고 했지만, 사실 제네릭 타입 안에서 리스트를 사용하는 게 항상 가능하지도, 꼭 더 좋은 것도 아니다.
- 자바가 리스트를 기본 타입으로 제공하지 않으므로 ArrayList 같은 제네릭 타입도 결국은 기본 타입인 배열을 사용해 구현해야 한다. 또한, HashMap 같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용하기도 한다.

### 정리

- 클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다.
- 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 하자.
- 기존 타입 중 제네릭이었어야 하는 게 있다면 제네릭 타입으로 변경하자.
- 제네릭 타입으로 만들 수 없는 경우라면, 형변환을 해도 안전한지 사람이 직접 확인한 후, @SuppressWarnings("unchecked") 어노테이션을 달아 경고를 숨기자.
