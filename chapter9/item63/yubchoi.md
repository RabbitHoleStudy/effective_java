# 문자열 연결은 느리니 주의하라

**📌 문자열 연결 연산자(+)는 성능 저하를 불러온다. StringBuilder의 append 메서드를 사용하자.**

### 문자열 연결 연산자는 성능 저하를 불러온다

문자열 연결 연산자로 문자열 n개를 잇는 시간은 n제곱에 비례한다. 문자열은 불변이라서 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사해야 하기 때문에 성능이 저하된다.

```java
public String statement() {
	String result = "";
	for (int i = 0; i < 10; i++) {
		result += lineForItem(i);
	}

	return result;
}
// run time : 17066792ns
```

**String 대신 StringBuilder를 사용하자.**

```java
public String statement2() {
	StringBuilder b = new StringBuilder();
	// StringBuilder b = new StringBuilder(10 * LINE_WIDTH);
	for (int i = 0; i < 10; i++) {
		b.append(lineForItem(i));
	}
	return b.toString();
}
// run time : 107500ns => 기본값을 사용한 경우
// run time : 90791ns => 적절한 크기로 초기화한 경우
```
