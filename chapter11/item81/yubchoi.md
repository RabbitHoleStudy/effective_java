**📌 `wait`와 `notify`는 어셈블리어, `java.util.concurrent`는 고수준 언어. 굳이 어셈블리어로 코딩할 필요가 없다.
레거시 코드 유지보수 등의 이유로 wait를 사용해야 한다면 꼭 while문 안에서 호출하자.
일반적으로 `notify`보다는 `notifyAll`을 사용하자.**

<br/>

`wait`와 `notify`는 올바르게 사용하기가 어려우니 `고수준 동시성 유틸리티`를 사용하자.

- `wait()`: 현재 스레드를 다른 스레드가 이 객체에 대한 `notify()` 또는 `notifyAll()` 메소드를 호출할 때까지 대기
- `notify()`: 이 객체에 대해 대기 중인 스레드 하나를 깨움
- `notifyAll()`: 이 객체에 대해 대기 중인 모든 스레드를 깨움

`wait` 메서드를 사용할 때는 반드시 `wait loop` 관용구를 사용하라. 반복문 밖에서는 절대로 호출하면 안된다. 이 반복문은 `wait` 호출 전후로 조건이 만족하는지 검사한다.

일반적으로 `notify`보다는 `notifyAll`을 사용하는 것이 더 합리적이고 안전하다. 깨어나야 하는 모든 스레드가 깨어남을 보장하니 항상 정확한 결과를 얻을 것이다. 단, 모든 스레드가 같은 조건을 기다리고, 조건이 한 번 충족될 때마다 단 하나의 스레드만 혜택을 받을 수 있다면 `notifyAll` 대신 `notify`를 사용하는 것이 좋다.

<br/>

`java.util.concurrent`의 고수준 유틸리티는 세 범주로 나눌 수 있다.

- 실행자 프레임워크(아이템 80), 동시성 컬렉션(concurrent collection), 동기화 장치(synchronizer)

### **동시성 컬렉션(concurrent collection)**

- List, Queue, Map과 같은 표준 컬렉션 인터페이스에 동시성을 가미하여 구현한 컬렉션
- 동시성 컬렉션에서 동시성을 무력화하는 것은 불가능하며 외부에서 락을 추가로 사용하면 오히려 성능이 저하될 수 있다.
- **ConcurrentHashMap**
    - 동시성이 뛰어나며 속도가 무척 빠르다.
    - `Collections.synchronizedMap`보다 성능이 우수하다.
    - `SynchronizedMap`과 `ConcurrentHashMap`
        ![SynchronizedMap](https://github.com/TightJava/effective_java/assets/65652094/8bc8c9ee-7837-450d-a885-8b04b70a5c8b)
        ![ConcurrentHashMap](https://github.com/TightJava/effective_java/assets/65652094/da2a8659-bf5a-41e6-aaac-2bf8efe0a024)

- **BlockingQueue**
    - take 메서드는 큐의 첫 원소를 꺼낸다. 만약 큐가 비어있다면 새로운 원소가 추가될 때까지 기다린다.
    - 작업 큐로 쓰기에 적합하다. 작업 큐는 하나 이상의 생산자 스레드가 작업을 큐에 추가하고, 하나 이상의 소비자 스레드가 큐에 있는 작업을 꺼내 처리하는 형태다.
    - `ThreadPoolExecutor`를 포함한 대부분의 실행자 서비스 구현체에서 `BlockingQueue`를 사용한다.

### 동기화 장치(synchronizer)

- 스레드가 다른 스레드를 기다릴 수 있게 하여 서로 작업을 조율할 수 있도록 해준다.
- `CountDownLatch`
    ![CountDownLatch](https://github.com/TightJava/effective_java/assets/65652094/5f732048-bdb9-4d22-a5e2-f93c8d9974e7)

    
    - 일회성 장벽. 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다.
- `Semaphore`
    - 특정 자원이나 특정 연산을 동시에 사용하거나 호출할 수 있는 스레드의 수를 제한한다.
- `Phaser`
    ![Phaser](https://github.com/TightJava/effective_java/assets/65652094/c8be28e0-c744-487d-aa0f-1300ccff0981)

    
    - 가장 강력한 동기화 장치.
    - 동기화에 참여할 스레드의 수가 동적이고 재사용이 가능하다.

💡 시간 간격을 잴 때는 항상 `System.currentTimeMillis`가 아닌 `System.nanoTime`을 사용하자. 더 정확하고 정밀하다.
