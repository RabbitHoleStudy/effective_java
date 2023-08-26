

# 요약

- 아래 계층에서 발생한 예외를 잡아서, 추상화 수준에 맞는 예외로 바꿔서 던져라
- 그대로 메세지를 전달해야할 경우 예외 번역을 해라 ( 래핑해서 줘라 )

---

- 메서드가 저수준 예외를 처리하지 않고 바깥으로 전파하면 안된다.
- 내부 구현방식을 드러내여 윗 레벨의 API를 오염시킨다.
- 다음 릴리즈에서 구현방식이 바뀌면 다른 예외가 나올 수 있다.

- 상위 계층에서 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다

⇒Exception translation 이라 한다.

---

# CABI 에서 나온 예시

## auth / service / FtApiManager

```java
public class FtApiManager {
...
/**
	 * 42 토큰을 발급받는다.
	 */
	public void issueAccessToken() {
		log.info("called issueAccessToken");
		accessToken =
				WebClient.create().post()
						.uri(ftApiProperties.getTokenUri())
						.body(BodyInserters.fromFormData(
								ApiRequestManager.of(ftApiProperties)
										.getAccessTokenRequestBodyMapWithClientSecret(
												"client_credentials")))
						.retrieve()
						.bodyToMono(String.class)
						.map(response -> {
							try {
								return objectMapper.readTree(response)
										.get(ftApiProperties.getAccessTokenName()).asText();
							} catch (Exception e) {
								throw new RuntimeException();
							}
						})
						.onErrorResume(e -> {
							throw new ServiceException(ExceptionStatus.OAUTH_BAD_GATEWAY);
						})
						.block();
	}

```

```java
/**
	 * 유저의 이름으로 42API를 통해 특정 유저의 정보를 가져온다.
	 *
	 * @param name 유저의 이름
	 * @return JsonNode 형태의 유저 정보
	 */
	public JsonNode getFtUsersInfoByName(String name) {
		log.info("called getFtUsersInfoByName {}", name);
		Integer tryCount = 0;
		while (tryCount < MAX_RETRY) {
			try {
				JsonNode results = WebClient.create().get()
						.uri(ftApiProperties.getUsersInfoUri() + '/' + name)
						.accept(MediaType.APPLICATION_JSON)
						.headers(h -> h.setBearerAuth(accessToken))
						.retrieve()
						.bodyToMono(JsonNode.class)
						.block();
				return results;
			} catch (Exception e) {
				tryCount++;
				log.info(e.getMessage());
				log.info("요청에 실패했습니다. 최대 3번 재시도합니다. 현재 시도 횟수: {}", tryCount);
				this.issueAccessToken();
				if (tryCount == MAX_RETRY) {
					log.error("요청에 실패했습니다. 최대 재시도 횟수를 초과했습니다. {}", e.getMessage());
					throw new ServiceException(ExceptionStatus.OAUTH_BAD_GATEWAY);
				}
			}
		}
		return null;
	}
```

- auth / service / FtApiManager
- 여기서 ServiceException 을 Throw 한다.

## utils/blackhole/manager

```java
public class BlackholeManager {
	
	public void handleBlackhole(UserBlackholeInfoDto userInfoDto) {
			log.info("called handleBlackhole {}", userInfoDto);
			LocalDateTime now = LocalDateTime.now();
			try {
				JsonNode jsonRefreshedUserInfo = getBlackholeInfo(userInfoDto.getName());
				if (!isValidCadet(jsonRefreshedUserInfo)) {
					handleNotCadet(userInfoDto, now);
					return;
				}
				LocalDateTime newBlackholedAt = parseBlackholedAt(jsonRefreshedUserInfo);
				log.info("갱신된 블랙홀 날짜 {}", newBlackholedAt);
				log.info("오늘 날짜 {}", now);
	
				if (isBlackholed(newBlackholedAt)) {
					handleBlackholed(userInfoDto);
				} else {
					handleNotBlackholed(userInfoDto, newBlackholedAt);
				}
			} catch (HttpClientErrorException e) {
				handleHttpClientError(userInfoDto, now, e);
			} catch (ServiceException e) {
					// 렌트한 캐비넷이 없을경우
				if (e.getStatus().equals(ExceptionStatus.NO_LENT_CABINET)) {
					userService.deleteUser(userInfoDto.getUserId(), now);
				}
				// API 에서 에러가 났을경우
				else if (e.getStatus().equals(ExceptionStatus.OAUTH_BAD_GATEWAY))
					log.info("handleBlackhole ServiceException {}", e.getStatus());
					throw new UtilException(e.getStatus()); //
			} catch (Exception e) {
				log.error("handleBlackhole Exception: {}", userInfoDto, e);
			}
		}
```

- UtilException 추가

```java
/**
 * Util에서 throw하는 exception들을 위한 exception 사용 예시:
 * <pre>
 *     {@code throw new UtilException(ExceptionStatus.NOT_FOUND_USER);}
 *</pre>
 * 만약 새로운 exception을 만들 필요가 있다면 {@link ExceptionStatus}에서 새로운 enum값을 추가하면 된다.
 * @see org.ftclub.cabinet.exception.ExceptionStatus
 */

public class UtilException extends RuntimeException {

	final ExceptionStatus status;

	/**
	 * @param status exception에 대한 정보에 대한 enum
	 */
	public UtilException(ExceptionStatus status) {
		this.status = status;
	}

}
```

```java
@Log4j2
@RestControllerAdvice
public class ExceptionController {

...
	@ExceptionHandler(UtilException.class)
	public ResponseEntity<?> utilExceptionHandler(UtilException e) {
		log.warn("[UtilException] {} : {}", e.status.getError(), e.status.getMessage());
		return ResponseEntity
				.status(e.status.getStatusCode())
				.body(e.status);
	}

}
```

- 추상화 레벨이 다르다 ⇒ ServiceException TO UtilsException 예외 번역
- 만약 ServiceException 단일로 던졌다면, 
blackhole 검사 로직에서 에러가 났는지?
Login 하고 정보를 가져오는 부분에서 에러가 난건지 ?
⇒ 추적이 어렵지 않았을까?
