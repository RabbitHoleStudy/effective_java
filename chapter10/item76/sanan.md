# 실패(예외 발생)했을 시에 원자성을 보장하도록 설계해라.

## 핵심어

### 실패 원자적(failure-atomic)

**하나의 작업에서, 작업을 이루는 동작 중 한 개라도 실패하면 그 작업은 실패하고 이전의 상태가 유지되는 것**

우리는 일련의 동작들이 한번에 이루어지는 것처럼 작성한다. 

모든 일련의 동작들은 하나의 원자적인 동작으로 취급되며(한 번의 복잡한 결제 요청 - 이에 따른 무수한 동작들), 이 동작들은 모두 성공하거나 아니면 아무것도 변경되지 않아야 한다.

우리에게 익숙한 것은 트랜잭션(ACID 원칙)이 있다.

---

## 요약

### 실패 원자적으로 작성하는 방법

1. 불변 객체로 설계하기
    
    불변 객체는 태생적으로 변할 여지가 없으므로, 실패하든 성공하든 지지고 볶든 상관이 없다. 변하지 않기 때문에.
    
2. 가변 객체의 경우, 매개변수 유효성을 먼저 검사하기 / 부작용이 있는 코드를 나중에 배치하기
    
    초입에 틀어 막아서 해당 객체의 상태가 변하기 전에 끝내는 방식으로 작성할 수 있다.
    
3. 객체의 임시 복사본에서 작업을 수행하고, 완료되면 원래 객체와 교체하기(혹은 상태를 변경하기)
    
    데이터를 임시 자료구조에 저장해 두었다가 적용하는 것이 성능적으로 더 좋을 때 주로 선택한다.
    
4. 실패를 인터셉트하여 해당 실패에 대해서 이전 상태로 돌아가게끔 코드를 작성하는 방법
    
    자주 쓰이는 방법이 아니고, (개인적인 생각으로는) 일일이 해주기에 버거운 작업일 것 같다.
    

### 예외가 발생했다면 객체의 상태는 호출 전과 동일해야 하는 것이 국룰이다

아니면 명세에 잘 써놔라 ← 근데 잘 안 지켜짐.

---

## 내 생각

책에서는 메서드 위주로 내용이 나타났지만, 결국 우리가 원자성에 대해 신경써야 하는 것은 서비스-리포지터리로 이어지는 데이터의 변화일 것이다.

단순한 객체 자체에 대한 원자성을 따지는 것보다도 비즈니스적으로 엮여있는(예를 들면 게시물과 게시물 이미지) 관계들에 대해서도 고민해봐야 한다고 생각한다. 결국 객체나 데이터 자체로서는 분리되어 있지만, 우리가 비즈니스적으로 생각하여 부여하는 논리적 관계로 하나의 묶음으로서 원자적으로 관리할 수 있어야하기 때문이다(~~DDD?!~~). 

---

## 예제 코드

- 임시 복사본은 아니지만, 가변 객체의 업데이트를 위해서 별도의 책임을 분리하고, 이에 대해 본인이 직접 수행한 다음 set하는 방식으로 구현된 코드(from dongglee)

```java
@Override
@Transactional // <----- 해당 업데이트에 대한 원자성을 보장
	public void updateProfile(Long memberId, MultipartFile profileImage, MemberUpdateRequest request) {
		Member member = memberRepository.findById(memberId)
				.orElseThrow(ExceptionStatus.NOT_FOUND_MEMBER::toServiceException);
		// 위 요약에 2번에 해당하는 경우이다. <- 비록 매개변수 자체는 아니지만 동작에 사용되는 값의 유효성을 검사한다.

		request.setMemberPolicy(memberPolicy); 
		// 밖에서 request라는 별도의 객체를 생성해서 넣어주면, 서비스에서 이 객체를 이용하여 검증을 수행한다.
		// 이 코드자체에서는 성능의 개선보다는 책임의 분리 목적(엔티티는 건드리지 않고 도메인 로직 수행)이 더욱 강하다.
		request.register(member);
		validateRequest(request);
		request.setProfileFilename(
				updateProfileImageIfPossible(member, profileImage));
		validateRequest(request);
		member = request.apply();

		memberRepository.save(member);
	}

/*--------------------------------------------------------------------*/

public class UpdateRequest<T extends IdDomain<?>> {
	private T entity = null;
	private final Queue<UpdateValidator<T>> validators = new LinkedList<>();
	private final Queue<UpdateApplier<T>> appliers = new LinkedList<>();

	/**
	 * 업데이트 요청을 처리한다.
	 * 업데이트 처리가 완료되면 업데이트된 엔티티를 반환하고 엔티티는 null로 초기화한다.
	 * 유효성 검증이 되지 않은 값이 있다면 실행할 수 없다.
	 * @return 업데이트된 엔티티
	 */
	public T apply() {
		if (!isValidated()) {
			throw new IllegalStateException("validate() must be called before apply()");
		}
		while (!appliers.isEmpty()) {
			appliers.poll().apply(getEntity());
		}
		T ret = entity;
		entity = null;
		return ret;
	}

	/**
	 * 업데이트를 할 엔티티를 등록한다.
	 * 유효성 검증이 된 값이 없다면 실행할 수 없다.
	 * @param entity 업데이트를 할 엔티티
	 */
	public void register(T entity) {
		if (!isSafe()) {
			throw new IllegalStateException("there is an unevaluated applier");
		}
		this.entity = entity;
	}

	/**
	 * 유효성 검증을 수행한다.
	 * 업데이트를 할 엔티티가 등록되지 않았다면 실행할 수 없다.
	 * @throws UpdateException
	 */
	public void validate() throws UpdateException {
		if (!isRegistered()) {
			throw new IllegalStateException("register must be called before validate()");
		}
		while (!validators.isEmpty()) {
			validators.poll().validate(entity);
		}
	}
//... 생략
```
