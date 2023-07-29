# [아이템 33] 타입 안전 이종 컨테이너를 고려하라

# Class

- class 리터럴 타입은 Class<T> 이다
    - String.class 의 타입은 Class<String>
    - Integer.class 의 타입은 Class<Integer>
- 컴파일 타임 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴을 타입 토큰 이라 한다.

# 타입 안전 이종 컨테이너 예제

```java
public class Favorites {

	private Map<Class<?>, Object> favorites = new HashMap<>() ;

	public <T> void putFavorite(Class<T> type, T instance){
		favorites.put(Objects.requireNonNull(type), instance);
	}
	public <T> T getFavorite(Class<T> type){
		return type.cast(favorites.get(type));
	}

	public static void main(String[] args) {
		Favorites f = new Favorites();
		f.putFavorite(String.class, "JAVA");
		f.putFavorite(Integer.class, 0xcafebabe);
		f.putFavorite(Class.class, Favorites.class);

		String favoriteString = f.getFavorite(String.class);
		int favoriteInteger = f.getFavorite(Integer.class);
		Class<?> favoriteClass = f.getFavorite(Class.class);

		System.out.printf("%s %x %s%n", favoriteString, favoriteInteger, favoriteClass.getName());
	}

}
```

```java
private Map<Class<?>, Object> favorites = new HashMap<>() ;
```

`Map<Class<?>, ~>`

- Map<?,?> 였다면 아무것도 넣을 수 없겠지만, 중첩되어있다.
- 맵의 타입이 아니다.
    - 맵의 키 타입은 Class 타입
    - 인데 Class에서 <?> 모든것
- 이렇게 중첩되면 모든 클래스가 들어갈 수 있게된다.
- 다양한 타입을 지원하는 힘?

`Map<~,Object>`

- 이 맵은 키와 값 사이의 타입 관계를 보증하지 않는다
- 키로 명시한 타입임을 보증하지 않는다.
보증할 방법이 없다.

```java
	public <T> void putFavorite(Class<T> type, T instance){
		favorites.put(Objects.requireNonNull(type), instance);
	}
```

```java
//		f.putFavorite((Class)Integer.class, "ASDF");
```

- 아래와 같이 강제로 형변환하여 putFavorite 메소드로 넣을 수 있다.

```java
int favoriteInteger = f.getFavorite(Integer.class);

public <T> T getFavorite(Class<T> type) {
		return type.cast(favorites.get(type));
	}
```

- 하지만 get 에서  type.cast 로 동적 변환하기때문에, 컴파일에러를 낸다.
    - 전달인자로 받은 Type 은 Integer
    - type.cast(”ASDF”) 의 결과는⇒ String

넣는것도 안전하게 하려면

```java
public <T> void putSafeFavorite(Class<T> type, T instance) {
		favorites.put(Objects.requireNonNull(type), type.cast(instance));
	}
```

- Collection의 checkedSet, checkedList, checkedMap 같은 메서드에서 위와같은 로직을 사용하여, 타입 안전성을 보장

# 일단 정리

![images](https://github.com/TightJava/effective_java/assets/13278955/b2fd5331-a619-48c7-b7c6-a061eae6378e)

- Map<K,V> 같은경우 사용할 수 있는 타입이 키타입과 밸류타입 두가지로 정의 된다.
- 하지만 이종 컨테이너를 쓰면 타입에 제약없이 한 컨테이너에서 다양한 타입을  지원할 수 있다.
- 이때 Class<T> type.cast(컨테이너.get(type))
즉 cast 메소드로, 현재 컨테이너안에 저장된 타입과, 키 타입을 cast 하여 비교함으로서, 타입안전성을 보장할 수 있다
