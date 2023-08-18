# 일반적으로 통용되는 명명 규칙을 따르라
**📌 철자 규칙은 특별한 이유가 없는 한 지키자. 문법 규칙은 유연하게 지키자. 오랫동안 따라온 규칙과 충돌한다면 상식적으로 해결하자.**

명명 규칙은 대부분 [자바 언어 명서](https://docs.oracle.com/javase/specs/jls/se7/html/jls-6.html)에 기술되어 있다.

## 철자 규칙
### 패키지와 모듈
- 각 요소를 `.`으로 구분하여 계층적으로 짓는다.
- 모두 소문자 알파벳 혹은 숫자로 이뤄진다.
- 조직 바깥에서도 사용될 패키지라면 조직의 인터넷 도메인 이름을 역순으로 사용한다. ex) com.google, org.ftclub
- 패키지 이름의 나머지는 해당 패키지를 설명하는 하나 이상의 요소로 이뤄진다. 각 요소는 일반적으로 8자 이하의 짧은 한 단어로 한다.
- 많은 기능을 제공하는 경우는 계층을 나눠 구성한다. ex) java.util.concurrent.TimeUnit

### 클래스와 인터페이스
- 하나 이상의 단어로 이뤄지며, 각 단어는 대문자로 시작한다. ex) AdminUserController, AdminRole
- 약어는 널리 통용되는 것(max, min 등)이 아니라면 사용을 지양한다.
- 열거 타입과 어노테이션도 해당된다.

### 메서드와 필드
- 클래스 명명 규칙과 같다. 단, 첫 글자를 소문자로 쓴다. ex) getCabinetInfo
- 첫 단어가 약자라면 단어 전체가 소문자여야 한다.
- 단, 상수 필드는 예외적으로 모든 글자를 대문자로 쓰고 단어 사이를 밑줄로 구분한다.   
  상수 필드는 값이 불변인 static final 필드를 말한다. `static final` 필드이면서, 가리키는 객체가 불변이라면 비록 그 타입은 가변이더라도 상수 필드이다. 즉, `static final` 타입이 기본 타입이나 불변 참조 타입이라면 상수 필드에 해당한다.
    ex) MAX_RETRY

### 지역 변수
- 다른 멤버와 비슷한 명명 규칙이 적용된다. 단, 약어를 써도 좋다. ex) visibleNum

### 타입 매개변수
- 보통 한 문자로 표현한다.
  
| 타입 | 문자 |
| --- | --- |
| 임의의 타입 | T |
| 컬렉션 원소 타입 | E |
| 맵의 키와 값 | K, V |
| 예외 | X |
| 메서드 반환 타입 | R |
| 그 외에 임의 타입 시퀀스 | T, U, V 혹은 T1, T2, T3 |   

</br>

## 문법 규칙
철자 규칙과 달리 더 느슨하고 복잡하다.

### 클래스
- **객체를 생성할 수 있는 클래스**
    - 단수 명사나 명사구로 짓는다. ex) Thread, PriorityQueue, ChessPiece
    - 열거 타입도 해당된다.
    
- **객체를 생성할 수 없는 클래스**
    - 복수형 명사로 짓는다.
    ex) Collectors, Collections
    
- **인터페이스**
    - 클래스와 똑같이 짓거나 able/ible로 끝나는 형용사로 짓는다.
    ex) Collection, Runnable
    

### 메서드

- **동작 수행 메서드**
    - 동사나 목적어를 포함한 동사구로 짓는다.
        ex) updateCabinetGrid, createUser
        
- **booelan 값을 반환하는 메서드**
    - is나 has로 시작한다.
    - 명사나 명사구, 혹은 형용사로 끝난다.
    ex) isBlackholed, isActiveBanHistory
    
- **반환 타입이 boolean이 아니거나 해당 인스턴스의 속성을 반환하는  메서드**
    - 명사, 명사구, 혹은 get으로 시작하는 동사구로 짓는다.
    ex) getCabinet, findAllClubUser
    
- **객체 타입을 바꿔서 다른 타입의 또 다른 객체를 반환하는 인스턴스 메서드: `toType`**   
  ex) toString, toArray
    
- **객체의 내용을 다른 뷰로 보여주는 메서드: `asType`**   
    ex) asList
    
- **객체의 값을 기본 타입 값으로 반환하는 메서드의 이름: `typeValue`**   
    ex) intValue
    

### 필드
- 클래스, 인터페이스, 메서드 문법 규칙에 비해 덜 명확하고 덜 중요하다.
- **boolean 타입의 필드**
    - boolean 접근자 메서드에서 앞 단어를 뺀다.
        ex) initialized, composited
- **다른 타입의 필드**
    - 명사나 명사구를 사용한다.
        ex) userId, name
