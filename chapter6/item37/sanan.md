## 핵심어

- EnumMap
    - EnumMap은 **해시 기반 맵에 비해 매우 빠른 성능**을 보인다. 이는 내부적으로 배열을 사용하며, 열거형의 순서를 이용하여 데이터를 저장하고 액세스하기 때문이다. 여기서 중요한 점은 **열거형 값은 정적이므로 변경되지 않아 동일한 인덱스로 항상 접근할 수 있다는 것**이다.
    - EnumMap은 키가 들어온 순서를 유지한다. Enum의 순서에 따라 맵에 유지되므로, 이 맵의 entrySet이나 keySet을 순회할 때 순서가 유지된다.
    - EnumMap에서는 키로 null을 사용할 수 없다. 그러나 값(Value)은 null이 될 수 있다.
    - EnumMap은 스레드 안전(Thread-Safe)하지 않다. **멀티스레드 환경에서는 Collections.synchronizedMap() 메서드를 사용하여 동기화해야** 한다.

---

## 요약

![asdf](https://github.com/TightJava/effective_java/assets/105692206/054625bd-5e0a-4ed5-acfd-48d74622ecb4)


- ordinal을 배열의 index로 사용하지 마라.
    
    정확한 정숫값을 너가 보장할거야?
    
- 열거 타입을 Key로 갖게끔 하는 Map이 있으니 그것이 EnumMap.

---

## 내 생각

결국 EnumMap 같은 자료형이 있다는 것을 알아야 쓸 수 있는 것 같다. 뭔가 java.util에서 제공하는 자료형들을 한번쯤 훑어보는게..

---

## 예제 코드

```java
public enum DayOfWeek {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
}

public class Restaurant {
    public static void main(String[] args) {
        EnumMap<DayOfWeek, String> menu = new EnumMap<>(DayOfWeek.class);

        menu.put(DayOfWeek.MONDAY, "Pasta");
        menu.put(DayOfWeek.TUESDAY, "Burger");

        System.out.println(menu);
    }
}
```
