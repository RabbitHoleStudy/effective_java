

# 직렬화란?

- 프로그램이 돌아가면서, 메모리상에 존재하는 데이터를 영속화 하는 행위
- 프로그램에서 사용되는 데이터나 객체들 → 파일 or 네트워크 전송을 위해 바이트 형태로 변환하는 과정

- 파일 저장, Jackson Library ObjectMapper
- Database
- 서버에서 클라이언트로 전달하는 JSON

⇒ 이런 것들이 직렬화라고 할 수 있다. ( Serialization )

JSON → Object : 역직렬화 (Deserialization)

# Java 객체 직렬화

- Object Serilaization ⇒ 자바 객체(Object) 를 바이트 스트림으로 변환하는 기술
- [java.io](http://java.io) 패키지의 Serializable 인터페이스를 구현하면 된다.

## 직렬화 과정

- ObjectOutputStream Class → 객체를 바이트 스트림으로 변환하여 저장하는 역할
객체를 출력 스트림에 쓴다

## 역직렬화 과정

- ObjectInputStream 클래스로, 바이트 스트림을 읽어와 객체로 변환한다.

# 자바 직렬화의 대안을 찾으라

## 요약

- 직렬화는 취약점이 많다
- 직렬화 - 역직렬화는 쓰지말고 대안을 찾자
- JSON 써라

---

## 문제점

- 역직렬화 과정 → ObjectInputStream - readObject
- readObject 메서드는 클래스패스 안의 거의 모든 타입의 객체를 만들어 낼 수 있는 마법같은 생성자다
- 바이트 스트림 → 객체
역직렬화 과정에서 이 메서드는 타입들 안의 모든 코드를 수행할 수 있다.
코드 전체가 공격 범위에 들어간다.

## 가젯 gadget

- 서드파티 라이브러리에서 직렬화 가능 타입들의 역직렬화 과정에서 호출되어 잠재적으로 위험한 동작을 수행하는 메서드
- 공격자가 기반 하드웨어의 네이티브 코드를 마음대로 실행할 수 있는 가젯 체인도 발견되곤 한다.
- 공격 시나리오
    - 역직렬화 는 `ac ed 00 05` 패턴과 base64 인코딩에서는  `rO0AB` 로 메모리 주소가 시작된다. 따라서 직렬화된 데이터가 전송되는 부분을 알아 낼 수 있다.
    - 직렬화 전송 뒤에, `nc -lvnp 4000` 와 같은 코드를 페이로드에 삽입하여, 역직렬화 하는 과정에서 서버가 실행되도록 만든다. (4000번 포트에서 연결을 기다리고 데이터를 받을 준비를 한다)
        - 상세 옵션
            - **`nc`**: 네트워크 연결을 생성하거나 관리하기 위한 명령어입니다. 'netcat'라고도 불립니다.
            - **`l`**: Listen 모드를 의미하며, 다른 컴퓨터에서 연결을 기다리고 데이터를 받을 준비를 합니다.
            - **`v`**: Verbose 모드로, 더 많은 정보를 자세하게 출력합니다.
            - **`n`**: 호스트 이름을 해석하지 않고 IP 주소 형식으로 사용합니다.
            - **`p 4000`**: 리스닝할 포트 번호를 4000으로 지정합니다. 즉, 4000번 포트에서 연결을 기다립니다.

---

## 역직렬화 DDOS

- 역직렬화 폭탄 deserialization bomb
- 역직렬화에 오래걸리는 스트림으로 공격하는 기법

```jsx
public class DeserializationBomb {

	public static void main(String[] args) {
		System.out.println(bomb().length);
		bomb();
	}
	static byte[] bomb() {
		Set<Object> root = new HashSet<>();
		Set<Object> s1 = root;
		Set<Object> s2 = new HashSet<>();
		for (int i =0; i < 100; i++){
			Set<Object> t1 = new HashSet<>();
			Set<Object> t2 = new HashSet<>();
			t1.add("foo"); // t1을 t2와 다르게 만든다.
			s1.add(t1); s1.add(t2);
			s2.add(t1); s2.add(t2);
			s1 = t1;
			s2 = t2;
		}
		return serialize(root);
	}

	public static byte[] serialize(Object o) {
		ByteArrayOutputStream ba = new ByteArrayOutputStream();
		try {
			new ObjectOutputStream(ba).writeObject(o);
		} catch (IOException e) {
			throw new IllegalArgumentException(e);
		}
		return ba.toByteArray();
	}

	public static Object deserialize(byte[] bytes) {
		try {
			return new ObjectInputStream(
					new ByteArrayInputStream(bytes)).readObject();
		} catch (IOException | ClassNotFoundException e) {
			throw new IllegalArgumentException(e);
		}
	}

}
```

- bomb() 함수는 root 라는 HashSet 생성
    - 이어서 루프를 100번 반복하면서 2개의 중첩된 HashSet 객체인 s1 과 s2 에 객체를 추가
    - 이 때 t1 객체를 생성해서 “foo” 문자 추가, t2 객체를 생성해서 추가하지 않는다.
    - t1 과 t2는 다른 객체다.
    - s1 → t1 
    s2 →t2 로 갱신하면서 서로 다른 객체들을 참조하도록 변경
    - root HashSet 객체를 반환하고, 이 객체를 serialize로 바이트로 배열로 반환

⇒ 위 메소드를 역직렬화 하게된다면??

- 결과 콜스택
    
    ![159151847-43326cd9-9a02-48c8-99fe-25912663bbd7.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/da9fff23-8987-42c2-81bd-d5da1b25e66b/159151847-43326cd9-9a02-48c8-99fe-25912663bbd7.png)
    
- 2^100 의 hashCode() 메서드가 호출된다.
- 역직렬화시 Hashcode가 변경될 수 있기때문에, 동등함을 증명하기 위해서 hashCode 계산을 한다.

- 믿을수 없는 객체는 직렬화 - 역직렬화 하지말자
- 우리가 쓰는 시스템에서 자바 직렬화를 써야할 이유는 전혀 없다.
- 그냥 하지마

---

- 직렬화의 위험을 회피하면서, 다양한 플랫폼끼리의 데이터 전송을 지원하는 도구가 너무나도 많다.

⇒ 크로스-플랫폼 구조화된 데이터 표현 ( cross-platform structred-data representation)

- 속성 - 값 쌍의 집합으로 간단하고 구조화된 데이터 객체를 사용한다.

# 크로스-플랫폼 구조화된 데이터 표현

- Protocol Buffers - protobuf → C++ 용
    - 이진 표현
    - 텍스트 표현(pbtxt)
    - 효율좋음
- JSON → Javascript용
    - 텍스트 기반

## 굳이 써야한다면

java.io.ObjectInputFiler 를 이용해라 (java 9 +)

- 스트림 역직렬화 전에 필터를 설치하는 기능
- 클래스 단위로, 받거나 거부가 가능
- 블랙리스트보다는 화이트리스트로 쓰라
