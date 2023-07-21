# 제네릭 타입

- 클래스와 인터페이스 선언 타입에 매게변수(type-parameter)가 쓰이면, 이를 제네릭 클래스 혹은 제네릭 인터페이스 라고 한다.
- List 인터페이스는 원소의 타입을 나타내는 타입 매개변수 E 를 받는다.
    
    `List<E> == List`
    
- List<String> 원소 타입이 String

```java
public interface List<E> extends Collection<E> {
```

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

```java
public class Test {

	public static <T> void genericAdder(T e, T e2){
		System.out.println(e.toString()+ " " + e2.toString());
	}
	public static void main(String[] args) {
		genericAdder(1, 2);
		genericAdder("A","B");
		genericAdder(1.0,2.0);
	}
}
```

# RAW Type

- 제네릭 타입을 하나 정의하면 그에 딸린 raw type 도 함께 정의된다.
- 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않은 것을 말한다. 
⇒ List<E> 의 raw 타입은 List
- 제네릭이 도래하기 전 코드와 호환되도록 하기 위해 존재
    
    ```java
    //raw type
    private final Collection stamps = ...;
    
    //generic specific type
    private final Collection<String> stamps = ...;
    ```
    

## Raw type 으로 표기 하지마

WHY? 런타임에가 나니까!

- next 에 int 만 넣어서, add 해준다고 생각하고 만든 List class

```java
Collection number = new ArrayList();
		number.add("a");
		number.add("b");
		for (Object o : number) {
			Integer next = (Integer) o;
			next.toString();
		}
```

- 선언하고 넣을때 까지는 unchecked Call 정도의 warning 발생
- 하지만 사용하려고 하는순간 `ClassCastException`

- 언어 차원에서 raw 타입을 쓰는걸 막지 않았지만, 쓰지마라
- 쓸거면 와일드카드 ? 몰루 를 써라
- 제네릭이 주는 안전성과 표현력 둘다 일는다
  
    ![image](https://github.com/TightJava/effective_java/assets/13278955/87d835f3-bac5-44ce-8ff2-d1290ce1df43)
    
- 즉 raw 타입은 구시대의 유물이다

# 내 생각

- 사용하는사람이나, 만드는사람이나 어떤것이든 명확한 것이 좋다
- 따라서 저런 코드의 오용을 막기 위해서는 타입을 꼭 명시하도록 하자
- (기존에도 잘 하고 있었던 거지만)

- 와일드 카드를 직접 활용할일이 있을지 모르겠다
- List 에서 넓은 바운더리로 받고 싶을때, `?` 와 제한된 제네릭
`? extends Type
? super Type`
    
     조합해서 사용할만한 케이스를 생각해봐야겠다
