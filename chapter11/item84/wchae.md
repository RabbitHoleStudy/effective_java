
# 요약

- 이식성 좋은 프로그램을 작성하려면, 스레드 스케줄러에 따라서 성능이 달라지는 프로그램을 작성하면 안된다.
- Thread.yield 와 스레드 우선순위에 의존하면 안된다.
- 스레드를 프로그램을 ‘고치는 용도’로 쓰지 말자

---

# 이식성

- 정확성이나 성능이 스레드 스케줄러에 따라 달라지는 프로그램은 다른 플랫폼에 이식하기 어렵다.
    - 이식성 좋은 프로그램을 작성하는것 == 스레드의 평균적인 수를 프로세서 수 보다 지나치게 많아지지 않도록 하는 것

# ThreadScheduler

- 여러 스레드가 실행 중 일때, 운영체제의 스레드 스케줄러가 어떤 스레드를 얼마나 실행시킬지 결정한다.

# 스레드 수

- 실행 가능한 스레드의 수와 전체 스레드 수는 구분해야한다.
    - 전체 스레드 수는 많고, 대기 중인 스레드는 실행 가능하지 않다.
    
- 스레드는 당장 처리해야할 작업이 없다면 실행돼서는 안된다.
- 스레드 풀 크기를 적절히 설정하고, 작업은 짧게 유지하면 된다. 너무 짧으면 또 작업분배 부담이 오히려 성능을 떨어뜨린다.

# busy waiting

```jsx
while(true){
	//내부 상태가 변할때
  //break
}
```

- 원하는 자원을 얻기 위해 기다리는게 아니라, 얻을 때까지 계속 확인하는 것

- 스레드는 buzy waiting 상태가 되면 안된다.
- 공유 객체의 상태가 바뀔 때까지 쉬지 않고 검사하면 안된다.

```jsx
public class BusyWaiting {

	public static void main(String[] args) {
		SlowCountDownLatch slowCountDownLatch = new SlowCountDownLatch(10);
		slowCountDownLatch.countDown();
		slowCountDownLatch.await();
	}

	static class SlowCountDownLatch {

		private int count;

		public SlowCountDownLatch(int count) {
			if (count < 0) {
				throw new IllegalArgumentException(count + " < 0");
			}
			this.count = count;
		}

		public void await() {
			while (true) {
				synchronized (this) {
					if (count == 0) {
						return;
					}
				}
			}
		}

		public synchronized void countDown() {
			if (count != 0) {
				count--;
			}
		}
	}

}
```

# Thread.yield

- 타이밍을 맞추려고 Thread.yield 를 써서 문제를 고치면안된다.

```jsx
public class ThreadYield {
		public static void main(String[] args) {
			Thread producer = new Thread(new ProducerThread(), "생산자");
			Thread consumer = new Thread(new ConsumerThread(), "소비자");

			producer.start();
			consumer.start();
		}
	}

	class ProducerThread implements Runnable {
		@Override
		public void run() {
			for (int i = 0; i < 5; i++) {
				System.out.println(Thread.currentThread().getName() + " 아이템 생산 " + i);
				Thread.yield(); // 시간 양보
			}
		}
	}

	class ConsumerThread implements Runnable {
		@Override
		public void run() {
			for (int i = 0; i < 5; i++) {
				System.out.println(Thread.currentThread().getName() + " 아이템 소비 " + i);
				Thread.yield(); // 시간 양보
			}
		}

}

생산자 아이템 생산 0
생산자 아이템 생산 1
생산자 아이템 생산 2
생산자 아이템 생산 3
생산자 아이템 생산 4

소비자 아이템 소비 0
소비자 아이템 소비 1
소비자 아이템 소비 2
소비자 아이템 소비 3
소비자 아이템 소비 4

//이게 이렇게 나올 확률이 높아질 뿐 보장하진 않는다.

```
![image](https://github.com/TightJava/effective_java/assets/13278955/a4437388-776e-4c2b-a5c8-ba669d901c6c)

- Thread.yield() 메서드는 현재 스레드가 현재의 시간 슬라이스를 양보하고 다른 스레드가 실행될 수 있도록 스케줄러에게 “제안”한다.
- 스레드를 “프로그램을 고치는” 용도로 사용하면 안된다.
