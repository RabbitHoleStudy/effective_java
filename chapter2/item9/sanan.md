## 핵심어

- try-finally
try-finally는 예외 발생 여부와 관계없이 항상 실행되어야 하는 코드를 정의할 때 사용된다. 일반적으로 파일, 네트워크 연결, 데이터베이스 연결과 같은 리소스를 사용하는 경우에 try-finally 구문을 사용하여 리소스를 명시적으로 닫아준다.
- try-with-resources
**`try-with-resources`**는 리소스를 명시적으로 닫지 않아도 자동으로 리소스를 해제해주는 기능을 제공한다. 이를 위해 리소스는 **`AutoCloseable`** 인터페이스를 구현해야 한다.

---

## 요약

close 해야 하는 자원에 대해서 AutoCloseable 인터페이스를 구현하고, Java 7부터 지원하는 try-with-resources를 이용해 예쁘고 좋은 코드를 작성하자.

---

## 내 생각

할당과 해제가 있듯이, 사용하려고 open한 자원도 close해야하는 거고, 이미 42 과제들을 하면서 충분히 겪어보았다. 외부 라이브러리를 사용하거나 새로이 구현할 때, AutoCloseable을 염두에 두고 해봐야할 것 같다.

---

## 예제 코드

```java
public class TryWithResources {
	public String readFirstLineFromFile(String path) {
		try (BufferedReader br = new BufferedReader(new FileReader(path))) {
			return br.readLine();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			throw new RuntimeException("finally 블록에서도 예외가 발생하면 try 블록에서 발생한 예외는 무시된다.");
		}
	}
}
```
