**📌 `java.util.concurrent`의 실행자 프레임워크는 태스크의 실행과 관리를 최적화한다.
태스크(`Runnable`과 `Callable`)는 실행 매커니즘과 분리되어 실행자 서비스에 의해 관리되며, `포크-조인 풀`은 병렬 작업을 최적화해 CPU 활용을 극대화한다.
스레드를 직접 다루지 말고 실행자, 태스크, 스트림을 이용하자.**   

### 실행자 서비스

`java.util.concurrent` 패키지에는 실행자 프레임워크(`Executor Framework`)라고 하는 인터페이스 기반의 태스크 실행 기능이 있다.

```java
// 실행자(작업큐) 생성
ExecutorService exec = Executors.newSingleThreadExecutor();

// 실행자에 실행할 task를 넘김
exec.execute(runnable);

// 실행자 종료
exec.shutdown();
```

### 실행자 서비스의 주요 기능

- 특정 태스크가 완료되기를 기다린다(`get` 메서드).
    
    ```java
    ExecutorService exec = Executors.newSingleThreadExecutor();
    exec.submit(() -> s.removeObserver(this)).get();
    ```
    
- 태스크 모음 중 임의의 한 태스크(`invokeAny` 메서드) 또는 모든 태스크(`invokeAll` 메서드)가 완료되기를 기다린다.
    
    ```java
    String anyResult = exec.invokeAny(tasks);
    System.out.println("Any Task done with result: " + anyResult);
    
    List<Future<String>> futures = exec.invokeAll(tasks);
    System.out.println("All Tasks done");
    ```
    
- 실행자 서비스가 종료하기를 기다린다.(`awaitTermination` 메서드).
    
    ```java
    exec.awaitTermination(10, TimeUnit.SECONDS)
    ```
    
- 완료된 태스크들의 결과를 차례로 받는다(`ExecutorCompletionService` 이용).
- 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다(`ScheduledThreadPoolExecutor` 이용).
    
    ```java
    ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(1);
    
    executor.scheduleAtFixedRate(() -> {
        System.out.println(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")
                .format(LocalDateTime.now()));
    }, 0, 2, TimeUnit.SECONDS);
    ```
    

### ThreadPool

- **ThreadPoolExecutor**
    - `ExecutorService`는 인터페이스이고, 구현체인 `ThreadPoolExecutor`를 통해 초기화할 수 있다.
    - 스레드풀 동작을 결정하는 거의 모든 속성을 설정할 수 있다.
- **CachedThreadPool**
    - 작은 프로그램이나 가벼운 서버에 사용하기 적합하다.
    - 요청 받은 task들을 큐에 쌓지 않고 즉시 실행한다. 가용한 스레드가 없다면 새로 생성해 실행한다.
    - 무거운 서버라면 CPU 이용률이 100%에 가까워지고, 새로운 task가 도착할 때마다 다른 스레드를 생성하면서 상황을 더욱 악화시킨다.
- **FixedThreadPool**
    - 무거운 프로덕션 서버에 사용하기 적합하다.
    - 스레드 개수를 고정해 사용한다.

### Task

실행자 프레임워크에서는 **작업 단위**와 **실행 매커니즘**이 분리된다.

`태스크`는 작업 단위를 나타내는 핵심 추상 개념이다. `실행자 서비스`는 태스크를 수행하는 일반적인 매커니즘이다.

태스크에는 `Runnable`과 `Callable` 2가지가 있다. `Callable`은 `Runnable`과 비슷하지만 값을 반환하고 임의의 예외를 던질 수 있다.

### fork-join task
![image](https://github.com/TightJava/effective_java/assets/65652094/702cdac7-f851-4eab-81fb-9b890581f920)

`포크-조인 태스크`는 `포크-조인 풀`이라는 특별한 실행자 서비스가 실행해준다.

ForkJoinTask의 인스턴스는 작은 하위 태스크로 나뉠 수 있고, ForkJoinPool을 구성하는 스레드들이 이 태스크들을 처리하며, 일을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크를 가져와 대신 처리할 수도 있다.

이를 통해 CPU를 최대한 활용하면서 높은 처리량과 낮은 지연시간을 달성한다. 병렬 스트림(아이템 48) 또한 포크-조인 풀을 이용해 만들어졌다.
