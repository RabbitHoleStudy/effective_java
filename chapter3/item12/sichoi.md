# toString을 항상 재정의하라

## 개요

```java
public class UserBlackholeInfoDto {

	private final Long userId;
	private final String name;
	private final String email;
	private final LocalDateTime blackHoledAt;
}
```

```java
class Test {

	public static void main(String[] args) {
		UserBlackholeInfoDto userBlackholeInfoDto = new UserBlackholeInfoDto(1L, "name", "email",
				LocalDateTime.now());
		System.out.println(userBlackholeInfoDto);
	}
}
```

<img width="558" alt="스크린샷 2023-07-15 오후 10 48 50" src="https://github.com/TightJava/effective_java/assets/83565255/666ca567-5fd4-4ec6-b866-9a1694291597">


`toString`을 잘 모를 때, 클래스의 인스턴스 내용을 출력 하려면 어떻게 해야하는지 잘 몰랐었다.

유저의 블랙홀 날짜 정보를 제공하는 DTO 클래스를 생성하고, System.out.println으로 찍어보면 저런식으로 해시된 값만 확인이 되고, 내용이 찍히지 않아 의아했었다.

## Object의 toString

이는 Object의 toString이 저런 값을 출력하도록 정의되어 있기 때문이다..!

`Object`의 기본 `toString`은 대부분의 경우 우리가 원하는 문자열을 반환해주지 않는다.

이 메소드는 **PhoneNumber@adbbd**와 같이 단순히 **클래스명@16진수의해시코드** 형태로 반환해준다.

사실 클래스의 인스턴스 값을 출력해보려는 이유는 이 안에 어떤 내용들이 들어있는지를 보려는 것이다.

그렇기 때문에 클래스를 정의할 때, toString을 Override해서 원하는 값을 출력할 수 있도록 재정의하는 것이 좋다.

실제로 `toString`의 규약에서도 **모든 하위 클래스에서 이 메소드를 재정의하라**고 되어 있다!

```java
public class UserBlackholeInfoDto {

	private final Long userId;
	private final String name;
	private final String email;
	private final LocalDateTime blackHoledAt;

	public Long getUserId() {
		return this.userId;
	}

	public String getName() {
		return this.name;
	}

	public String getEmail() {
		return this.email;
	}

	public LocalDateTime getBlackHoledAt() {
		return this.blackHoledAt;
	}

	@Override
	public String toString() {
		return "UserBlackholeInfoDto(userId=" + this.getUserId() + ", name=" + this.getName()
				+ ", email=" + this.getEmail() + ", blackHoledAt=" + this.getBlackHoledAt() + ")";
	}
}
```
<img width="840" alt="스크린샷 2023-07-15 오후 10 57 54" src="https://github.com/TightJava/effective_java/assets/83565255/e12aa055-ec6b-45e5-96ee-35f1e0a52618">

`toString`을 재정의해서 클래스 안에 내용을 출력한 예시이다. 

굳이 필요 없는 해시 값 대신 우리가 원하는 값들을 한 눈에 알아볼 수 있게 되었다!

## 로그를 남길 때 유용

이는 디버깅 용도 뿐만 아니라 프러덕션 환경에서 로그를 남길 때도 유용하게 적용될 수 있다.

```java
public void handleBlackhole(UserBlackholeInfoDto userBlackholeInfoDto) {
		log.debug("called handleBlackhole {}", userBlackholeInfoDto);

//  ... 블랙홀 처리
}
```

예를 들어 다음과 같이 유저의 블랙홀 정보를 바탕으로 어떤 처리를 하는 코드를 작성할 수 있을 것이다.

이때, **블랙홀 날짜가 언제**인 **어떤 유저**에 대해 **언제** 이 메소드를 호출했는지 로그를 남기기 굉장히 유용해진다..!

## 의도를 명확히!

toString을 구현한다고 했을 때, 반환값의 포맷을 명시할지 말지 정해야 한다.

```java

/**
 * 이 전화번호의 문자열 표현을 반환한다.
 * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
 * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
 * 각각의 대문자는 10진수 숫자 하나를 나타낸다.
 * 
 * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
 * 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
 * 전화번호의 마지막 네 문자는 ''0123''이 된다.
 */
@Override 
public String toString() { 
	returnstring.format( ''%03d-%03d-%04d'',
	areaCode, prefix, lineNum);
}
```

책에 나온 예시가 제일 직관적인 것 같아, 책의 예시로 가져왔다.

이런식으로 `PhoneNumber` 클래스의 문자열 반환 값에 대한 포맷을 명시하여 작성할 수 있을 것이다.

포맷을 미리 맞춘다면, 추후 이 포맷에 얽매이게 될 수 있고, 포맷을 추후 변경하려고 시도한다면 사용되는 코드나 이미 만들어진 데이터를 모두 변경해야 하는 문제점이 발생할 수 있다는 것을 주의해야 한다.

```java
/**
 * 이 약물에 대한 대략적인 설명을 반환한다.
 * 다음은 이 설명의 일반적인 형태이지만,
 * 상세 형식은 정해지지 않았으며 추후 변경될 수 있다.
 * 
 * "[약물 #9: 유형=사랑, 냄새=테레빈유, 겉모습=먹물]"
 */
public String toString() {
	...
}
```

이런식으로 포맷을 따로 지정하지 않은 경우도 있을 것이다. 

이런 경우 향후 릴리스에서 추가적인 정보를 넣거나 포맷을 개선하는 등 유연하게 대처할 수 있다.

포맷을 지정하든 지정하지 않든 주석으로 **의도를 명확히 밝혀야 한다.**

## 추가 고려사항

- 포맷 명시 여부랑 무관하게 **toString이 반환할 값에 포함된 정보를 얻어올 수 있는 API를 제공**해야 한다.

예를 들어 PhoneNumber 클래스의 areaCode, prefix, lineNum를 포맷을 정해 toString으로 제공해주고 있지만 이들 각각을 얻을 수 있는 API가 있어야 한다.

그렇지 않다면 toString으로 얻어진 정보를 파싱해야 하는 문제가 발생할 수 있다.

- **정적 유틸리티 클래스**는 toString을 제공할 이유가 없다.
- **열거 타입** 역시 자바가 toString을 제공하기 때문에 재정의 할 필요 없다.
- **하위 클래스들이 공유해야할 문자열 표현이 있는 추상 클래스**는 toString을 재정의해야 한다.
    - **AbstractColletion의 예시 보러가기**
        
        ```java
        public abstract class AbstractCollection<E> implements Collection<E> {
        	public String toString() {
        	        Iterator<E> it = iterator();
        	        if (! it.hasNext())
        	            return "[]";
        	
        	        StringBuilder sb = new StringBuilder();
        	        sb.append('[');
        	        for (;;) {
        	            E e = it.next();
        	            sb.append(e == this ? "(this Collection)" : e);
        	            if (! it.hasNext())
        	                return sb.append(']').toString();
        	            sb.append(',').append(' ');
        	        }
        	    }
        }
        ```
        
        AbstractCollection는 컬렉션(List, Deque, ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Stack 등등등~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~)들에 대한 추상 클래스 역할을 수행한다.
        
        이러한 컬렉션들의 내용을 출력할 때, `[1, 2, 3 …]`과 같은 형태로 표현해주는 것이 직관적일 것이다.
        
        따라서 아예 `AbstarctCollection`에서 toString을 재정의하여 공통의 표현을 제공해주고 있다!
        
        ```java
        public class ArrayDeque<E> extends AbstractCollection<E>
                                   implements Deque<E>, Cloneable, Serializable;
        ```
        
        ```java
        class Test {
        
        	public static void main(String[] args) {
        		ArrayDeque<Integer> arrayDeque = new ArrayDeque<>();
        		arrayDeque.add(1);
        		arrayDeque.add(2);
        		System.out.println(arrayDeque);
        	}
        }
        ```
        
<img width="739" alt="스크린샷 2023-07-15 오후 11 34 56" src="https://github.com/TightJava/effective_java/assets/83565255/6b3063e3-1d41-4259-8958-27252602be5b">

        `ArrayDeque`는 `AbstractCollection`를 상속받고 있다. `AbstrractCollection`이 이미 toString을 재정의해두었기 때문에, 편하게 이쁜 형태로 출력을 확인할 수 있다..!!
        

AutoValue 프레임워크가 toString도 생성해준다. 이는 필드 내용을 깔끔하게 표현시켜주긴 하지만 클래스의 **의미**까지는 파악해주지 않기 때문에 포맷을 적용하기 어려울 수 있다.

자동 생성에 적합하진 않더라도, 그럼에도 객체의 값에 관해 아무것도 알려주지 않는 Object의 toString 보다는 훨씬 낫다!

## 저자의 핵심 정리

**모든 구체 클래스에서 0bject의 toString을 재정의하자.** 

상위 클래스에서 이미 알맞게 재정의한 경우는 예외다. 

toString을 재정의한 클래스는 사용하기도 즐겁고 그 클래 스를 사용한 시스템을 디버깅하기 쉽게 해준다.

toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.
