## 핵심어

- Type Safe Heterogeneous Container Pattern
    
    고정된 수와 타입의 객체를 저장하는 것이 가능한 일반적인 컨테이너와 달리, 다양한 타입의 객체를 안전하게 저장할 수 있는 컨테이너를 말한다.
    이 패턴은 주로 컨테이너가 특정 타입의 요소들만 가지고 있을 때 그 요소의 타입을 보장하며, 런타임에 발생할 수 있는 ClassCastException 등의 문제를 컴파일 타임에 체크하여 방지해준다.
    제네릭을 이용해서 구현되며, 컨테이너가 가지고 있을 수 있는 객체의 타입을 이용하여 객체를 가져올 때 안전하게 타입 변환을 수행할 수 있다.
    

---

## 요약

- 타입 안전 이종 컨테이너라는 것도 있단다~

---

## 내 생각

얼마 전에 JPA를 통해서 여러 엔티티를 Object[]로 가져오고, 이에 대한 형변환을 직접 런타임에서 수행하게끔 짜놓은 코드가 있었다. 해당 Repository ↔ Fetcher 관계에서 코드가 변하지 않는다면 문제가 되지 않겠지만, 결국 런타임 중에 에러가 날 여지가 분명히 존재하는 코드였다. 게다가, 가져오는 데이터의 순서도 고려해야하고 매 쿼리를 작성할 때 마다 일일히 그 부분들을 신경쓰면서 작성해야할 것 같아서 짜면서도 안티패턴인 것 같은 인상을 받았다(물론 성능개선이 급해서 넘어갔다).

이 아이템을 딱 보고 ‘써먹을 수 있을 것 같은데?’ 라는 생각이 먼저 들었다. 여러 엔티티를 담은 컨테이너를 반환해주면 되지 않을까 싶었다. 하지만 결국 그 ‘**컨테이너를 받는 입장에서 반환된 컨테이너에 무엇이 담겨있는지 반드시 알아야 오류없이 사용할 수 있다**’라는 점에서 문제가 있지 않을까, 엄밀하게 객체간 메시지만으로 소통하는 방식이 아니라는 생각이 들었고, 결국 지금의 방식처럼 별도의 DTO 클래스로 래핑해서 이름으로 데이터를 명확히 표현하고 해당 DTO를 사용하는 계층의 입장에서 신뢰하고 바로 사용할 수 있는(반드시 그 정보가 무엇인지 알 필요가 없는) 상태로 사용하는게 맞지 않을까 하는 생각이 들었다.

---

## 예제 코드

```java
class TypeSafeContainer {
  private Map<Class<?>, Object> favoriteMap = new HashMap<>();

  <T> void putFavorite(Class<T> type, T instance) {
    if (type == null) {
      throw new NullPointerException("Type is null");
    }
    favoriteMap.put(type, instance);
  }

  <T> T getFavorite(Class<T> type) {
    return type.cast(favoriteMap.get(type));
  }

  public static void main(String[] args) {
    TypeSafeContainer container = new TypeSafeContainer();
    container.putFavorite(String.class, "Java");
    container.putFavorite(Integer.class, 0xcafebab);
    container.putFavorite(Class.class, TypeSafeContainer.class);

    String favoriteString = container.getFavorite(String.class);
    int favoriteInteger = container.getFavorite(Integer.class);
   Class<?> favoriteClass = container.getFavorite(Class.class);

    System.out.printf("%s %x %s%n", favoriteString, favoriteInteger, favoriteClass.getSimpleName());
  }
}
```
