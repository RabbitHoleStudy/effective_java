# [아이템 19] 상속을 고려해 설계하고 문서화하라 그러지 않았다면 상속을 금지하라


# 안전한 상속방법

(인터페이스 “구현” 이 아닌 클래스 상속(extend) )

- 메서드를 재정의하면 어떤 일이 일어나는지를 정확히 정리하여 문서화 하라
- 상속용 클래스는 재정의 할 수 있는 메서드들을 내부적으로 어떻게(자기사용) 되는지 문서로 남겨야 한다.
- 클래스의 API 로 공개된 클래스에서 자신의 또 다른 메서드를 호출할 수도 있다.
    - 자신의 또 다른 메서드가 재정의 가능 메서드라면
     그 사실을 호출하는 메서드의 API 설명에 적시해야한다.
    - 어떤 순서로 호출하는지, 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지
    - public, protected 중 final 이 아닌 모든 메서드
- @implSpec 태그로 설정 가능
    - Implementation Requirements : 그 메서드의 내부 동작 방식을 설명하는 곳이다.

## @ImplSpec 문서화

## 문서화 하는 방법

- 

### 자바 제공 클래스 예시

- 아이템 18에서 봤던 addAll
    
    ```java
    /**
         * {@inheritDoc}
         *
         * @implSpec
         * This implementation iterates over the specified collection, and adds
         * each object returned by the iterator to this collection, in turn.
         *
    
    이 구현은 지정된 컬렉션을 반복하고 이터레이터가 반환한 각 객체를 이 컬렉션에 차례로 추가합니다.
    
         * <p>Note that this implementation will throw an
         * {@code UnsupportedOperationException} unless {@code add} is
         * overridden (assuming the specified collection is non-empty).
         * 생략     
    이 구현은 {add}가 재정의되지 않는 한(지정된 컬렉션이 비어 있지 않다고 가정할 때) 
    UnsupportedOperationException을 throw합니다.
    
    */
        public boolean addAll(Collection<? extends E> c) {
            boolean modified = false;
            for (E e : c)
                if (add(e))
                    modified = true;
            return modified;
        }
    ```
    
- remove
    
    ```java
    /**
         * {@inheritDoc}
         *
         * @implSpec
         * This implementation iterates over the collection looking for the
         * specified element.  If it finds the element, it removes the element
         * from the collection using the iterator's remove method.
         *
    이 구현은 지정된 요소를 찾기 위해 컬렉션을 반복합니다.  
    요소를 찾으면 이터레이터의 제거 메서드를 사용하여 컬렉션에서 요소를 제거합니다.
    
         * <p>Note that this implementation throws an
         * {@code UnsupportedOperationException} if the iterator returned by this
         * collection's iterator method does not implement the {@code remove}
         * method and this collection contains the specified object.
    
    이 컬렉션의 이터레이터 메서드가 반환하는 이터레이터가 {remove} 메서드를 구현하지 않고 
    이 컬렉션에 지정된 객체가 포함되어 있는 경우 
    이 구현은 UnsupportedOperationException을 던집니다.
         *
         */
        public boolean remove(Object o) {
            Iterator<E> it = iterator();
            if (o==null) {
                while (it.hasNext()) {
                    if (it.next()==null) {
                        it.remove();
                        return true;
                    }
                }
            } else {
                while (it.hasNext()) {
                    if (o.equals(it.next())) {
                        it.remove();
                        return true;
                    }
                }
            }
            return false;
        }
    ```
    
    - 이렇게 구구절절?

```java
一tag "impl5pec:a:Implementation Requlrements:"
```

명령줄 매개변수로 지정해주면 @implSepc 태그를 사용할 수 있다.

- 좋은 API 문서란 어떻게? 가아니라 무엇 을 하는지 설명해야한다
- 상속이 캡슐화를 해치기 때문에 어쩔 수 없이 쓰는것.
- 안전하게 상속할 수 있다면 기술하지 말아야한다.

---

# protected 로 공개해야 하는 함수

- 클래스의 내부 동작 과정 중간에 끼어들 수 있는 hook 을 잘 선별하여 protected 메서드 
형태로 공개해야 할 수도 있다.

- 상속용 클래스를 설계할 때 어떤 메서드를 protected로 할까?
    - 실제 하위 클래스를 3개정도 만들어보고 시험해봐야한다
    - 필요한 protected 멤버를 놓쳤다면, 하위 클래스를 작성할 때 티난다
    - 하위 클래스를 여러개 만들때까지 전혀 protected 멤버를 안썼으면 private 여야할 가능성이 크다
    
    ⇒ 배포 전 반드시 하위 클래스를 만들어 검증해라
    

---

# 생성자 에서 주의해야 할점

- 상속용 클래스의 생성자는 직접이든 간접이든 재정의 가능 메서드를 호출해서는 안된다.
- clone, readObject 도 생성자와 비슷한 역할을 하기 때문에 마찬가지이다.

## 생성자 재정의 에러 예제코드

```java
class Super {
	public Super() {
		overrideMe();
	}
	public void overrideMe() {
	}
}

final class Sub extends Super {
	private final String name;

	Sub(){
		name = "ANONYMOUS";
	}

	@Override
	public void overrideMe() {
		System.out.println("name = " + name);
	}
}
public class OverrideExtend {

	public static void main(String[] args) {
		Sub sub = new Sub();
		Super subSuper = new Sub();
		sub.overrideMe();
	}
}

// 예상결과
// 출력없음 (Super.overrideMe())
// ANONYMOUS

// null (Sub.overrideMe())
// ANONYMOUS
```

# 일반 클래스의 안전한 상속

- 재정의 가능 메서드의 본문 코드를 private 도우미 메서드로 옮기고 이 도우미 메서드를 통해 호출

```java
class AddSuper {

	public int addAll(int... numList) {
		int result = 0;
		for (int num : numList) {
			result += add(result, num);
		}
		return result;
	}

	public int add(int a, int b) {
		return a + b;
	}

	private int myAdd(int a, int b) {
		return a + b;
	}

}

class Jasik extends AddSuper {

	@Override
	public int add(int a, int b) {
		return a * b;
	}
}

public class MakePrivate {

	public static void main(String[] args) {
		Jasik jasik = new Jasik();
		int i = jasik.addAll(1, 2, 3, 4, 5);
		System.out.println("i = " + i);
	}

}

//result  0

// addMy  result == 15
```

# 상속 금지시키기

1. final class
2. static factory 메소드 사용
