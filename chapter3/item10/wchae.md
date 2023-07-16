- equals 는 규약을 지켜서 재정의 하라
- 그러지 못할꺼면 재정의 하지마라

# 각 인스턴스가 본질적으로 고유하다

```java
class Obejct {
...
	public boolean equals(Object obj) {
	        return (this == obj);
	    }
...
}
```ㅓ

- Object 의 필드가 아닌, Object 자체의 값 (메모리 주소) 비교

# 인스턴스의 논리적 동치성(logical equality) 를 검사할 일 이 없다.

- equals 를 재정의 하지 않았다는 것은, 설계자가 *논리적 동치성을 검사하지 말라는 의미
    
    *값 비교
    

```java
Pattern pattern1 = Pattern.compile("\\d+");
Pattern pattern2 = Pattern.compile("\\d+");

pattern1.equals(pattern2)
// 결과는 false
```

- Pattern 의 equals( )  는 Object 의 equals 를 그대로 사용한다 (재정의 하지 않았다)
- 인스턴의 값끼리 비교하지 말라는 얘기다

![그거 그렇게하는거아닌데 ㅋㅋ](https://github.com/TightJava/effective_java/assets/13278955/04db9e18-cbcc-4777-a0ae-af657791ee27)

그거 그렇게하는거아닌데 ㅋㅋ

- 설계자의 의도 자체가, 위와 같은짓을 하지말라고 만들어 놓은 것이다.
- 만약 같은 패턴이라면 새로 만드는 것은 메모리 낭비다 
⇒인스턴스 하나로 하라는 의미

# 상위 클래스에서 재정의한 equals 가 
하위 클래스에도 딱 들어맞는다.

- 대부분의 Set 구현체는 AbstractSet이 구현한 구현한 Equals 를 상속받아서 사용한다
    
    ```java
    public abstract class AbstractSet<E> extends AbstractCollection<E> implements Set<E> {
    
           public boolean equals(Object o) {
            if (o == this) // 주소값 비교
                return true;
    
            if (!(o instanceof Set)) // 인스턴스 타입 (Set) 비교
                return false;
            Collection<?> c = (Collection<?>) o; 
            if (c.size() != size()) // 사이즈 비교
                return false;
            try {
                return containsAll(c); // 구성체 (값) 비교
            } catch (ClassCastException | NullPointerException unused) {
                return false;
            }
        }
    ```
    
- List 구현체들은 AbstractList
    - ArrayList 는 아님
        
        ```java
        //ArrayList.equals
        public boolean equals(Object o) {
                if (o == this) {
                    return true;
                }
        
                if (!(o instanceof List)) {
                    return false;
                }
        
                final int expectedModCount = modCount; //modCount 는 객체 실행시마다 변경사항++ 카운터
                // ArrayList can be subclassed and given arbitrary behavior, but we can
                // still deal with the common case where o is ArrayList precisely
                boolean equal = (o.getClass() == ArrayList.class)
                    ? equalsArrayList((ArrayList<?>) o)
                    : equalsRange((List<?>) o, 0, size);
        
                checkForComodification(expectedModCount);
                return equal;
            }
        ```
        
    - LinkedList.equals
        
        ```java
        public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
        
        	public boolean equals(Object o) {
        	        if (o == this)
        	            return true;
        	        if (!(o instanceof List))
        	            return false;
        	
        	        ListIterator<E> e1 = listIterator();
        	        ListIterator<?> e2 = ((List<?>) o).listIterator(); // 이터레이터 비교
        	        while (e1.hasNext() && e2.hasNext()) { // 링크를 타고 돌면서 구성물 비교
        	            E o1 = e1.next();
        	            Object o2 = e2.next();
        	            if (!(o1==null ? o2==null : o1.equals(o2)))
        	                return false;
        	        }
        	        return !(e1.hasNext() || e2.hasNext()); 
        	    }
        }
        ```
        
- Map 구현체들은 AbstractMap
    
    ```java
    //AbstractMap
    public boolean equals(Object o) {
            if (o == this)
                return true;
    
            if (!(o instanceof Map<?, ?> m))
                return false;
            if (m.size() != size())
                return false;
    
            try {
                for (Entry<K, V> e : entrySet()) {
                    K key = e.getKey();
                    V value = e.getValue();
                    if (value == null) {
                        if (!(m.get(key) == null && m.containsKey(key)))
                            return false;
                    } else {
                        if (!value.equals(m.get(key)))
                            return false;
                    }
                }
            } catch (ClassCastException | NullPointerException unused) {
                return false;
            }
    ```
    

# 클래스가 private 이거나 package-private 이고 equals 메서드를 호출할 일이 없다

- 클래스가 비공개이거나 패키지-비공개일 경우, 해당 클래스는 내부에서만 사용되고 외부에서는 직접 접근 불가
- 따라서 호출할 일이 없다
    
    ```java
    @Override
    public boolean equals(Object o) {
    	throw new AssertionError();
    }
    ```
    

⇒ 막아두자

# Equals 재정의시 지켜야하는 규약

## 반사성 (reflexivity)

- null 이 아닌 모든 참조값 x 에 대해
    - x.equals(x) 는 true

## 대칭성 (symmetry)

- null 이 아닌 모든 참조 값 x, y에 대해
    - x.equals(y) ==  true
    - y.equals(x) 도 true

## 추이성 (transitivity)

- null 이 아닌 모든 참조 값 x,y,z에 대해
    - x.equals(y) == true
    - y.equals(z) == true
    - x.equals(z) == true

## 일관성 (consistency)

- x.equals(y) 를 반복해서 호출하면 항상 true / false
- java.net.URL 의 equals 는 URL 과 매핑된 호스트 IP 주소를 비교한다 ⇒ 호스트이름을 IP로 바꾸려면 네트워크를 통해야하는데, 그 결과가 항상 같다고 보장할 수없다 ⇒ 규약 위반
- 항시 메모리에 존재하는 객체만 사용한 결정적 deterministic 계산만 수행해야 한다

## null - 아님

- x.equals(null)은 false
- null 검사를 하도록 한다. ⇒ ≠null 이 아닌 (o instanceof MyType) 으로 검사한다.
    - instanceof 는 첫 번째 피 연산자가 null 이면 false를 반환한다

# Equals 를 구성하는 요소

## 1. == 연산자를 이용해 입력이 자기 자신의 참조인지 확인한다

a.equals(a) 일경우 true 반환

성능 최적화용이다.

## 2. instanceof 연산자로 입력이 올바른 타입인지 확인한다

인터페이스를 구현한 클래스라면 equals 에서 해당 클래스가 아닌 인터페이스로 비교해야한다.

```java
//AbstractMap
public boolean equals(Object o) {
  ...
        if (!(o instanceof Map<?, ?> m)) //인터페이스 타입 비교
            return false;
```

## 3. 입력을 올바른 타입으로 형변환한다

2번에서 instanceof 검사를 했기때문에 이 단계는 100% 성곤한다

## 4. 입력 객체와 자기 자신의 대응되는 ‘핵심’ 필드들이 모두 일치하는 하나씩 검사

- 기본타입은 == 연산자로 비교
- 참조타입은 equals
- float, double 은 [Float.compare](http://Float.compare) 혹은 Double.compare
    - equals 는 오토박싱을 수반할 수 있으니 성능상 좋지않다.
- 배열 필드는 모든 원소가 핵심필드라면 Arrays.equals 메서드들 중 하나를 사용해라
- null 이 필수적이라면 Objects.equals(Object, Object)로 비교하여 NPE 를 피하자

# Equals 를 재정의했다면 hashCode도 재정의하자
