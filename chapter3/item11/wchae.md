# [아이템 11] equals 재정의시 hashCode도 재정의

# 해시코드 작성요령

1.  int 변수 result 를 선언한 후, c 값으로 초기화한다
- c : 객체의 첫번째 핵심 필드(equals 비교시 비교하는 필드)를 해시코드로 계산한것
1. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다
    1. 해당 필드의 해시코드 c 를 계산한다
        1. 기본 타입 필드라면, Type.hashCode(f) (Type 은 Boxed클래스)
        2. 참조타입 필드
        이 필드의 equals 를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출
        필드 값이 null 이면 0을 사용한다
        복잡해질것 같으면 필드의 표준형(canonical representation 을 만들어 그 표준형의 hashCode 호출
        3. 필드가 배열이라면 핵심 원소 각각을 필드처럼 다룬다, 
        원소가 없다면 0
        모든 원소가 핵심원소라면 Arrays.hashCode 사용
    2. 단계 2.a 에서 계산한 해시코드 c로 result 를 갱신한다
    `result = 31 * result + c;`
2. Result 반환

## 예시

```java
@0verride 
public int hashCode() 
{ 
	int result = Short.hashCode(areaCode);  //areaCode : 필드
	result= 31 * result + Short.hashCode(prefix) ; 
	result= 31 * result + Short.hashCode(lineNum) ; 
	return result;
}
```

## 대체품

```java
@0verride public int hashCode() 
	{ 
		return Objects.hash(lineNum, prefix, areaCode) ;
}
```

- 한줄짜리 hash
- 하지만 속도가 느리다.
- 입력인수를 담기위한 배열이 만들어지고, 입력중 기본타입이 있으면 박싱-언박싱도 거친다
- 성능에 민감하지 않으면 사용

## 지연초기화가 필요한경우

```java
private int hashCode; // 자동으로 0으로 초기화된다.
@0verride public int hashCode( ) { 
	int result = hashCode;
		if (result==0) {
		result= Short.hashCode(areaCode); 
		result= 31 * result + Short.hashCode(prefix); 
		result= 31 * result + Short.hashCode(lineNum); 
		hashCode=result;
		}
	return result;
}
```

---

필드의 hascode 표준형을 만드는 방법
