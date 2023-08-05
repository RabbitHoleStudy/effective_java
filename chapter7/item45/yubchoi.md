## 스트림은 주의해서 사용하라
### 요약

- 스트림 API를 제대로 사용하면 프로그램이 짧고 깔끔해지지만, 잘못 사용하면 읽기 어렵고 유지보수도 힘들어진다.
- 스트림과 반복문을 적절히 조합해서 사용하자.
- 둘 중 어느 쪽이 더 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 선택하자.
---

### 스트림 API

- 스트림 API의 추상 개념 중 핵심이 되는 2가지
    - `stream`. 데이터 원소의 유한 혹은 무한 시퀀스
    - `stream pipeline`. 이 원소들로 수행하는 연산 단계
- 스트림 안의 데이터 원소들은 객체 참조나 기본 타입 값(int, long, double)이다.

### 스트림 파이프라인
![stream_pipeline](https://github.com/TightJava/effective_java/assets/65652094/a2eb5300-7bb4-4868-b7ab-773a28881cc0)
https://dev.to/ggorantala/how-does-streams-api-work-a-deep-dive-on-streamoperation-flow-4e07

스트림 파이프라인은 소스 스트림에서 시작해 0개 이상의 중간 연산을 거쳐 종단 연산으로 끝난다.

- 중간 연산(Intermediate Operation)   
  각 중간 연산은 스트림을 어떠한 방식으로 변환한다. 변환된 스트림의 원소 타입은 변환 전 스트림의 원소 타입과 같을 수도 있고 다를 수도 있다.
  ex) 각 원소에 함수 적용, 특정 조건을 만족하지 못하는 원소 걸러내기
    
- 종단 연산(Terminal Operation)   
  마지막 중간 연산이 내놓은 스트림에 최후의 연산을 가한다.
  ex) 원소를 정렬해 컬렉션에 담기, 특정 원소 하나 선택, 모든 원소 출력
    
</br>

스트림 파이프라인은 `지연 평가(lazy evaluation)`된다. 종단 연산이 호출될 때 연산이 이뤄짐으로써 무한 스트림을 다룰 수 있게 한다. 종단 연산이 없는 스트림 파이프라인은 아무 일도 하지 않는 no-op 명령어와 같으므로 종단 연산을 잊지 말자.
- 지연 평가(lazy evaluation)   
  결과 값이 필요할 때까지 계산을 늦추는 기법. 불필요한 연산을 의도적으로 수행하지 않음으로써 실행 속도를 높일 수 있다. 예를 들어 `limit`을 사용해 불필요한 연산을 생략할 수 있다.
  ![lazy_evaluation](https://github.com/TightJava/effective_java/assets/65652094/2ea453e4-0c84-4729-acc5-af178282e362)
  https://www.logicbig.com/tutorials/core-java-tutorial/java-util-stream/lazy-evaluation.html
</br>

![stream_example](https://github.com/TightJava/effective_java/assets/65652094/bfce9f08-aa96-477e-bad8-1a6b9e62fa4e)
https://www.geeksforgeeks.org/java-8-stream-tutorial/

```java
// Stream operations
List<Integer> transactionsIds =
		transactions.stream()
				.filter(t -> t.getType() == Transaction.GROCERY)    // Grocery만 필터링
				.sorted(comparing(Transaction::getValue).reversed())    // 내림차순 정렬
				.map(Transaction::getId)    // id만 추출
				.collect(toList());    // 필터링된 요소를 리스트로 변환
```

스트림 API는 메서드 연쇄를 지원하는 fluent API다. 즉, 파이프라인 하나를 구성하는 모든 호출을 연결하여 단 하나의 표현식으로 완성할 수 있고, 파이프라인 여러 개를 연결해 표현식 하나로 만들 수도 있다.

기본적으로 스트림 파이프라인은 순차적으로 수행된다.   
</br>

**스트림 API를 제대로 사용하면 프로그램이 짧고 깔끔해지지만, 잘못 사용하면 읽기 어렵고 유지보수도 힘들어진다.**

- 스트림을 과하게 사용한 예제 코드
    
    ```java
    try (Stream<String> words = Files.lines(dictionary)) {
    	words.collect(
    					groupingBy(word -> word.chars().sorted()	// 애너그램을 수집
    							.collect(StringBuilder::new,
    									(sb, c) -> sb.append((char) c),
    									StringBuilder::append).toString()))
    			.values().stream()	// 맵의 값들을 스트림으로 변환
    			.filter(group -> group.size() >= minGroupSize)	// 충분히 큰 그룹만 필터링
    			.map(group -> group.size() + ": " + group)	// 결과를 문자열로 변환
    			.forEach(System.out::println);	// 결과 출력
    }
    ```
    
- **스트림을 적절하게 활용하자**
    
    ```java
    try (Stream<String> words = Files.lines(dictionary)) {
    	words.collect(groupingBy(Anagrams::alphabetize))    // 애너그램을 수집
    			.values().stream()    // 맵의 값들을 스트림으로 변환
    			.filter(group -> group.size() >= minGroupSize)    // 충분히 큰 그룹만 필터링
    			.forEach(group -> System.out.println(group.size() + ": " + group));    // 결과 출력
    }
    
    /* alphabetize */
    private static String alphabetize(String s) {
    	char[] a = s.toCharArray();
    	java.util.Arrays.sort(a);
    	return new String(a);
    }
    ```
    
    - 도우미 메서드로 분리해 가독성을 높였다.
    - 람다에서는 타입 이름을 자주 생략하므로 매개변수의 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다.
    - alphabetize는 단어를 알파벳 순으로 정렬하는 메서드인데, 스트림으로 구현한다면 명확성이 떨어지고 잘못 구현할 가능이 커진다. 자바가 기본 타입인 char용 스트림을 지원하지 않기 때문에, **char 값들을 처리할 때는 스트림을 삼가자.**
    
**💡 기존 코드는 스트림을 사용하도록 리팩토링하되, 새 코드가 나아 보일 때만 반영하자**

### 스트림 파이프라인과 코드 블록
되풀이되는 계산을 스트림 파이프라인은 람다나 메서드 참조 같은 함수 객체로 표현한다. 반면, 반복 코드에서는 코드 블록을 사용해 표현한다.
- 함수 객체로는 할 수 없고, 코드 블록으로는 할 수 있는 일
    - 범위 안의 지역 변수를 읽고 수정하기
        람다에서는 final이거나 사실상 final인 변수만 읽을 수 있고, 지역변수를 수정하는 것은 불가능함
    - return / break / continue를 통한 블록 흐름 제어
    - 메서드 선언에 명시된 검사 예외 던지기
- 스트림으로 처리하기 좋은 일
    - 원소들의 시퀀스를 일관되게 변환 (`map`)
    - 원소들의 시퀀스를 필터링 (`filter`)
    - 원소들의 시퀀스를 하나의 연산을 사용해 결합 (더하기, 연결하기, 최솟값 구하기 등)
    - 원소들의 시퀀스를 컬렉션에 모음 (`collect`)
- 스트림에서 처리하기 어려운 일   
  한 데이터가 파이프라인의 여러 단계(stage)를 통과할 때 이 데이터의 각 단계에서의 값들에 동시에 접근하기는 어렵다. 스트림 파이프라인은 일단 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃는 구조이기 때문이다.
