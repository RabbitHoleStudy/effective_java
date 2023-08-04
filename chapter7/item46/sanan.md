**한 줄 요약 : Stream의 종단 연산인 ForEach의 경우 side effect를 낼 여지가 많으므로 생각해서 써라.**

## 핵심어

### 부작용(Side Effect)

함수형 프로그래밍에서 나온 개념으로, 프로그램의 **어떤 함수가 외부의 상태를 변경하거나, 함수의 실행이 그 외부 상태에 의존한다면 그 함수는 '부작용이 있다'**고 말한다.

간단히 말해, 함수가 시스템의 상태를 변경하거나, 함수 자신 이외의 것에 영향을 미치고, 그 상태 변경이 함수의 반환값 및 동작에 영향을 줄 때 이를 '부작용'이 있다고한다.

이러한 부작용이 있는 코드는 여러 스레드에서 동시에 실행되면 예상치 못한 결과를 초래할 수 있다. 예시로는 자바에서의 forEach가 있다.

### 스트림의 패러다임

스트림의 패러다임은 **계산 과정을 일련의 변환 단계로 본다는 점**이다.
각각의 단계에서, 이전 단계의 결과를 입력으로 받아 처리한다.

이 때, 변환 단계에 사용되는 함수는 순수 함수여야 한다.
순수 함수는 오직 입력만이 결과에 영향을 주는 함수를 말한다.
즉, **순수 함수는 내부 상태를 변경하지 않고(side effect가 없고), 같은 입력에 대해서는 항상 같은 결과를 반환하는 것**이다.

순수 함수는 문제를 더 작고 관리 가능한 부분으로 분리하는데 도움이 되며, 동시에 병렬로 실행할 수 있어 성능 향상에 도움이 된다.

### 무한 스트림과 유한 스트림

스트림은 크게 두 가지, **무한 스트림(Infinite Stream)과 유한 스트림(Finite Stream)**으로 구분할 수 있다.

- 유한 스트림 (Finite Stream)
유한 스트림은 사전에 정의된 자료구조(예: 리스트, 배열 등)의 요소를 처리하는 데 사용되는 스트림이다. 유한 스트림은 **특정 개수의 요소를 가지며, 이 요소들을 순차적 혹은 병렬적으로 처리**할 수 있다.
    
    ```java
    List<String> names = Arrays.asList("John", "Kim", "Lee", "Park");
    Stream<String> namesStream = names.stream();
    ```
    
- 무한 스트림(Infinite Stream)
반면, 무한 스트림은 **요소의 개수가 끝나지 않는 스트림**을 말한다. 
주로 Stream.iterate나 Stream.generate와 같은 함수를 통해 생성된다.
이런 무한 스트림을 제어하기 위해 **limit() 함수와 같이 스트림의 크기를 제한하는 연산을 사용**한다.
    
    ```java
    Stream<Integer> infiniteStream = Stream.iterate(0, n -> n + 1);
    infiniteStream.limit(10).forEach(System.out::println);  // 0부터 9까지 출력
    ```
    
- 무한 스트림임에도 불구하고 중간 연산이 가능한 이유 - 지연 수행
    
    무한 스트림에서도 중간 연산을 할 수 있는 핵심 이유는 **'지연 연산'** 때문이다.
    
    **중간 연산자는 스트림에 대한 변환을 정의하기만 하고, 실제로 연산을 수행하지는 않는다.**
    
    즉, 중간 연산자는 필터링(filter), 맵핑(map), 정렬(sort) 등과 같이 스트림의 요소를 어떻게 변환할지를 정의만 한다.
    
    실제 연산은 최종 연산자(collcet, forEach, Map…)가 호출될 때 수행된다.
    
    그래서 무한 스트림에 대해서도 중간 연산자를 사용하여 정의할 수 있으며, 최종 연산자가 호출되는 시점에서 연산이 수행된다.
    

---

## 요약

- Stream의 종단 연산인 ForEach의 경우 side effect를 낼 여지가 많으므로 생각해서 써라.
→ 여기서 사용하는 메서드들은 side effect가 없는 단순 조회류로 사용하는 것이 좋을 것!

---

## 내 생각

- 결국 CUD가 아닌 Read인 경우에 대해서 ForEach를 사용하고, 그 이외의 경우에는 순수 함수를 이용함으로써 Stream을 사용하라는 것으로 이해했다.

---

## 예제 코드

- forEach로 인한 Side Effect가 발생하는 코드

```java
@AllArgsConstructor
@Getter
@Setter
@ToString
public class Person {

	private String name;
	private int age;
}

@DisplayName("forEach를 하면 매 시행마다 side effect가 발생한다.")
	@Test
	void forEach_sideEffect() {
		List<Person> people = List.of(
				new Person("Alice", 20),
				new Person("Bob", 30),
				new Person("Charlie", 40)
		);

		people.forEach(person -> {
			person.setAge(person.getAge() + 1);
			System.out.println("people = " + people);
		});
		//people = [Person(name=Alice, age=21), Person(name=Bob, age=30), Person(name=Charlie, age=40)]
		//people = [Person(name=Alice, age=21), Person(name=Bob, age=31), Person(name=Charlie, age=40)]
		//people = [Person(name=Alice, age=21), Person(name=Bob, age=31), Person(name=Charlie, age=41)]
	}
```

- 무한 스트림의 사용

```java
@DisplayName("무한 스트림으로 출력해보기")
	@Test
	void infinite_stream() {
		Stream<Integer> infinite = Stream.iterate(0, i -> i + 3);
		infinite.limit(4).forEach(System.out::println);
		//0
		//3
		//6
		//9
	}
```

- toMap 써보기

```java
@DisplayName("toMap을 이용해보기")
	@Test
	void toMap() {
		List<Person> people = List.of(
				new Person("Alice", 20),
				new Person("Bob", 30),
				new Person("Charlie", 50)
		);

		// 매개변수가 2개인 toMap
		// 해당 stream의 원소를 기준으로 key와 value를 추출하여 Map으로 반환한다.
		Map<String, Integer> nameAndAgeMap = people.stream()
				.collect(Collectors.toMap(
						Person::getName,
						Person::getAge
				));
		System.out.println("nameAndAgeMap = " + nameAndAgeMap);
		//nameAndAgeMap = {Bob=30, Alice=20, Charlie=50}

		List<Person> people2 = List.of(
				new Person("Alice", 20),
				new Person("Alice", 30),
				new Person("Bob", 30),
				new Person("Bob", 40),
				new Person("Charlie", 50)
		);
		// 매개변수가 3개인 value 충돌시 병합 BinaryOperator를 받는 toMap
		Map<String, Person> nameToOldestPerson = people2.stream()
				.collect(Collectors.toMap(
						Person::getName,
						person -> person,
						(existingValue, newValue) -> existingValue.getAge() > newValue.getAge() ? existingValue : newValue
				));

		System.out.println(nameToOldestPerson);
		//{Bob=Person(name=Bob, age=40), Alice=Person(name=Alice, age=30), Charlie=Person(name=Charlie, age=50)}
	}
```

- groupingBy 써보기

```java
@DisplayName("groupingBy 이용해보기")
	@Test
	void test() {
		List<Person> people = List.of(
				new Person("Sangje", 20),
				new Person("Sangjai", 20),
				new Person("Sangjin", 30),
				new Person("Sangjing", 30),
				new Person("Eomma", 50),
				new Person("Appa", 50)
		);

		Map<Integer, List<Person>> peopleByAge = people.stream()
				.collect(Collectors.groupingBy(Person::getAge));
		System.out.println("peopleByAge = " + peopleByAge);
		//{50=[
		// Person(name=Eomma, age=50),
		// Person(name=Appa, age=50)],
		// 20=[Person(name=Sangje, age=20),
		// Person(name=Sangjai, age=20)],
		// 30=[Person(name=Sangjin, age=30),
		// Person(name=Sangjing, age=30)]}
		// {나이 = [Person의 배열]}
	}
```

- joining 써보기

```java
@DisplayName("joining 써보기")
	@Test
	void joining_test() {
		List<Person> people = List.of(
				new Person("Alice", 20),
				new Person("Bob", 30),
				new Person("Charlie", 50)
		);

		String names = people.stream()
				.map(Person::getName)
				.collect(Collectors.joining(", "));
		System.out.println("names = " + names);
		//names = Alice, Bob, Charlie
	}
```
