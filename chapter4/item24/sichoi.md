# 멤버 클래스는 되도록 static으로 만들라

## 정적 멤버 클래스

다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에 접근할 수 있다는 점만 빼면 일반 클래스가 다르지 않다.

바깥 클래스와 함께 쓰일 때만 유용한 public 헬퍼 클래스로 쓰인다.

- **정적 멤버 클래스 예시**
    
    ```java
    public class Calculator {
    	public static void main(String[] args) {
    		Calculator calculator = new Calculator();
    		int result1 = calculator.calculate(10, 5, Calculator.Operator.PLUS);
    		int result2 = calculator.calculate(10, 5, Calculator.Operator.MINUS);
    		System.out.println("Result1 (PLUS): " + result1);
    		System.out.println("Result2 (MINUS): " + result2);
    	}
    
    	// 내부에 Operator라는 정적 멤버 클래스
    //	enum == static final class
    	public enum Operator {
    		PLUS,
    		MINUS
    		// 원하는 만큼 다른 연산자를 추가할 수 있습니다.
    	}
    
    	public int calculate(int a, int b, Operator op) {
    		return switch (op) {
    			case PLUS -> a + b;
    			case MINUS -> a - b;
    			default -> throw new UnsupportedOperationException("Not supported operator");
    		};
    	}
    }
    ```
    
    ```java
    Result1 (PLUS): 15
    Result2 (MINUS): 5
    ```
    

## 비정적 멤버 클래스

구문상으로는 정적 멤버 클래스에서 static만 빠졌지만, 의미상으로 차이가 크다.

**바깥 클래스의 인스턴스와 연결된다.**

비정적 멤버 클래스의 인스턴스 메소드에서 정규화된 this를 사용해 바깥 인스턴스의 메소드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있다.

- **정규화된 this란?**
    
    **클래스명.this** 형태로 바깥 클래스의 이름을 명시하는 용법을 말한다.
    
- **비정적 멤버 클래스 예시 - 1 (책의 예제)**
    
    ```java
    public class OuterClass {
    
    	public static void main(String[] args) {
    		OuterClass outer = new OuterClass();
    		OuterClass.InnerClass inner = outer.createInner();
    		inner.printOuterField(); // "Outer class field" 출력
    	}
    
    	private String outerField = "Outer class field";
    
    	class InnerClass {
    		void printOuterField() {
    			System.out.println(outerField); // Outer class field 접근 가능
    			System.out.println(OuterClass.this.outerField); // 정규화된 this를 이용한 접근
    		}
    	}
    
    	// InnerClass 인스턴스화를 위한 메소드
    	public InnerClass createInner() {
    		return new InnerClass();
    	}
    
    }
    ```
    
    ```java
    Outer class field
    Outer class field
    ```
    
- **비정적 멤버 클래스 예시 - 2 (현실 사용)**
    
    현실에서는 비정적 멤버 클래스는 `어댑터`를 정의할 때 자주 사용된다고 한다.
    
    어떤 클래스의 인스턴스를 감싸 **마치 다른 클래스의 인스턴스 처럼 보이게 하는 뷰**로써 사용하는 것이다.
    
    ```java
    import java.util.*;
    
    public class MySet<E> extends AbstractSet<E> {
    
    		public static void main(String[] args) {
    				MySet<Integer> set = new MySet<>();
    				set.add(1);
    				set.add(2);
    				set.add(3);
    				set.add(4);
    				set.add(5);
    		
    				Iterator<Integer> iterator = set.iterator();
    				while(iterator.hasNext()) {
    					System.out.println(iterator.next());
    				}
    			}
    
        private List<E> list;
    
        public MySet() {
            this.list = new ArrayList<>();
        }
    
        @Override
        public boolean add(E e) {
            if(list.contains(e)) {
                return false;
            }
            list.add(e);
            return true;
        }
    
        @Override
        public Iterator<E> iterator() {
            return new MyIterator();
        }
    
        @Override
        public int size() {
            return list.size();
        }
    
        // 비정적 멤버 클래스
        private class MyIterator implements Iterator<E> {
            private int index = 0;
    
            @Override
            public boolean hasNext() {
                return index < list.size();
            }
    
            @Override
            public E next() {
                if(!hasNext()) {
                    throw new NoSuchElementException();
                }
                return list.get(index++);
            }
        }
    }
    ```
    
    ```java
    1
    2
    3
    4
    5
    ```
    
    여기에서 `MySet`은 내부 리스트를 사용해서 집합을 구현한다.
    **MyIterator**는 MySet의 `비정적 멤버 클래스`로서, Iterator 인터페이스를 구현한다. 
    
    비정적 멤버 클래스인 이유는 Iterator가 자신이 순회할 집합의 상태에 접근해야하기 때문이다. 
    

**멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자.**

static을 생략하게 되면 숨은 외부 참조를 불필요하게 갖게되어, 이 참조를 저장하기 위한 시간과 공간이 낭비된다.

심지어 가비지 컬렉터가 수거해가지 못하여 메모리 누수가 발생할 수 있다.

## 익명 클래스

익명 클래스는 이름이 없는 클래스이다. (그것이 익명이니까..!)

- **익명 클래스란?**
    
    익명 클래스, 즉 이름이 없는 클래스는 자바에서 주로 빠르게 클래스를 만들어 사용하고자 할 때 쓰이는 문법입니다. 익명 클래스는 클래스의 선언과 객체의 생성이 일단에 이루어집니다. 따라서 익명 클래스는 주로 일회성의 사용에 적합합니다.
    주로 인터페이스나 추상클래스의 메서드를 잘라내어 빠르게 사용할 경우에 쓰입니다. 또한 GUI 프로그래밍에서 이벤트 처리 등에도 자주 사용됩니다.
    

얘는 바깥 클래스의 멤버도 아니다. 멤버와는 달리 쓰이는 시점에 선언과 동시에 인스턴스가 생성된다.

람다를 사용하기 전에는 **즉석에서 빠르게 작은 함수 객체나 처리 객체를 만들기 위해 사용**했으나, 지금은 **정적 팩터리 메소드**를 구현할 때 정도로만 사용되는 듯 하다.

- **익명 클래스 사용 예시**
    
    [https://inpa.tistory.com/entry/JAVA-☕-익명-클래스Anonymous-Class-사용법-마스터하기](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%9D%B5%EB%AA%85-%ED%81%B4%EB%9E%98%EC%8A%A4Anonymous-Class-%EC%82%AC%EC%9A%A9%EB%B2%95-%EB%A7%88%EC%8A%A4%ED%84%B0%ED%95%98%EA%B8%B0)
    
    ```java
    class Animal {
    	public static void main(String[] args) {
    		// 익명 클래스 : 클래스 정의와 객체화를 동시에. 일회성으로 사용
    		Animal dog = new Animal() {
    			@Override
    			public String bark() {
    				return "개가 짖습니다";
    			}
    		}; // 단 익명 클래스는 끝에 세미콜론을 반드시 붙여 주어야 한다.
    
    		// 익명 클래스 객체 사용
    		System.out.println(dog.bark());
    
    		Animal cat = new Cat();
    		System.out.println(cat.bark());
    	}
    
    	// 자식 클래스
    //	멤버 클래스에서 바깥 인스턴스에 접근할 일이 없으므로
    //	static을 붙여주었다. (킹텔리제이도 그걸 추천한다.)
    	static class Cat extends Animal {
    		@Override
    		public String bark() {
    			return "고양이가 웁니다";
    		}
    	}
    
    	public String bark() {
    		return "동물이 웁니다";
    	}
    
    }
    ```
    
    ```java
    개가 짖습니다
    고양이가 웁니다
    ```
    

## 지역 클래스

가장 안쓰인다. 

지역 클래스는 지역변수를 선언할 수 있는 곳이라면 어디서든 선언할 수 있고, scope도 지역변수와 같다.

멤버 클래스처럼 이름이 있고 반복해서 사용가능하다.

익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있다.

정적 멤버를 가질 수 없고, 가독성을 위해 짧게 작성해야 한다.

- **지역 클래스 사용 예시**
    
    ```java
    public class OuterClass {
    
        void aMethod() {
            
            class LocalClass {
                void print() {
                    System.out.println("Hello, I'm a local class");
                }
            }
            
            LocalClass localClass = new LocalClass();
            
            localClass.print();
        }
    
        public static void main(String[] args) {
            OuterClass outerClass = new OuterClass();
            outerClass.aMethod();
        }
    }
    ```
    
    ```java
    Hello, I'm a local class
    ```
    
    메소드 안에 이런식으로 클래스를 정의할 수 있다.
    

익명 클래스처럼 한 메소드 안에서만 쓰이면서 인스턴스를 한 곳에서만 쓰일 때 사용하지만, 위에서 Animal 예시처럼 리스코프 교체가 원할하게 이루어질 수 있는 상황이라면 익명 클래스를 이용하고, 그렇지 않으면 지역 클래스로 활용하는 것이 좋다고 이해했다.

## 저자의 핵심 정리

증첩 클래스에는 네 가지가 있으며, 각각의 쓰임이 다르다. 

메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기엔 너무 길다면 멤버 클래스로 만든다. 

멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조한다면 비정적으로, 그렇지 않으면 정적으로 만들자. 

중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스틴스를 생성하는 지점이 단 한 곳이고 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있다면 익명 클래스로 만들고, 그렇지 않으면 지역 클래스로 만들자.
