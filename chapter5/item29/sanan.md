## 핵심어

- Heap Pollution
    
    Java의 제네릭 시스템에서 발생하는 문제로, 제네릭을 사용하여 정의한 변수나 컬렉션에 예상하지 않은 타입의 객체가 할당되어 문제가 발생하는 상황을 일컫는다.
    
    ```java
    List<String> stringList = new ArrayList<String>();
    List rawList = stringList;
    rawList.add(7);  // Heap pollution 발생
    ```
    
    힙 오염으로 인해 변수의 선언된 타입과 실제 객체 타입이 다르면 예상치 않은 동작이 발생하거나, ClassCastException가 발생할 수 있다.
    

---

## 요약

- 제네릭 타입을 새로 만드는 일은 조금 더 어렵다. 그래도 배워두면 그만 한 값어치는 충분히 한다.
- 실체화 불가 타입으로는 배열을 만들 수 없다.
→ 런타임에서 타입 추적이 불가능하기 때문!
- 제네릭에는 원시 타입을 사용할 수 없다.
    
    → 제네릭은 컴파일 시점에 타입의 안전성을 체크하기 위한 메커니즘이다.
    이를 위해 제네릭은 타입을 체크하고, 적절하지 않는 타입의 사용을 방지한다.
    제네릭은 이러한 타입 체크를 컴파일 시에 수행한 후, 런타임에는 원래의 타입 정보를 제거하는 타입 소거(Type Erasure)를 수행한다.
    따라서, 런타임 시점에는 제네릭 타입 정보가 없고 모든 객체는 Object 타입으로 취급된다.
    그런데, 원시 타입은 Object가 아닌 별도의 타입으로 취급되기 때문에, 타입 소거 과정에서 문제가 발생할 수 있다.
    원시 타입은 null을 가질 수 없고, Object 타입으로 자동 형변환(Autoboxing)되지 않기 때문에, 컴파일 시점과 런타임 시점에서의 차이로 인해 제네릭에 원시 타입을 사용할 수 없게 되었다.
    
- 클라이언트가 직접 형변환하는 것보다 제네릭 타입이 더 안전하고 쓰기 편하다.

---

## 내 생각

제네릭 타입과 와일드 카드를 이용해서 제네릭의 이점을 통해서 재사용은 하되, Readonly와 아닌 것을 구분하는 것도 좋은 부분인 것 같다.

현업에서 구현한다면 시니어급들이 하지 않을까..?

---

## 예제 코드

- Object로 구현된 스택 클래스 제네릭 클래스로 바꿔보기

```java
public class GenericStack<E> {
	private static final int DEFAULT_INITIAL_CAPACITY = 16;
	private E[] elements;
	private int size = 0;

	@SuppressWarnings("unchecked")
	public GenericStack() {
		elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
	}

	public void push(E e) {
		ensureCapacity();
		elements[size++] = e;
	}

	public E pop() {
		if (size == 0)
			throw new EmptyStackException();
		E result = elements[--size];
		elements[size] = null; // 다 쓴 참조 해제
		return result;
	}

	public boolean isEmpty() {
		return size == 0;
	}

	public int size() {
		return size;
	}

	private void ensureCapacity() {
		if (elements.length == size)
			elements = Arrays.copyOf(elements, 2 * size + 1);
	}
}
/*----------------------------------------------------------------*/
@Test
	void 제네릭_스택() {
		GenericStack<String> stack = new GenericStack<>();
		stack.push("1");
		stack.push("2");
		stack.push("3");
		stack.push("4");
		stack.push("5");

		assertThat(stack.size()).isEqualTo(5);
		assertThat(stack.pop()).isEqualTo("5");
	}
```
