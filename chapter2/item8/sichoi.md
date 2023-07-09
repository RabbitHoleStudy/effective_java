# finalizer와 cleaner 사용을 피하라

자바는 객체 소멸자가 2가지 있다.

## finalizer

그 중 `finalizer`는 `예측이 어렵고 상황에 따라 위험`하기 때문에 일반적으로 불필요하다.

finalizer가 나름 쓰임을 가지긴 하지만 default는 쓰지 말아야 함이다.

자바 9에서는 finalizer를 deprecated API로 지정하고 `cleaner`를 대안으로 제시했을 정도이다. (but 자바 라이브러리에서 여전히 finalizer를 사용함)

## Cleaner

`cleaner`는 finalizer보다는 `덜 위험하지만, 여전히 예측이 어렵고, 느리기` 때문에 일반적으로 불필요하다.

자바의 finalizer와 cleaner는 C++의 destructor(소멸자)와는 다른 개념이라는 것을 주의해야 한다!

C++의 destructor는 객체의 자원 회수를 위한 보편적 방법이고, 자바는 이를 가비지 컬렉터가 담당하고, 프로그래머에게는 아무런 책임을 요구하지 않는다.

## 즉시 수행 x

둘 다 즉시 수행된다는 보장이 없다. (JLS, 12.6)

때문에 파일 close같은걸 얘네들에게 맡기게 된다면, 큰 일 날지도 모른다.

시스템에서는 동시에 열 수 있는 파일의 개수에 제한이 있기 때문이다.

(GNL 할 때 봤던 것 같은데..)

파일을 여러개 만들었는데 바로바로 close가 안된다면 새로운 파일을 열지 못해 프로그램이 뻗어버리게 될 수 있다.

## finalizer와 cleaner의 대안

파일이나 스레드 등 종료해야할 자원을 담고 있는 객체에서 finalizer나 cleaner를 대신해줄 방법.

`AutoColseable`을 구현하고 클라이언트에서 인스턴스를 다 쓰면 close 메소드를 꼭 호출하도록 함.

이때 예외가 발생하더라도 제대로 종료되도록 `try-with-resources`를 사용해야 함.

- **try-with-resources이란?**
    
    자바 7부터 도입된 기능으로, 자원 해제를 간편하게 처리할 수 있게 해줍니다. **`try-with-resources`** 문은 일반적으로 입출력 스트림, 데이터베이스 연결 등 자원을 사용하는 코드에서 유용하게 사용됩니다.
    
    **`try-with-resources`** 문은 다음과 같은 구문으로 작성됩니다:
    
    ```java
    
    try (리소스 선언) {
        // 리소스를 사용하는 코드
    } catch (예외 타입 예외변수) {
        // 예외 처리 코드
    }
    
    ```
    
    리소스 선언 부분에는 자원을 초기화하는 코드가 들어갑니다. 이 때, 리소스는 **`AutoCloseable`** 인터페이스를 구현한 클래스의 인스턴스여야 합니다. **`AutoCloseable`** 인터페이스는 **`close()`** 메서드를 정의하고 있으며, 이 메서드를 통해 리소스를 해제할 수 있습니다.
    
    **`try-with-resources`** 문을 사용하면 해당 블록이 종료될 때 자동으로 리소스가 해제됩니다. 예외가 발생한 경우에도 예외가 처리된 후에 리소스가 자동으로 해제되므로, 개발자는 명시적으로 리소스를 해제하는 코드를 작성하지 않아도 됩니다. 이를 통해 자원 관리에 대한 실수를 줄이고 코드의 가독성을 향상시킬 수 있습니다.
    

클래스에 finalizer를 달게 되면 해당 클래스의 인스턴스 자원 회수가 맘대로 지연될 수 있다.

```

```

신입 개발자(시최): 부장님… 제가 짠 GUI 프로그램이 자꾸 0utOfMemoryError 에러를 던지면서 뻗어버려요 ㅠㅠ 🥲

부장님(사난): 엥 머라구?? 또 뭔짓을 한거야!! 흠 어디보자…

<img width="514" alt="스크린샷 2023-07-08 오후 9 59 12" src="https://github.com/TightJava/effective_java/assets/83565255/bb5cc1d1-4241-4b82-980b-0ddc625ab787">
화가난 부장님

부장님(사난): 아니 바보야!! 그래픽스 클래스에다가 finalizer를 단거야..? 이거 땜에 그래픽스 객체 수천 개가 finalizer 대기열에서 줄 서고 있다가 메모리 다 써서 터졌네!!

부장님(사난): 이걸 불행이라고 해야할 지 발견됐으니 다행이라고 해야할 지… 스레드 스케줄러가 finalizer 스레드를 우선순위를 낮다고 판단해서 실행될 기회를 제대로 못 얻었었나보네!

부장님(사난): 내가 fianlizer랑 cleaner는 꼭 필요할 때만 쓰라고 얘기했건만… 뭘들은거야!! 안되겠다 오늘 너 야근해서 공부해와!!

신입 개발자(시최): 허걱.. 넴……. 🥲 (안돼 내 칼퇴…)

신입 개발자(시최): (흠 그러면 언제 필요하다고 했었더라..?)

## finalizer나 cleaner의 쓰임새

일단 상태를 영구적으로 수정하는 작업에서는 절대 쓰면 안됨.

ex. DB같은 공유 자원의 lock 해제를 얘네한테 맡기면 분산 시스템 자체가 서서히 뻗어버릴 것

### **자원의 소유자가 close 메소드를 호출하지 않는 것에 대비한 안정망 역할**

클라이언트가 하지 않은 자원 회수를 늦게라도 해줄 수 있게함.

이런 안정망 역할의 finalizer를 작성하는게 그만한 값어치가 있는지 고민 필요.

안정망 역할의 finalizer ex. **FileInputStream**, **FileOutputStream**, **ThreadPoolExecutor**

```java
package chapter2.item8;
import java.lang.ref.Cleaner;

public class Room implements AutoCloseable {
  private static final Cleaner cleaner = Cleaner.create();

  // 청소가 필요한 자원. 절대 Room을 참조해서는 안 된다!
  private static class State implements Runnable {
    int numJunkPiles; // Room 안의 쓰레기 수

    State(int numJunkPiles) {
      this.numJunkPiles = numJunkPiles;
    }

    // close 메서드나 cleaner가 호출한다.
    @Override public void run() {
      System.out.println("방을 청소합니다.");
      numJunkPiles = 0;
    }
  }

  // 방의 상태. cleanable과 공유한다.
  private final State state;

  // cleanable 객체. 수거 대상이 되면 방을 청소한다.
  private final Cleaner.Cleanable cleanable;

  public Room(int numJunkPiles) {
    state = new State(numJunkPiles);
    cleanable = cleaner.register(this, state);
  }

  @Override public void close() {
    cleanable.clean();
  }
}
```

여기서 State는 cleaner가 방을 청소할 때 수거할 자원을 담고 있고,

numJunkPiles는 수거할 자원을 의미함.

State는 Runnable을 구현하고, run 메소드에서 cleanable에 의해 딱 한 번 호출됨.

cleanable 객체는 Room 생성자에서 cleaner에 Room과 State를 등록할 때 얻음.

run 메소드가 호출되는 상황은 클라이언트가 Cleanable의 clean을 호출하거나, 클라이언트가 clean을 호출하지 않아서 가비지 컬렉터가 Room을 회수할 때, cleaner가 State의 run 메소드를 호출하는 경우임.

cleaner가 근데 즉시 수행된다는 보장이 없긴하지만 얘가 없었다면 아예 회수되지 못했을 뻔한 자원이 그나마 회수될 확률이라도 생기는 것이므로 안정망으로서 역할을 수행할 수 있음.

```java
package chapter2.item8;

public class Adult {
  public static void main(String[] args) {
    try (Room myRoom = new Room(7)) {
      System.out.println("안녕~");
    }
  }
}
```

클라이언트가 try-with-resources 블록에 Room을 생성한 경우 해당 블록이 종료될 때 자동으로 리소스가 해제됨.

출력도 **안녕~**이 출력되고, **방을 청소합니다.**가 출력됨.

```java
package chapter2.item8;

public class Teenager {
  public static void main(String[] args) {
    new Room(99);
    System.out.println("아무렴");
  }
}
```

try-with-resources 블록에 Room을 생성하지 않았으므로 해당 블록이 종료될 때 자동으로 리소스가 해제되지 않음.

이제 유일한 방법은 cleaner가 제때 실행해주는 것 밖에 없음.

그런데 **아무렴**만 출력되고, **방을 청소합니다.**는 출력되지 않음.

여러 번 실행해봤는데도 단 한 번도 **방을 청소합니다.**가 출력되지 않았음.

cleaner가 즉시 수행된다는 보장이 없다곤 했지만 그래도 한 10번 중 한 번은 수행이 될 줄 알았는데 `단 한 번도 수행되지 않았다……….!`

### **네이티브 피어(native peer)와 연결된 객체에서 사용**

`네이티브 피어`는 일반 자바 객체가 네이티브 메소드를 통해 기능을 위임한 네이티브 객체를 말한다.

- **테이티브 피어, 네이티브 메소드 네이티브 객체**
    
    네이티브 피어(Native Peer)는 자바에서 네이티브 코드(주로 C, C++ 등의 언어로 작성된 코드)와 상호 작용하기 위한 개념입니다. 네이티브 피어는 일반적으로 자바 객체와 연결되어 있으며, 네이티브 코드에서 해당 객체를 조작하고 사용할 수 있습니다. 네이티브 피어는 주로 JNI(Java Native Interface)를 통해 자바 코드와 네이티브 코드 간의 상호 작용을 가능하게 합니다.
    
    네이티브 메소드(Native Method)는 자바에서 선언되었지만 실제 구현은 네이티브 코드로 작성된 메소드입니다. 네이티브 메소드는 `native` 키워드를 사용하여 선언되며, 해당 메소드의 구현은 자바 가상 머신(Java Virtual Machine, JVM)이 아닌 네이티브 코드로 작성됩니다. 네이티브 메소드는 자바 언어로는 표현하기 어려운 기능을 수행하거나, 특정 플랫폼의 기능을 활용하기 위해 사용됩니다. 예를 들어, 그래픽 라이브러리, 하드웨어 접근 등의 기능은 네이티브 메소드를 통해 구현될 수 있습니다.
    
    네이티브 객체(Native Object)는 네이티브 코드에서 생성되고 관리되는 객체를 의미합니다. 네이티브 객체는 자바 힙 메모리가 아닌 네이티브 메모리에서 할당되며, 네이티브 코드에서의 접근과 관리를 위해 사용됩니다. 네이티브 객체는 네이티브 피어를 통해 자바 코드와 연결되어 상호 작용할 수 있습니다. 네이티브 객체는 일반적으로 특정 라이브러리나 프레임워크에서 제공하는 기능을 사용하기 위해 생성되며, 자바에서는 해당 객체를 조작하거나 사용하기 위해 네이티브 메소드를 호출하게 됩니다.
    
    네이티브 피어, 네이티브 메소드, 네이티브 객체는 모두 자바와 네이티브 코드 간의 상호 작용을 위한 중요한 요소들입니다. 네이티브 코드를 활용하여 자바 언어로는 어려운 기능을 구현하거나, 특정 플랫폼의 기능을 활용할 수 있게 해줍니다.
    

네이티브 피어는 자바 객체가 아니기 때문에, 가비지 컬렉터가 존재를 모름.

자바 피어를 회수할 때 네이티브 객체까지는 회수하지 못함.

cleaner나 finalizer가 늦게서라도 네이티브 객체까지 회수하는 것이 의미가 있을 때 사용하기에 적합.

대신, 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지지 않고 있을 때에만 해당됨.

## 저자의 핵심 정리

cleaner(자바 8까지는 finalizer)는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자. 물론 이런 경우라도 불확실성과 성능 저하에 주의해야 한다.
