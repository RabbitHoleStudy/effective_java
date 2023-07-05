## 핵심어

- 빌더 패턴
- Recirsove Type Bound(재귀적 타입 한정)
    
    > 타입 매개변수가 자신의 타입을 포함하는 상위 모듈에 한정되는 것을 말한다.
    > 
    
    무슨 말이지..? 나중에 또 나오니 그 때 알아보자..
    
- Covariant Return Typing(공변반환 타이핑)

  뭔소리인지 아직 잘 감이 안 온다..
    
    [공변반환 타이핑](https://bperhaps.tistory.com/entry/공변반환-타이핑)
    

---

## 요약

매개변수가 많으면(설령 정적 팩터리 메서드를 쓰더라도) 빌더 패턴을 고려해라.

---

## 내 생각

동의함.

---

## 예제 코드

- @Builder 없이 빌더 패턴 만들어보기

```java
@AllArgsConstructor(access = PRIVATE) // new Profile() 방지
@Getter
public class Profile {
	private static final ProfileBuilder BUILDER_INSTANCE = new ProfileBuilder(); // Profile.ProfileBuilder가 불편해..
	private final String name;
	private final String hobby;
	private final int age;

	public static ProfileBuilder builder() {
		return BUILDER_INSTANCE; // new ProfileBuilder()가 아까워서 한번 해봤다.
	}

	protected static class ProfileBuilder {
		private String name;
		private String hobby;
		private int age;

		private ProfileBuilder() { // new ProfileBuilder() 방지
		}

		public ProfileBuilder name(String name) {
			this.name = name;
			return this;
		}

		public ProfileBuilder hobby(String hobby) {
			this.hobby = hobby;
			return this;
		}

		public ProfileBuilder age(int age) {
			this.age = age;
			return this;
		}

		public Profile build() {
			Profile profile = new Profile(name, hobby, age);
			this.age = 0;
			this.name = null;
			this.hobby = null;
			return profile;
		}
	}
}

/*-------------------------------------------------------*/

@Test
	void 빌더_테스트() {
		Profile profile = Profile.builder()
				.name("홍길동")
				.hobby("테니스")
				.age(30)
				.build();

		assertEquals("홍길동", profile.getName());
		assertEquals("테니스", profile.getHobby());
		assertEquals(30, profile.getAge());

		Profile profile2 = Profile.builder().name("금춘향").build();

		assertEquals("금춘향", profile2.getName());
		assertEquals(null, profile2.getHobby());
		assertEquals(0, profile2.getAge());
	}
```

왜 내 Builder는 Profile.ProfileBuilder로 접근이 가능할까..? 롬복은 안되던데.. 어떻게 한거지?
