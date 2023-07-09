
## 불필요한 객체 생성을 피하라

### new 대신 리터럴

```java
String s = new String("대우리는daewoolee가아니라daewoole");
```

```java
String s = "대우리는daewoolee가아니라daewoole";
```

리터럴을 사용하면 모든 코드가 같은 객체 사용이 보장됨. JLS, 3.10.5

### 생성자 대신 팩토리 메소드

```java
String s = "오늘안온박재범씨";
Boolean(s);
```

자바 9부터 deprecated API로 지정됨.

```java
String s = "오늘안온박재범씨";
Boolean.valueOf(s);
```

### 비싼 객체는 캐싱해서

```java
static boolean isRomanNumberal(String s) {
    return s.matches("M{0,3}(CM|CD|D?C{0,3})(XC|XL|L?X{0,3})(IX|IV|V?I{0,3})");
}
```

```java
public class RomanNumberals {
  private static final Pattern ROMAN = Pattern.compile(
      "^(?=[MDCLXVI])M*(C[MD]|D?C{0,3})"
          + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

  static boolean isRomanNumberal(String s) {
    return ROMAN.matcher(s).matches();
  }
}
```

Pattern 인스턴스를 정적으로 초기화해서 캐싱해두고, isRomanNumberal 메소드가 호출될 때마다 이 인스턴스를 재사용.

저자의 상황에서는 길이가 8인 문자열을 입력했을 때, 개선 전 1.1us에서 개선 후 0.17us로 6.5배정도 빨라졌다함.

Pattern 인스턴스가 static final 필드로 빠짐으로써 의미도 더 잘 드러남.

isRomanNumeral이 처음 호출될 대 필드를 초기화시키는 Lazy initialization을 이용하여 불필요한 초기화를 없앨 순 있지만 성능이 크게 개선되진 않아서 권장되지 않음.

### 오토박싱

프로그래머가 기본 타입(int)과 박싱된 기본 타입(Integer)를 섞어서 쓸 때 자동으로 상호 변환해주는 기술

의미상으론 기본 타입과 박싱된 기본 타입을 섞어써도 차이가 없지만 성능면에서는 차이가 큼.

```java
private static long sum() {
  Long sum = 0L;
  for (long i = 0; i < Integer.MAX_VALUE; i++) {
    sum += i;
  }
  return sum;
}
```

### ⚠️ 주의

객체 생성은 비싸니까 만들지 말아야지~

→ 이건 잘못된 것임

요즘 JVM은 짱짱해서 primitive한 작은 객체를 생성하고 회수하는 일이 부담되지 않음.

조금 더 명확하고 간결하고 이해하기 쉬운 코드를 짜기 위해 객체를 추가로 생성하는 것은 오히려 좋은 일임.

단순히 오로지 객체 생성을 피하려는 목적으로만 객체 pool을 만들어 관리하지 말자

객체 pool을 만들어 관리하는게 좋은 상황도 있음.

ex. DB 연결을 위한 객체같이 워낙 비용이 비싼건 재사용하는게 좋음.

근데 일반적인 상황에서는 괜히 메모리 사용양을 늘리고 성능을 떨어뜨리고, 코드도 어렵게 만들게 함.

Item 50(새로운 객체를 만들어야 한다면 기존 객체를 재사용하지 마라)의 defensive copy와는 조금 대조적인 챕터임. 방어적 복사에 실패하면 버그 펑펑 보안 문제. but 불필요한 객체 생성은 코드 형태와 성능에만 문제.
