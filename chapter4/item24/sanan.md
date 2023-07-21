## 핵심어

- **중첩 클래스(Nested Class)**
    
    다른 클래스 안에서 정의된 클래스
    
    ```java
    public class OuterClass {
        ...
        class NestedClass {
            ...
        }
    }
    ```
    
- **정적 멤버 클래스(Static Member Class)**
    
    외부 클래스의 static 멤버처럼 동작하는 중첩 클래스로, 어느 인스턴스에도 속하지 않으며 외부 클래스의 인스턴스 없이도 생성될 수 있다. ← because it’s static
    
    ```java
    public class OuterClass {
        ...
        static class StaticNestedClass {
            ...
        }
    }
    ```
    
    **바깥 클래스의 private 멤버에 접근할 수 있다.**
    
    **private으로 선언되면 바깥 클래스만 접근할 수 있다.**
    
- **(비정적) 멤버 클래스(Member Class)**
    
    위에서 static이 없는 것.
    
    **비정적 멤버 클래스의 메서드에서 정규화된 this를 이용해 바깥 인스턴스의 메서드를 호출하거나 참조를 가져올 수 있다.**
    
- **익명 클래스(Anonymous Class)**
    
    이름 없이 클래스를 선언하고 동시에 인스턴스화하는 기능을 제공한다.
    즉, 클래스 선언과 객체 생성을 한 문장에 모두 수행한다.
    익명 클래스는 주로 한 번만 사용되는 객체를 만드는 경우나, 클래스를 별도로 선언하기엔 부담스러울 때 사용된다.
    특히 GUI 이벤트 리스너나 Runnable 객체를 만드는 경우에 많이 사용된다.
    
    인터페이스, 추상 클래스, 구체 클래스에 대해서도 @Override를 이용하여 별개로 구현할 수 있지만, 코드의 복잡도도 고려해야한다.
    
    - 예제 코드
        
        ```java
        // "MyClass"는 일반(구체) 클래스 입니다.
        public class MyClass {
            public void myMethod() {
                System.out.println("구체 클래스의 메서드랍니다.");
            }
        }
        
        // 메인 함수에서 익명 클래스를 선언하고 사용합니다.
        public static void main(String[] args) {
            MyClass instance = new MyClass() {
                @Override
                public void myMethod() {
                    System.out.println("이것은 제가 재정의해서 지금 익명 클래스로 쓸겁니다.");
                }
            };
            instance.myMethod();
        }
        ```
        
- **지역 클래스(Local Class)**
    
    메서드 안에서 선언되는 클래스로, 해당 메서드에서만 사용할 수 있다. ← 라고 하는데, 해당 스코프에서 임시로 클래스를 만들어서 사용해버리는 것을 얘기하는 것(뉘앙스가 살짝 다름) 같다.
    
    ```java
    public class OuterClass {
        void myMethod() {
            class LocalClass {
                ...
            }
        }
    }
    ```
    
- **내부 클래스(Inner Class) - Non-Static Nested Class**

---

## 요약

- 정적 멤버 클래스는 바깥 클래스의 도우미 클래스로 쓰인다.
→ Calculator(바깥).Operation(정적멤버).MINUS
- 멤버 클래스에서 비깥 인스턴스에 접근할 일이 없다면 무조건 staUc을 붙여서 정적 멤버 클래스로 만들어라.

---

## 내 생각

정적 멤버, (비정적) 멤버 클래스에 대해서 주요하게 구분하여 생각해보면 될 것 같다.

---

## 예제 코드

- 정적 멤버 클래스 구현해보기

```java
public class StaticMemberClass {
	private static final String outerField = "Outer Field";

	public void printFields() {
		System.out.println("Outer Field : " + outerField);
	}

	public static class StaticNestedClass {
		private final String innerField;

		public StaticNestedClass(String innerField) {
			this.innerField = innerField;
		}

		public void printFields() {
			System.out.println("Outer Field : " + outerField);
			System.out.println("Inner Field : " + innerField);
		}
	}
}
/*------------------------------------------------------*/
@Test
	void 정적_멤버_클래스() {
		StaticMemberClass outer = new StaticMemberClass();
		StaticMemberClass.StaticNestedClass inner = new StaticMemberClass.StaticNestedClass("Inner Field");
		
		outer.printFields();
		//Outer Field : Outer Field

		inner.printFields();
		//Outer Field : Outer Field
		//Inner Field : Inner Field
	}
```

- 비정적 멤버 클래스로 바깥 클래스 심기 건드려보기 + 어댑터 구현해보기

```java
public class NonStaticMemberClass {
	private String outerField = "Outer Value";

	public class NonStaticNestedClass {
		private final String innerField;

		public NonStaticNestedClass(String innerField) {
			this.innerField = innerField;
		}

		public void printFields() {
			// 외부 클래스의 상태를 변경함
			outerField = "Modified by Inner Class";
			System.out.println("Outer Field : " + outerField);
			System.out.println("Inner Field : " + innerField);
		}
	}
}
/*------------------------------------------------------*/
public class NonStaticViewClass {
	private final String outerField = "Outer Value";

	public class View {
		public void printField() {
			System.out.println("Outer Field : " + outerField);
		}
	}
}
/*------------------------------------------------------*/
@Test
	void 비정적_멤버_클래스() {
		NonStaticMemberClass outer = new NonStaticMemberClass();
		NonStaticMemberClass.NonStaticNestedClass inner = outer.new NonStaticNestedClass("Inner Field");

		inner.printFields();
		//Outer Field : Modified by Inner Class
		//Inner Field : Inner Field

		NonStaticViewClass outer2 = new NonStaticViewClass();
		NonStaticViewClass.View view = outer2.new View();
		view.printField();
		//Outer Field : Outer Value
	}
```
