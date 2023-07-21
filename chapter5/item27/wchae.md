# 비 검사 경고

## 매개변수 명시

```java
public static void main(String[] args) {
		List list = new ArrayList();

		list.add("Hello");
		list.add("World");

	}
//Raw use of parameterized class 'ArrayList'
//Unchecked call to 'add(E)' as a member of raw type 'java.util.List'
//Unchecked call to 'add(E)' as a member of raw type 'java.util.List'
```

![image](https://github.com/TightJava/effective_java/assets/13278955/68bcdbbe-65e0-4b0e-a6e5-83128f0b0781)

# @Suppress Warnings(”checked”)

```java
import java.util.ArrayList;
import java.util.List;

public class NoWarning {
	@SuppressWarnings({"rawtypes", "unchecked"})
	public static void main(String[] args) {
		List list = new ArrayList();

		list.add("Hello");
		list.add("World");
	}
}
```

![image](https://github.com/TightJava/effective_java/assets/13278955/423db5f7-0831-45fb-8945-cc0789414829)

- 이렇게 Warning 을 숨길수는 있으나, 주석으로 명확하게 남겨야한다.

## 예시

```java
public class NoWarning {

	private void foo(Object object) {
		if (object instanceof Collection) {
			// ? 로 하면 bar(Object object) 가 불리는데,
			// 내가 원하는건 bar(Collection<Object> object) 가 불리길 원한다
			// 물음표로 하면 warning 이 없어지긴한다.
			@SuppressWarnings("unchecked")
			Collection<Object> coll = (Collection<Object>) object;
			//Collection<?> coll = (Collection<?>) object;
			bar(coll);
		}
	}

	// called when param is Collection<Object>
	private void bar(Collection<Object> object) {
		System.out.println("Collection<Object>");
	}

	// called when the param is Collection<?>
	private void bar(Object object) {
		System.out.println("object");
	}

	@SuppressWarnings({"rawtypes", "unchecked"})
	public static void main(String[] args) {
		List list = new ArrayList();
		///
		NoWarning thisClass = new NoWarning();
		Collection collection = list;

		thisClass.foo(collection);
	}
}
```

# 내 생각

- Warning 을 발생시키지 않는 걸 찾아서 작성해야할듯
- @SurpresseWarnings 를 우리가 쓸 일 이 있을까? ⇒ 적절한 예시를 아무리 생각 해봐도 잘 모르겠다..

![image](https://github.com/TightJava/effective_java/assets/13278955/ea613906-a269-4553-899c-ee96ed9f3b1a)
