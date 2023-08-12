# 요약

- 내가 필요한 기능이 있을때 먼저 표준 라이브러리를 찾아보자
- 기존의 Java 라이브러리를 쓸때는 어떤 한계를 가지고 있는지 알아보고 쓰자
- 표준 라이브러리를 쓸 경우 장점들이 많다.
- `java.lang, java.util, [java.io](http://java.io)` 와 그 하위 패키지들에는 익숙해지자
    - 스트림 라이브러리 java.util.concurrent 의 동시성 기능도 알아두면 좋다 (아이템 80, 81에서 다룬다.)
- 장점요약
    - 집중해야될 내 서비스의 로직에 집중
    - 계속해서 일어나는 기능개선
    - 여러사람들이 보고 이해하기 쉬운코드

# Random method의 결함

( 으로 알아보는 라이브러리를 알고쓰자는 메세지 )

- 무작위 정수를 생성하고 싶을때 사용할 수 있는 java.util.Random

```java
public class RandomGenerater {
	static Random rnd = new Random();

	static int random(int n) {
		return Math.abs(rnd.nextInt()) % n;
	}
```

### 첫번째 - 전달인자가 2의 제곱수이고, 작으면 같은수가 나올 확률이 높다.

- `int random(int n)`
    - n 이 크지 않은 2의 제곱수라면 같은 수열이 반복된다.

---

### 두번째 -  n이 2의 제곱수가 아니라면, 얼마 지나지 않아 같은 수열이 반복된다.

```java
	public static void main(String[] args) {
		int n = 2 * (Integer.MAX_VALUE / 3);
		int low = 0;

		for(int i = 0; i < 1000000; i++) {
			if (random(n) < n/2)
				low++;
		}
		System.out.println(low);
	}
}

// 예상 값 : 
// 실제 수치 : 666636
```

- 무작위수 100만개
- 중간 값 보다 작은값이 몇개인지 확인
- 3 분의 2가 중간값보다 낮은값 ⇒ 수열이 반복됨을 알 수 있다

---

### 세 번째 - 지정한 범위 바깥의 수가 나올 가능성이 있다.

- rnd.nextInt() 가 반환한값을 Math.abs (절대값) 을 이용해서 음수가 아닌 정수로 매핑하는 과정에서 발생한다.
    - Integer.MIN_VALUE 반환시, 언더플로우가 난다.
    
    ```java
    int absedRandom = Math.abs(Integer.MIN_VALUE);
    System.out.println(absedRandom);
    
    //-2147483648
    ```
    
- 알고리즘 전문가들이 해결해놨다. 	`Random.nextInt(int)`
    
    ```java
    public class RandomGenerater {
    	static Random rnd = new Random();
    
    static int random(int n) {
    		//Random.nextInt(int)
    		return rnd.nextInt(n);
    //		return Math.abs(rnd.nextInt()) % n;
    	}
    
    //결과값 : 500390
    ```
    

---

# 표준 라이브러리를 써라

## 자바 7부터는 ThreadLocalRandom을 써라

- Random 보다 고품질의 부작위 수를 생성 + 속도도 더 빠름
    - full code
        
        ```java
        import java.util.Random;
        import java.util.concurrent.ThreadLocalRandom;
        
        public class RandomGenerater {
        	static Random rnd = new Random();
        	static ThreadLocalRandom rnd2 = ThreadLocalRandom.current();
        	static int random(int n) {
        		return rnd.nextInt(n);
        //		return Math.abs(rnd.nextInt()) % n;
        	}
        	static int randomVersion2(int n){
        		return rnd2.nextInt(n);
        	}
        
        	public static void main(String[] args) {
        		long start = System.currentTimeMillis();
        		int n = 2 * (Integer.MAX_VALUE / 3);
        		int low = 0;
        		for(int i = 0; i < 1000000; i++) {
        			if (random(n) < n/2)
        				low++;
        		}
        		System.out.println(low);
        		long end = System.currentTimeMillis();
        		System.out.println("time = " + (end - start));
        
        		start = System.currentTimeMillis();
        		low = 0;
        		for(int i = 0; i < 1000000; i++) {
        			if (randomVersion2(n) < n/2)
        				low++;
        		}
        		System.out.println(low);
        		end = System.currentTimeMillis();
        		System.out.println("time = " + (end - start));
        	}
        }
        ```
        
    
    ```java
    static ThreadLocalRandom rnd2 = ThreadLocalRandom.current();
    
    static int randomVersion2(int n){
    		return rnd2.nextInt(n);
    	}
    
    500197
    time = 15
    
    500246
    time = 17
    ```
    
- 포크 조인 풀이나 병렬 스트림에서는 SplittableRandom을 써라

- 핵심적인 개발사항과 상관없는것에 시간을 쓰지 않을 수 있다.
- 따로 노력하지 않아도 성능이 지속적으로 개선된다 → 커뮤니티의 빡고수들이 개선사항을 제시하고 계속해서 발전시켜준다.
- 내가 작성한 코드를 인수인계하기 쉬워진다 ( 표준라이브러리를 잘 아는 사람들이 고치기 쉬워진다)

---

## 자바프로그래머라면 이정도는 알자

- java.lang
- java.util
- java.io

- 위 패키지들과 하위 패키지는 한번쯤 보자
- 메이저 버전별 차이점도 알아두어야할것같다.

### Java 버전별 차이점

- java 5 :
    - 제네릭스, 에어노테이션, enum 타입추가
    - 자동 박싱-언박싱
- java 7 :
    - <> 연산자,
    - try-with-resources 기능 추가,
    - 문자열 switch문,
    - 새로운 파일 IO API 도입
- java 8 :
    - 람다
    - 스트림
    - 날짜와 시간 API 추가
- java 9 :
    - 모듈 시스템 도입
- java 10 :
    - var 키워드
    - 지역 변수 형식 추론 추가
- java 11 :
    - oracle jdk 와 open jdk 의 라이센스 변경
    - HTTP 클라이언트 표준 API 추가
- Java 17
    - permits 키워드 추가
    - JDK 내부 모듈이 기능완성 상태로 변경

---

# 바퀴를 재발명하지말자
![img](https://github.com/TightJava/effective_java/assets/13278955/d3fa7a04-0935-4acc-b327-9aae1246b3ed)


바퀴의 재발명
