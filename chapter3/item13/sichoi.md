# clone 재정의는 주의해서 진행하라

## 개요

Cloneable은 복제해도 되는 클래스임을 명시하는 용도로 사용되는 `믹스인 인터페이스(mixin interface)`이다.

- **믹스인 인터페이스(mixin interface)란?**
    
    **믹스인 인터페이스(mixin interface)**는 클래스에 추가적인 속성이나 기능을 제공하기 위한 인터페이스를 의미합니다. 여기서 중요한 점은 믹스인 인터페이스는 메인 형태와 비교했을 때 **보조적인 역할을 수행**한다는 것입니다.
    믹스인 인터페이스는 여러 클래스에서 공통적으로 사용되는 메서드를 선언함으로써 **코드의 중복을 줄이고 모듈성을 증가**시키는데 사용됩니다.
    간단한 예를 들면, `Comparable` 인터페이스는 `compareTo()` 메서드를 제공하여 객체간의 비교를 가능하게 합니다. 이 `Comparable` 인터페이스를 구현하는 클래스 (Integer, String, 등) 는 해당 메서드를 구현하게 되어 객체간의 비교가 가능해집니다. 이처럼 `Comparable`은 믹스인 인터페이스의 한 예로 볼 수 있습니다.
    Java 8 이후로 인터페이스는 디폴트 메소드와 정적 메소드를 가질 수 있게 되었고, 인터페이스가 루틴을 가질 수 있게 되어 믹스인 기능이 더욱 강화되었습니다.
    다음은 믹스인 인터페이스의 예시입니다:
    
    ```java
    public interface Flyable {
      void fly();
    }
    
    public class Bird implements Flyable {
      @Override
      public void fly() {
        System.out.println("Bird is flying");
      }
    }
    
    public class Plane implements Flyable {
      @Override
      public void fly() {
        System.out.println("Plane is flying");
      }
    }
    ```
    
    위의 코드에서 Flyable는 믹스인 인터페이스입니다. Bird와 Plane 클래스 모두 이를 구현하여 fly라는 동작을 수행할 수 있습니다.
    

Cloneable은 의도한 목적을 제대로 이루지 못했다고 한다… 가장 큰 문제는 **clone 메소드가 선언된 곳이 Cloneable이 아닌 Object**이고, 그마저도 protected라는 데 있다.

Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메소드를 호출할 수 없다.

> `리플렉션`을 이용하면 가능하긴하지만 해당 객체가 접근이 허용된 clone 메소드를 제공해야지만 가능하기에 100% 성공한다는 보장이 없다.
> 

## Cloneable 인터페이스가 하는 일

**Object의 protected 메소드인 clone의 동작 방식을 결정한다.**

`Cloneable`을 구현한 클래스의 인스턴스에서 clone 메소드를 호출하면 **그 객체의 필드들이 하나하나 복사된 객체를 반환**하고, 그렇지 않은 경우에는 `CIoneNotSupportedException`를 throw한다.

## clone 메소드의 규약

이 객체의 복사본을 생성해 반환한다. **복사**의 정확한 뜻은 그 객제를 구현한 클래스에 따라 다를 수 있다. 

일반적인 의도는 다음과 같다. 

1. **어떤 객체 x에 대해 다음 식은 참이다.**

`x.clone() != x`

1. **또한 다음 식도 참이다.** 

`x.clone().getClass() == x.getClass()`

하지만 이상의 요구를 반드시 만족해야 하는 것은 아니다. 

1. **한편 다음 식도 일반적으로 참이지만, 역시 필수는 아니다.** 

`x.clone().equals(x)`

관례상, 이 메서드가 반환하는 객체는 `super.clone()`을 호출해 얻어야 한다. 

1. **이 클래스와 (Object를 제외한) 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.**

`x.clone().getClass() == x.getClass()`
관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 

이를 만족하려면 `super.clone()`으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

즉, clone()은 지정된 객체의 새로운 복제본을 만들어야 하고, 이 복제본은 원본과 동일한 값을 가져야하지만, 원본 객체와는 달라야 한다. (다른 주소값)

원본 객체를 변경해도 복제본에는 영향이 없어야 한다. (deep copy)

상속하는 클래스가 있다면, `super.clone()`을 호출해 얻는 것이 관례이다.

예를 들어 클래스 B가 클래스 A를 상속한다고 할 때,

하위 클래스의 B의 `clone()`은 B 타입인 객체를 반환해야 한다.

그런데 A의 `clone()`이 자신의 생성자(new A(…))로 생성한 객체를 반환한다면, 

B의 `clone()`으로 만들어진 객체는 A 타입인 객체일 것이다.

## 잘 작성된 clone 예시

```java
@0verride 
public PhoneNumber Clone() { 
	try {
		return (PhoneNumber) super.clone();
	} catch (CloneNotSupportedException e) {
		throw new AssertionError();
	}
}
```

## 문제가 좀 있는 clone 예시

```java
class Stack implements Cloneable {

	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16;

	public Stack() {
		elements = new Object[DEFAULT_INITIAL_CAPACITY];
	}

	public void push(Object e) {
		ensureCapacity();
		elements[size++] = e;
	}

	public Object pop() {
		if (size == 0) {
			throw new EmptyStackException();
		}
		Object result = elements[-size];
		elements[size] = null; //다 쓴 참조 해제
		return result;
	}

	/**
	 * 원소를 위한 공간을 적어도 하나 이상 확보한다. 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
	 */
	private void ensureCapacity() {
		if (elements.length == size) {
			elements = Arrays.copyOf(elements, 2 * size + 1);
		}
	}

	@Override
	public Stack clone() {
		try {
			return (Stack) super.clone(); // 무언가 문제점이 느껴지시나요?
		} catch (CloneNotSupportedException e) {
			throw new AssertionError();
		}
	}

	@Override
	public String toString() {
		return Arrays.toString(elements);
	}
}
```

**신입 개발자(시최):** 저번에 부장님이 말씀하신 `다 쓴 참조 해제`도 추가했고, `clone()`이랑 `toString()`도 구현 해야겠다!

**신입 개발자(시최):** 히히 이러면 이뻐하시겠지? 난 천재야!

**신입 개발자(시최):** 조아..! 이대로 올려야지~

<img width="514" alt="스크린샷 2023-07-08 오후 9 59 12" src="https://github.com/TightJava/effective_java/assets/83565255/e902a97d-6cd8-4c5d-bc62-6a5acc865265">


**부장님(사난):** 에헤이~ `super.clone()`을 그대로 반환하면 어떡해!!

**부장님(사난):** elements 필드 저거저거 원본 Stack이랑 주소값 똑같겠고만~

```java
class Test {

	public static void main(String[] args) {
		Stack stack1 = new Stack();
		stack1.push("1");
		stack1.push("2");
		stack1.push("3");
		stack1.push("4");

		Stack stack2 = stack1.clone();
		stack2.push("5");

		System.out.println(stack1);
		System.out.println(stack2);
	}
}
```

<img width="674" alt="Untitled2" src="https://github.com/TightJava/effective_java/assets/83565255/47979ea2-eca5-443d-9d4c-f09624a5912e">


**부장님(사난):** 자 봐바! stack2에다가 5 넣었는데, stack1 찍었을 때도 5가 나오자나!

**신입 개발자(시최):** 허거덩?! 뭐가 문제에요??

**부장님(사난):** Stack의 상위 클래스가 Object이니까 당연히 Object는 Stack이 어떤 멤버 변수를 가졌는지 모르잖아. 너무 당연한건데 공부 안할거야?! 떼잉!

**신입 개발자(시최):** 넴.. 🥲 (그럼 어떻게 해야하지..?)

이런 사태를 막으려면 어떻게 해야할까?

**clone을 구현할 클래스의 하나뿐인 생성자를 호출**한다면 이런 상황은 절대 일어나지 않는다.

**clone 메소드 자체가 사실상 생성자와 같은 효과**를 낸다. **원본 객체에 아무런 해를 끼치지 않으며**, **복제된 객체의 불변식을 보장**해야 한다..!

### 수정된 clone

```java
	@Override
	public Stack clone() {
		try {
			Stack result = (Stack) super.clone();
			result.elements = elements.clone();
			return result;
		} catch (CloneNotSupportedException e) {
			throw new AssertionError();
		}
	}
```

elements 배열의 clone을 재귀적으로 호출해주면 된다.

이 결과는 굳이 형변환이 필요하진 않는다. 배열 자체가 원본과 똑같은 배열을 반환하도록 설계되어있기 때문에

배열을 복제할 때는 배열의 clone 메소드를 사용하기를 권장하고 있다.

## 재귀적 clone 호출만으로 해결이 안되는 경우

```java
public class HashTable implements Cloneable { 
	private Entry[] buckets = ...;
	
	private static Class Entry {
		final Object key;
		Object value;
		Entry next;

		Entry(Object key, Object value, Entry next) {
			this.key = key;
			this.value = value;
			this.next= next;
		}
		... // 나머지 코드는 생략
}
```

이 경우에 해시 테이블 내부는 버킷들의 배열이 되고, 각 버킷은 key-value 쌍을 담는 연결 리스트의 첫번째 엔트리를 참조한다.

```java
@Override
public HashTable clone() {
	try {
		HashTable result = (HashTable) super.clone();
		result.buckets = buckets.clone();
		return result;
		} catch (CloneNotSupportedException e) { 
			throw new AssertionError() ;
		} 
}
```

이런식으로 단순히 buckets.clone()을 호출하는 것 만으로는 연결 리스트의 모든 요소들이 deep copy되지 않는다.

```java
@0verride
public HashTable clone() {
	try {
		HashTable result = (HashTable) super.clone();
		result.buckets new Entry[buckets.length];
		for (int i = 0; i < buckets.length; i++) {
			result.buckets[i] = buckets[i].deepCopy();
		}
		return result;
		} catch (CloneNotSupportedException e) { 
			throw new AssertionError() ;
		} 
}
```

이렇게 buckets를 순회하면서 각 요소들을 deep copy를 시켜주어야 한다.

## 추가 고려사항

생성자에서는 재정의될 수 있는 메소드를 호출하지 말아야 한다.

그런데 clone()은 생성자같은 효과를 낸다고 말했었다. 

따라서 **clone()에서도 역시 재정의될 수 있는 메소드를 호출해서는 안된다.**

clone이 하위 클래스에서 재정의한 메소드를 호출한다면 하위 클래스는 복제 과정에서 자신의 상태를 교정할 기회를 잃는다.

예를 들어, HashTable에 값을 추가하기 위해 정의한 put(key, value)라는 메소드를 사용한다면, 이 메소드는 final이거나 private이어야 한다.

Object의 clone()은 CIoneNotsupportedException를 throw한다고 선언했지만

재정의 메소드에서는 그렇지 않다.

**public인 clone 메소드에서는 throws 절을 없애야 한다..!**

불필요하게 throws를 선언한다면 이를 사용하는 클라이언트가 굉장히 불편해질 것이다.

**상속용 클래스는 Cloneable을 구현해서는 안된다.**

그렇게되면 Object에 clone()를 선언하는 것과 똑같은 실수를 저지르는 것이다…!

**Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone()도 적절히 동기화를 시켜주어야 한다.**

- **스레드 안전 클래스란?**
    
    **스레드 안전(Thread-Safety) 클래스**란 여러 스레드에서 동시에 접근해도 프로그램의 실행에 문제가 없는 클래스를 뜻합니다. 다시 말해, 스레드 안전 클래스는 동시에 여러 스레드가 같은 객체에 접근하더라도 객체의 상태가 예상대로 유지될 것임을 보장합니다.
    스레드 안전성을 보장하기 위해선 클래스 내부에서 동기화 기법 등을 적절히 사용해야 합니다. Java에서는 `synchronized` 키워드를 이용하여 메소드 혹은 코드 블록을 한 번에 하나의 스레드만 접근할 수 있도록 제한하거나, `volatile` 키워드를 사용하여 변수의 메모리 일관성을 보장합니다.
    예를들어, `java.util.concurrent.atomic` 패키지에서 제공하는 클래스 (예: `AtomicInteger`, `AtomicLong` 등)나 `java.util.concurrent` 패키지에서 제공하는 `ConcurrentHashMap`, `BlockingQueue`등은 스레드-안전 클래스의 예시입니다.
    다만, 모든 클래스를 스레드 안전하게 만들 필요는 없습니다. 특히, 클래스를 스레드 안전하게 만들려면 추가적인 동기화 비용이 발생하므로, 상황에 따라 요구사항과 성능 사이에서 적절한 선택이 필요합니다.
    

Object의 clone()은 동기화를 신경쓰지 않았기 때문에, super.clone을 호출하는 것 이외에 다른 할 일이 없는 상황일지라도, clone()을 재정의하고 꼭 동기화시켜주어야 한다..!

## 요약

Cloneable을 구현하는 모든 클래스는 clone을 재정의해야 한다.

이때 접근 제한자는 public으로, 반환 타입은 클래스 자신으로 변경한다.

메소드는 가장 먼저 super.clone()을 호출 한 후 필요한 필드를 적절히 수정한다. (배열이나 연결 리스트로 된 필드 등의 내부 구조를 모두 deep copy)

필드의 내부 구조를 deeop copy할 때는 주로 재귀적 호출로 구현하지만, 메모리 문제가 발생할 여지가 있음에 주의해야 한다.

기본 타입 필드와 불변 객체 참조만 갖는 클래스는 필드 수정이 불필요하다. 단, 고유 ID는 기본 타입이나 불변 타입일지라도 수정해야 한다.

## 다른 대안

Cloneable은 어떻게보면 참… 몬가몬가다..!

Cloneable을 꼭 구현해서 clone()을 재정의하는 것 말고 다른 대안은 없을까?

### 복사 생성자

```java
public Yum(Yum yum) {
	// 대충 필드 deep copy하는 부분
}
```

### 복사 팩토리

```java
public static Yum newInstance(Yum yum) {
	Yum result = new Yum();
	// 대충 필드 deep copy하는 부분
	return result;
}
```

복사 생성자와 복사 팩토리는 Cloneable/clone보다 여러 방면으로 낫다.

엉성하게 문서화된 규약에 기대지 않으며, 정상적인 final 필드 용법과도 충돌하지 않고, 불필요한 Checked Exception을 던지지 않아도 되며, 불필요한 형변환도 필요하지 않다.

게다가 이 둘은 해당 클래스가 구현한 인터페이스 타입의 인스턴스도 인수로 받을 수 있다.

```java
class Test {

	public static void main(String[] args) {
		AbstractCollection<Integer> collection1 = new ArrayDeque<>();
		collection1.add(1);
		collection1.add(2);

		AbstractCollection<Integer> collection2 = new ArrayList<>(collection1);

		System.out.println(collection1);
		System.out.println(collection2);
}
```

다음은 ArrayDeque → ArrayList 변환 예시이다.

너무나도 쉽게 베이스 타입의 또다른 서브 클래스 타입으로 변환이 이루어진다..!

이렇게 인터페이스 기반 복사 생성자와 복사 팩토리의 더 정확한 이름은

**변환 생성자(conversion constructor)**와 **변환 팩토리(converision factory)**이다.

이들을 활용하면 클라이언트는 원본 구현 타입에 얽매이지 않은 채로 복제본의 타입을 쉽게 선택할 수 있다.

## 핵심 정리

`Cloneable`이 몰고 온 모든 문제를 되짚어봤을 때, 새로운 인터페이스를 만들 때는 절대 `Cloneable`을 확장해서는 안 되며, 새로운 클래스도 이를 구현해서는 안 된다. 

final 클래스라면 `Cloneable`을 구현해도 위험이 크지 않지만, 성능 최적화 관점에서 검토한 후 별다른 문제가 없을 때만 드물게 허용해야 한다(아이템 67). 

기본 원칙은 **‘복제 기능은 생성자와 팩터리를 이용하는게 최고'**라는 것이다. 

단,**배열**만은 clone 메서드 방식이 가장 깔끔한, 이 규칙의 합당한 예외라 할 수 있다.
