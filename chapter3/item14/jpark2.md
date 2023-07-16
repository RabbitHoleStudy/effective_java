### Item 14. Comparable을 구현할지 고려하라

- Comparable 인터페이스의 compareTo 메서드는 Object 메서드가 아니지만, compareTo는 동치성 비교에 더해 순서를 비교할 수 있고 제너릭하다는 점만 제외하면 Object의 equals와 같다.
    
    ```java
    public interface Comparable<T> {
    	int compareTo(T t);
    }
    ```
    
    - Comparable을 구현한다는 것은 해당 클래스의 인스턴스에는 자연적인 순서(natural order)가 있다는 의미이다.
    - Comparable을 구현한 객체들은 `Arrays.sort(a)` 이처럼 쉽게 정렬할 수 있고, 정렬이나 검색, TreeMap, TreeSet과 같은 수많은 제네릭 알고리즘과 컬렉션을 사용할 수 있다.

- compareTo 메서드의 규약
    - 주체가 되는 객체와 주어진 객체의 순서를 비교하여, 주어진 객체보다 작으면 음의 정수, 같으면 0 크면 양의 정수를 반환한다. 비교할 수 없는 경우에는 ClassCastException을 던진다.
    - 모든 x, y에 대해 `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`여야 하고, `x.compareTo(y)`가 예외를 던진다면 `y.compareTo(x)`도 예외를 던져야 한다.
    - `x.compareTo(y) > 0 && y.compareTo(z)`이면 `x.compareTo(z) > 0`을 만족해야한다(추이성을 보장해야 한다).
    - `x.compareTo(y) == 0`이면 모든 z에 대해 `sgn(x.compareTo(z)) == sgn(y.compareTo(z))`이어야 한다.
    - `(x.compareTo(y) == 0) == (x.equals(y))`이어야 한다. 이 사항은 필수는 아니지만 지키는게 좋고, 지키지 않는다면 `이 클래스의 순서는 equals 메서드와 일관되지 않다`라고 명시해야 한다.

- 기존 클래스를 확장한 구체 클래스에서 새로운 컴포넌트 값을 추가한다면 compareTo 규약을 지킬 방법이 없다.
    - 객체 지향적 추상화의 이점을 포기할 생각이 아니라면, 기존 클래스를 확장한 구체 클래스에서 새로운 컴포넌트 값을 추가하는게 아니라 독립된 클래스를 만들고 원래 클래스의 인스턴스를 가리키는 필드를 두는 것이 좋다. 그 후 내부 인스턴스를 반환하는 뷰 메서드를 제공하면 바깥 클래스에서 compareTo 메서드를 구현해 넣을 수 있다.
        
        ```java
        public class A implements Comparable<A> {
        	
        	private Integer var1;
        
        	...
        
        	@Override
        	public int compareTo(A o) {
        		return Integer.compare(var1, o.var1);
        	}
        }
        
        /*
        public class B extends A {
        
        	...
        }
        */
        
        public class C implements Comparable<A> {
        
        	private A a;
        
        	...
        
        	@Override
        	public int compareTo(A o) {
        		return Integer.compare(a.var1, o.var1)
        	}
        }
        ```
        
    
- 마지막 규약인 compareTo의 순서와  equals의 결과가 일관되지 않는 경우에도 클래스는 동작하지만, 정렬된 컬렉션의 인터페이스(Colletion, Set, Map 등)에 넣으면 정의된 동작과는 다른 동작을 보여줄 수 있다.
    - 위의 정의된 동작과 달라지는 이유는 컬렉션의 인터페이스는 equals 메서드의 규약을 따른다고 되어있지만, 동치성을 비교할 때 equals 대신 compareTo를 사용하기 때문이다.
    - 하나의 예시로 BigDecimal 클래스의 경우에는 compareTo와 equals가 일관되지 않는데, `new BigDecimal(”1.0”)`과 `new BigDecimal(”1.00”)`을 추가한 경우 equals 메서드로 비교하는 HashSet은 2개의 원소를 가지지만 compareTo 메서드로 비교하는 TreeSet을 사용하면 1개의 원소만 가지게된다.

- Comparable은 타입을 인수로 받는 제너릭 인터페이스이므로 compareTo 메서드의 인수 타입은 컴파일 시간에 정해지고, 인수 타입이 잘못된다면 컴파일이 안되기 때문에 입력 인수의 타입을 확인하거나 형변환할 필요가 없다.

- compareTo는 각 필드가 동치인지를 비교하는 게 아니라 순서를 비교한다.
    - 객체 참조 필드를 비교하려면 compareTo 메서드를 재귀적으로 호출해야한다.
        
        ```java
        public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
        
        	...
        
        	public int compareTo(CaseInsensitiveString cis) {
        		return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
        	}
        }
        ```
        
    - Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 하는 경우에는 비교자(Comparator)를 직접 만들거나 자바에서 제공하는 것을 선택하여 사용한다.

- Java 7부터 Double이나 Float처럼 compare 정적 메서드를 제공하는 타입뿐 아니라 기본 타입 클래스에도 compare 정적 메서드가 추가되었다.
    - compareTo 메서드에서 관계 연산자 `<`, `>`을 사용하는 방식은 거추장스럽고 오류를 유발하니, 추천되지 않는다.

- 클래스에 필드가 여러 개라면 가장 핵심적인 필드부터 다음으로 중요한 필드 순으로 비교하자
    - 핵심적인 필드부터 비교하여, 비교 결과가 0이 아니라면 순서가 결정되고 거기서 결과를 반환하고 끝낸다.
    - 비교 결과가 0이라면(가장 핵심이 되는 필드가 똑같다면), 비교 결과가 0이 되지 않는 필드를 찾을 때까지 다음으로 중요한 필드를 비교해나간다.
        
        ```java
        public int comapareTo(PhoneNumber pn) {
        	int result = Short.compare(areaCode, pn.areaCode);                // 가장 중요한 필드
        	if (result == 0) {
        		result = Short.compare(prefix, pn.prefix);                      // 두 번째로 중요한 필드
        		if (result == 0) result = Short.compare(lineNum, pn.lineNum);   // 세 번재로 중요한 필드
        	}
        	return result;
        }
        ```
        
    - 자바 8부터 Comparator 인터페이스가 비교자 생성 메서드를 통해 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되어 compareTo를 훨씬 깔끔하게 작성할 수 있다. 다만 약간의 성능저하가 발생한다.
        
        ```java
        private static final Comparator<PhoneNumber> COMPARATOR = 
        			comparingInt((PhoneNumber pn) -> pn.areaCode)
        							.thenComparingInt(pn -> pn.prefix)
        							.thenComparingInt(pn -> pn.lineNum);
        
        public int compareTo(PhoneNumber pn) {
        	return COMPARATOR.compare(this, pn);
        }
        ```
        
    - comparingInt는 객체 참조를 int 타입 키에 매핑하는 키 추출 함수를 인수로 받아, 그 키를 기준으로 순서를 정하는 비교자를 반환하는 정적 메서드이다. 람다(lambda)로 PhoneNumber를 comparingInt의 인수로 넘겨주어 비교하고자 하는 키들을 순차적으로 비교하도록 구성할 수 있다.

- 가끔 아래와 같이 compareTo에 `두 값의 차`를 비교하여 결과를 반환하는 동작이 있다.
    
    ```java
    static Comparator<Object> hashCodeOrder = new Comparator<>() {
    	public int compare(Object o1, Object o2) {
    		return o1.hashCode() - o2.hashCode();
    	}
    };
    ```
    
    - 이 방식은 정수 오버플로우나 부동소수점 계산 방식에 다른 오류를 발생시킬 수 있어 사용하면 안되고 아래의 두 가지 예시 중 하나를 택하여 사용해야 한다.
    - 예시 1 : 정적 compare 메서드를 활용
        
        ```java
        static Comparator<Object> hashCodeOrder = new Comparator<>() {
        	public int compare(Object o1, Object o2) {
        		return Integer.compare(o1.hashCode(), o2.hashCode());
        	}
        };
        ```
        
    - 예시 2 : 비교자 생성 메서드를 활용
        
        ```java
        static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
        ```
