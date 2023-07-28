## 핵심어

- Method Reference

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/22de071a-e35e-46f5-a8b8-0fcb3be92fe3/Untitled.png)

---

## 요약

- 메서드 참조, 람다 표현식 둘 중에 더 간결하고 명확한 것을 알잘딱으로 사용해라.

---

## 내 생각

메서드 참조 형식을 잘 이해하고 쓸 수 있으면 자주 쓸 일이 많을 것 같다. 물론 그 전에 람다에 대한 이해가 선행되어야지 약식의 느낌으로 메서드 참조를 사용할 수 있을 것 같다.

---

## 예제 코드

- 람다와 메서드 참조

```java
@Test
	void 메서드_참조와_람다() {
		// 정적
		BiFunction<String, Integer, Integer> parseIntMethod = Integer::parseInt;
		BiFunction<String, Integer, Integer> parseIntLambda = (s, radix) -> Integer.parseInt(s, radix);
		int arithmetic = 10;
		String intString = "2147";
		assertThat(parseIntMethod.apply(intString, arithmetic)).isEqualTo(2147);
		assertThat(parseIntLambda.apply(intString, arithmetic)).isEqualTo(2147);
	}
```

- map.merge에서 일급 객체인 람다 표현식 - 익명 함수를 받을 수 있는 매개변수에 대해서 알아보자

```java
// java.util.Map
    default V merge(K key, V value,
            BiFunction<? super V, ? super V, ? extends V> remappingFunction) { 
						// BiFunction<oldValue - 소비, Value - 소비, Result - 생산> remappingFunction
						// PECS를 사용한 부분이다. 
						// 결국 merge에서 사용하게될 result는 해당 map에서 저장하는 V 타입이고,
						// 밖에서 '(V의 상위,  V의 상위) -> V 이하'라는 람다 표현식으로 사용할 수 있다.
        Objects.requireNonNull(remappingFunction);
        Objects.requireNonNull(value);
        V oldValue = get(key);
        V newValue = (oldValue == null) ? value :
                   remappingFunction.apply(oldValue, value);
        if (newValue == null) {
            remove(key);
        } else {
            put(key, newValue);
        }
        return newValue;
    }

@FunctionalInterface
public interface BiFunction<T, U, R> {

    R apply(T t, U u);

		//... 생략
}

/*----------------------------------------------------------------*/

@Test
	void 맵_머지가_머지() {
		// Map 자체는 변경 가능성에 대해 아무 것도 가정하지 않는다 - 인터페이스이기 때문에.
		// 즉, Map 인터페이스는 변경 가능한 맵에도, 불변의 맵에도 사용될 수 있는 메서드 세트를 정의한다.
		// 불변의 맵은 'put', 'remove' 등의 변경 연산이 호출되면 UnsupportedOperationException을 던진다.
		// 아래의 경우 Map.of만 사용하게 되면 '불변처럼'되기 때문에, 가변가능한 HashMap으로 래핑해야 한다.
		Map<String, Number> map = new java.util.HashMap<>(Map.of("a", 1, "b", 2, "c", 3));

		// value는 비록 현재 코드에서는 3L Long이지만 merge의 value는 Number이므로,
		// merge가 사용하는 람다 표현식 내부에서는 Number(? super V 타입)로 사용된다 -> intValue()로 변환해야 한다.
		// 결과는 int 밸류에 대한 덧셈이지만, int를 박싱한 Integer가 반환된다.
		assertThat(map.merge("a", 3L, (oldValue, value) -> oldValue.intValue() + value.intValue()))
				.isInstanceOf(Integer.class); // Integer는 Number의 자식이기에, Number.class를 해도 성공한다.
		
		assertThat(map.get("a")).isEqualTo(4);
	}

```
