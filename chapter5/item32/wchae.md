# [아이템32] 제네릭과 가변인수를 함께 쓸 때는 신중하라

# 가변인수

- 가변인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 해준다
- 가변인수 메서드를 호출시, 가변인수를 담기 위한 배열이 자동으로 하나 만들어진다

# 제네릭과 가변인수

- 실체화 불가 타입은 런타임에는 컴파일타임보다 타입관련 정보를 적게 담는다.
- 거의 모든 제네릭과 매개변수화 타입(List<?>, List<T>) 은 실체화 되지 않는다.
- 메서드 선언시 실체화 불가 타입으로 varargs 매개변수를 선언하면 컴파일러가 경고를 보낸다.

 

```java
warning: [unchecked] Possible heap pollution from parameterized vararg type List<String>
```

- 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다.

## 힙 오염

```java
import java.util.ArrayList;
import java.util.List;

public class HeapPollutionExample {
    public static void main(String[] args) {
        // String 타입을 저장하는 List를 생성
        List<String> stringList = new ArrayList<>();
        stringList.add("Hello");
        stringList.add("World");

        // String 타입을 저장하는 List를 다른 객체로 참조하도록 형변환
        List<Integer> integerList = (List<Integer>) (Object) stringList;

        // 정상적인 동작을 기대하지만 컴파일러는 이 부분에서 힙 오염 경고를 발생시킴
        for (Integer num : integerList) {
            System.out.println(num); // 런타임 시 ClassCastException 발생
        }
    }
}
```

- List<Integer> integerList = (List<Integer>) (Object) stringList;
⇒힙 오염이 발생하는 부분

```java
public class VarArg {

	static void dangerous(List<String>... stringLists){
		List<Integer> intList = List.of(42);

		Object[] objects = stringLists;
		objects[0] = intList; // 힙 오염 발생
		//objects[0] == List<Integer> intList 의 참조를 가진다.
//		System.out.println(objects[0].getClass());

		String s = stringLists[0].get(0); // class cast exception
		System.out.println(s);
	}
	public static void main(String[] args) {
		dangerous(List.of("a", "b","c"));
	}

}
```

- 지금 상태는 	`objects[0]` == `List<Integer> intList` 의 참조를 가지는 상태다.
- 따라서 get(0)을 했을때 나오는건, String이 아닌 List.class

⇒ 

# 근데 왜 씀?

- varargs 매개변수를 받는 메서드가 실무에서 유용하게 쓰인다
    - Arrays.asList(T… a)
    - Collections.addAll(Collection<? super T> c, T… elements)
    - EnumSet.of(E first, E… rest)

# @SafeVarargs

- Java 7 이후로 어노테이션 사용시 `@SupressedWarnings` 없이 경고를 제거할 수 있다.
- 메서드 작성자가, 메서드가 타입 안전함을 보장하는 장치다
- 따라서 타입 안정성이 확실할때만 사용하자 == 웬만하면 쓸일 없으니까 쓰지말자

```java
public static <T> List<T> pickTwo(T a, T b, T c){
		switch (ThreadLocalRandom.current().nextInt(3)){
			case 0: return List.of(a,b);
			case 1: return List.of(a,c);
			case 2: return List.of(b,c);
		}
		throw new AssertionError();
	}

	public static void main(String[] args) {
		List<String> attributes = pickTwo("아이패드","아이폰","맥북");
		for (String attribute : attributes) {
			System.out.println(attribute);
		}
	}
```

- 배열 없이 제네릭으로만 사용해서 안전하다.
