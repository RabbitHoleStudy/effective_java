# 톱레벨 클래스는 한 파일에 하나만 담으라

한 소스 파일에 여러 톱레벨 클래스를 두더라도 자바 컴파일러가 화내지는 않는다.

하지만 이는 별다른 이점도 없을 뿐더러 괜한 위험을 감수하게 되는 행위이다.

```java
class Utensil {

	public static void main(String[] args) {
		System.out.println(Utensil.NAME + Dessert.NAME);
	}
	
	static final String NAME = "pan";
}

class Dessert {
	static final String NAME = "cake";
}
```

```java
class Utensil {
	static final String NAME = "pot";
}

class Dessert {
	public static void main(String[] args) {
		System.out.println(Utensil.NAME + Dessert.NAME);
	}
	static final String NAME = "pie";
}
```

<img width="523" alt="스크린샷 2023-07-22 오전 12 37 32" src="https://github.com/TightJava/effective_java/assets/83565255/debff2a8-f6c5-422d-8375-6462183603df">
<img width="487" alt="스크린샷 2023-07-22 오전 12 37 39" src="https://github.com/TightJava/effective_java/assets/83565255/d143ebe9-770c-47f2-a75b-8e17768eb4b9">


이런식으로 **Utensil.java**와 **Dessert.java**를 만들게 되면 두 클래스 모두 중복이 감지되어 인텔리제이가 화를 낸다.

책에서는 **javac Dessert.java**를 실행하여 출력을 확인하면, **potpie**가 출력되고

**javac Utensil.java**를 실행하여 출력을 확인하면, **pancake**가 출력된다고 한다.

(하지만 실제로 돌려보려고 하면 기본 클래스를 찾을 수 없다는 에러가 발생한다.)

어쨋든 중요한 포인트는 **컴파일러에 어느 소스 파일을 먼저 제공하는지에 따라 동작이 달라질 수 있는 위험이 존재**한다는 것이다..!

## 핵심 정리

교훈은 명확하다. 

**소스 파일 하나에는 반드시 톱레벨 클래스(혹은 톱레벨 인터페이스)를 하나만 담자.**

이 규칙만 따른다면 컴파일러가 한 클래스에 대한 정의를 여러개 만들어 내는 일은 사라진다. 

소스 파일을 어떤 순서로 컴파일 하든 바이너리 파일이나 프로그램의 동작이 달라지는 일은 결코 일어나지 않을 것이다.
