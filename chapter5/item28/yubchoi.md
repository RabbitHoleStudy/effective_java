### 배열과 제네릭 타입의 차이

| 배열 | 제네릭 |
| --- | --- |
| 공변. </br> Sub가 Super의 하위 타입이라면, 배열 Sub[]는 배열 Super[]의 하위 타입이 된다. | 불공변.</br> 서로 다른 타입 Type1과 Type2가 있을 때, List<Type1> ≠ (List<Type2>의 하위 타입 || 상위 타입) |
| 실체화(reify)됨.</br> 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인. | 타입 정보가 런타임에는 소거(erasure)됨.</br> 원소 타입을 컴파일 타임에만 검사하며, 런타임에는 알 수 X. |
- 코드 예시 - 배열과 제네릭 타입의 차이
    - 배열
        
        ```java
        Object[] objectArray = new Long[1];
        objectArray[0] = "타입이 달라 넣을 수 없다.";  // ArrayStoreException
        ```
        
    - 제네릭
        
        ```java
        List<Object> ol = new ArrayList<Long>();  // 호환되지 않는 타입
        ol.add("타입이 달라 넣을 수 없다.");
        ```
        
    - 배열은 런타임에서 타입 안정성을 보장하지만, 컴파일 타임에서는 보장하지 못하고
        
        제네릭은 컴파일 타임에서 타입 안정성을 보장하지만 런타임에서는 제약이 사라진다.
        

### 제네릭 배열 생성 불가

자바에서는 제네릭 배열을 만들 수 없다. typesafe하지 않기 때문에 컴파일할 때 제네릭 배열 생성 오류를 일으킨다. 이를 허용한다면 컴파일러가 자동 생성한 형변환 코드(cast)에서 런타임에 `ClassCastException`이 발생할 수 있다. ⇒ 런타임에 ClassCastException이 발생하는 일을 막아주겠다는 제네릭 타입 시스템의 취지에 어긋남

- 코드 예시 - 제네릭 배열 생성을 허용하지 않는 이유
    
    ```java
    1. List<String>[] stringLists = new List<String>[1];
    2. List<Integer> intList = List.of(42);
    3. Object[] objects = stringLists;
    4. objects[0] = intList;
    5. String s = stringLists[0].get(0);
    ```
    
    1. 제네릭 배열 생성이 허용됐다고 가정한다.
    2. 원소가 하나인 List<Integer>를 생성한다.
    3. 1에서 생성한 List<String>의 배열을 Object 배열에 할당한다. 배열은 공변이므로 문제가 없다.
    4. 2에서 생성한 List<Integer>의 인스턴스를 Object 배열의 첫 원소로 저장한다. 제네릭은 소거 방식으로 구현되기 때문에 문제가 없다. 런타임에 List<Integer> 인스턴스의 타입은 단순히 List가 되고, List<Integer>[] 인스턴스의 타입은 List[]가 된다. 따라서 ArrayStoreException 은 일어나지 않는다.
        
        ⇒ List<String> 인스턴스만 담겠다고 선언한 stringLists 배열에는 지금 List<Integer> 인스턴스가 저장돼 있다.
        
    5. stringLists 배열의 처음 리스트에서 첫 원소를 꺼내려한다. stringLists[0]에는 intList가 들어가있는데 이것을 String으로 캐스팅하려고 한다. ⇒ 런타임에 ClassCastException이 발생한다.

**그러니 배열과 제네릭을 혼합해서 사용하지 말자~**

### **배열 `E[]` 대신 리스트 `List<E>`를 사용하면 해결된다.**

- 제네릭을 사용하지 않고 구현한 Chooser
    
    ```java
    public class Chooser {
    	private final Object[] choiceArray;
    
    	public Chooser(Collection choices) {
    		choiceArray = choices.toArray();
    	}
    
    	public Object choose() {
    		Random rnd = ThreadLocalRandom.current();
    		return choiceArray[rnd.nextInt(choiceArray.length)];
    	}
    }
    ```
    
    Object 배열을 사용하여 Collection에서 요소를 무작위로 선택한다.
    
    choose 메서드의 반환값이 Object 타입이 되기 때문에, 메서드를 호출할 때마다 원하는 타입으로 형변환해야 한다. 만약 타입을 잘못 형변환하면 런타임에 ClassCastException이 발생할 수 있다.
    
- 리스트 기반 Chooser → typesafe 확보
    
    ```java
    public class Chooser<T> {
    	private final List<T> choiceList;
    
    	public Chooser(Collection<T> choices) {
    		choiceList = new ArrayList<>(choices);
    	}
    
    	public T choose() {
    		Random rnd = ThreadLocalRandom.current();
    		return choiceList.get(rnd.nextInt(choiceList.size()));
    	}
    }
    ```
    
    코드양이 조금 늘고 속도 또한 조금 느리겠지만, 런타임에 ClassCastException을 만날 일은 없다.
    

### 정리

- 배열은 공변이고 실체화되는 반면, 제네릭은 불공변이고 타입 정보가 소거된다.
- 배열은 런타임에서 타입 안정성을 보장하지만, 컴파일 타임에서는 보장하지 못하고</br>
  제네릭은 컴파일 타임에서 타입 안정성을 보장하지만 런타임에서는 제약이 사라진다.
- 배열과 제네릭을 혼용하지 말자.
- 배열과 제네릭과 같이 써서 컴파일 오류나 경고를 만날 경우, 배열을 리스트로 대체해보자.
