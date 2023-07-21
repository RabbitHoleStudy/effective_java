## 핵심어

- **인터페이스**
    
    공통적으로 사용되어야 하는 **메서드(행동)에 대한 정의**를 결정하고, 이를 실제로 **어떻게 처리해야할지(구현) 정해주는 기능**을 제공한다. 이를 통해 이 인터페이스를 **구현하면 이 인터페이스의 규약을 지키는 하나의 타입으로 정해지고, 이는 다형성을 가능케한다.**
    

- **추상 클래스**
    
    추상 클래스는 인터페이스와 매우 유사하지만 약간 다르다. 추상 클래스는 "일반적인" 클래스와 같지만, 하나 이상의 메서드가 구현되지 않고 남아 있다는 점에서 다르다. 이런 메서드들을 추상 메서드라고 부르고, **하위 클래스는 이 추상 메서드(abstract 키워드)들을 반드시 재구현(Override)**해야 한다.
    
    - 예시 코드
        
        ```java
        public abstract class AbstractClass {
            // 추상 메서드, 구현부는 없다.
            public **abstract** void abstractMethod();
        }
        
        public class ChildClass extends AbstractClass {
            // 추상 클래스를 상속한다면, 추상 메서드는 반드시 구현되어야 한다.
        		// CPP의 순수 가상 함수와 동일하다.
            @Override
            public void abstractMethod() {
                System.out.println("이것은 반드시 구현됐어야 혀");
            }
        }
        ```
        
    
- **디폴트 메서드**
    
    자바 8 이후부터 인터페이스는 **default 키워드**를 사용해 디폴트 메서드를 포함할 수 있게 되었다. 이는 기존 인터페이스를 변경하지 않고도 새로운 메서드를 추가할 수 있게 해주는 특징인데, **인터페이스를 구현하는 클래스에서 오버라이드할 수 있지만, 그렇지 않은 경우에는 디폴트 메서드의 구현이 사용**된다.
    
- **템플릿 메서드 패턴(추상 클래스가 주가 되는 패턴)**
    
    템플릿 메서드 패턴은 **알고리즘의 골격을 정의**하는 데 사용된다.
    이는 **알고리즘의 일부를 하위 클래스에서 재정의할 수 있게 만듦으로써, 알고리즘의 구조는 그대로 유지하되, 특정 단계들을 변경하여 알고리즘의 행동을 변경**할 수 있다.
    이 패턴은 주로 추상 베이스 클래스에서 구현되는데, 이 클래스는 알고리즘의 단계를 정의하는 템플릿 메서드를 가진다.
    이 메서드의 각 단계는 일반 메서드, 추상 메서드 또는 둘 모두를 호출할 수 있다.
    추상 메서드는 하위 클래스에서 구현되며, 일반 메서드는 선택적으로 재정의될 수 있다.
    이렇게 하면 알고리즘의 흐름을 변경하지 않으면서도 알고리즘의 특정 단계를 재정의할 수 있다.
    
    - 예시 코드
        
        ```java
        abstract class AbstractClass {
            // 최종 실행 메서드 - 템플릿 메서드
            final void templateMethod() {
                primativeOperation1();
                concreteOperation();
                if(hook()) { 
                   primativeOperation2();
                }
            }
        
            // 기본적인 연산
            abstract void primativeOperation1();
            abstract void primativeOperation2();
        
            // 구체적인 연산
            final void concreteOperation() {
                // 반드시 수행되어야하는 구체적인 연산 - final로 재정의불가
            }
          
            // 훅
            boolean hook() {
                return true;
            }
        }
        
        class ConcreteClass extends AbstractClass {
            void primativeOperation1() {
                // Implementation here...
            }
          
            void primativeOperation2() {
                // Implementation here...
            }
        
            boolean hook() {
                return false;
            }
        }
        ```
        
    
- **골격 구현 클래스(Abstract - )**
    
    골격 구현(Skeleton implementation) 혹은 추상 클래스 구현은 **인터페이스와 골격 구현을 함께 사용하여 최종 사용자가 특정 메서드를 구현하는 것에만 초점**을 맞출 수 있게 하는 방식이다.
    골격 구현 클래스 이용으로 인터페이스를 구현하는 데 필요한 공통 코드를 재사용할 수 있다.
    이 패턴의 **주된 목적은 인터페이스를 구현하는 데 중복 코드를 최소화하고, 개발자가 인터페이스에 선언된 메서드 중 일부만 재정의하도록 하는 것**이다.
    예를 들어, Java의 AbstractList, AbstractSet, AbstractMap와 같은 클래스들은 각각 List, Set, Map 인터페이스의 골격 구현을 제공한다.
    이런 추상 클래스들을 상속받아서 개발자는 필요한 메서드만 오버라이드하면 된다.
    
    - 예시 코드
        
        ```java
        public interface MyInterface {
            void method1();
            void method2();
            void method3();
        }
        
        public abstract class MyInterfaceSkeleton implements MyInterface {
            @Override
            public void method1() {
                // 기본 구현
            }
        
            // 하위 클래스의 구현이 필요한 method2, method3
        		// 이 경우, MyInterface에 대한 구현이므로 abstract로 설정할 수 없다.
        		// 따라서, throw를 통해 구현을 강제하게끔 할 수 있다(라이브러리 참조).
        }
        
        public class MyClass extends MyInterfaceSkeleton {
            @Override
            public void method2() {
                // 상세 구현
            }
        
            @Override
            public void method3() {
                // 상세 구현
            }
        }
        ```
        
    
- **단순 구현(Simple - )**
    
    단순 구현(simple implementation)이란 해당 인터페이스를 구현하는 가장 기본적인 혹은 단순한 구현을 의미한다.
    이는 보통 인터페이스의 모든 메서드를 최소한의 기능만 가진 형태로 구현하는 것을 포함한다.
    
    - 예시 코드
        
        ```java
        public class SimpleArrayList<E> implements List<E> {
            private Object[] elements;
            private int size;
        
            public SimpleArrayList() {
                elements = new Object[10]; // 초기 수용량
                size = 0;
            }
        
            @Override
            public int size() {
                return size;
            }
        
            @Override
            public boolean isEmpty() {
                return size == 0;
            }
        
            @Override
            public E get(int index) {
                if (index >= size || index < 0) {
                    throw new IndexOutOfBoundsException();
                }
                return (E) elements[index];
            }
        
            @Override
            public boolean add(E e) {
                ensureCapacity();
                elements[size++] = e;
                return true;
            }
        
            // ..등등
        
            private void ensureCapacity() {
                if (size == elements.length) {
                    elements = Arrays.copyOf(elements, size * 2);
                }
            }
        }
        ```
        

---

## 요약

- 추상 클래스를 상속하면 하위 클래스는 단일 상속원칙으로 인해서 하나의 타입만을 구현한 것이 되지만, 인터페이스는 그렇지 않고 구현하는 모든 인터페이스에 대한 타입이 된다.
- 기존의 클래스에도 손쉽게 인터페이스를 구현시킬 수(타입을 지정해줄 수) 있다.
- Object의 기본 메서드들(equals, hashcode..)는 디폴트 메서드로 제공해서는 안 된다.
- 복잡한 구현이라면 골격 구현을 고려하자.

---

## 내 생각

- **인터페이스를 구현하는 것은 해당 인터페이스에 대한 타입으로 지정하는 것이다.**
- 일반 클래스와 추상 클래스의 상속이 무슨 차이일까?
    
    일반 클래스와 추상 클래스를 상속하는 것은 모두 부모 클래스의 특성과 메서드를 하위 클래스에 상속하게 해주는 방법이지만 차이점이 있다.
    
    - 추상 클래스의 **추상 메서드로 지정된 경우에는 해당 메서드에 대한 구현이 강제된다.**
    - **추상 클래스는 인스턴스화 될 수 없다.**
    → 추상 클래스는 그 자체로 사용되기보단, 다른 클래스가 그것을 상속하여 구현하도록 설계된 구조다.
    - 추상 클래스는 **하위 클래스에게 상위 클래스의 몇몇 핵심 기능을 공유하면서도, 디테일한 부분을 하위 클래스에게 미루어 구현하도록** 한다.
    - 따라서, 추상 클래스를 상속하려는 경우 이러한 특성을 고려하면 됩니다. 추상 메서드를 통해 구현을 강제하고 싶거나 일부 공통 기능을 공유하면서도 서브클래스에 유연성을 두려는 경우에 추상 클래스를 사용하면 됩니다. 반면, 특정 기능의 구체적인 버전을 상속받아 사용하려는 경우에는 일반 클래스 상속을 사용하면 됩니다.
- 추상 클래스 vs 인터페이스
    - 두 **기능의 가장 큰 차이점은 추상 클래스는 (자바에서) "단일 상속"만 지원한다는 점**이다.
    따라서, 한 클래스가 여러 추상 클래스를 동시에 확장하는 것은 불가능하다.
    이는 추상 클래스를 사용할 때 상당한 제약을 불러일으킬 수 있다.
    반면, **인터페이스는 "다중 상속"를 지원**하므로, 하나의 클래스가 여러 인터페이스를 동시에 구현할 수 있다.
- 그렇다면 언제 나누어서 사용하는가?
    - **다중 상속의 필요성이 있다면 인터페이스**를 사용하는 것이 좋다.
    - **상호 작용하는 다른 클래스가 많을수록, 그리고 클래스 사이의 상호 작용이 복잡할수록 인터페이스를** 사용하는 것이 좋다.
    - **인터페이스는 확장성을 고려하고 시스템이 변경되어야 할 필요성을 예측할 수 있는 경우에 유용**하다.
    새로운 메서드가 추가될 필요가 생기면 인터페이스에 추가 할 수 있고, 이를 구현하는 모든 클래스를 업데이트해야 한다.
    반면, 추상클래스는 기존 메서드의 기본구현을 제공하므로, 새로운 기능을 추가하는 것이 더 쉽다.
    - **특정 메서드 구현을 공유해야 하는 경우 추상 클래스**를 사용하는 것이 좋다.
    예를 들어, equals, hashCode, toString과 같은 메서드를 공유할 수 있다.
    추상 클래스의 구체적인 메서드를 모두 이용하려면 일부 메소드를 오버라이드해야 할 수도 있다.
    - **상위 클래스의 일련의 메서드가 서로에게 의존적인 경우, 추상 클래스를 사용**하여 하위 클래스가 이들 메서드를 적절하게 조합하도록 강제할 수 있다.
    - 인터페이스는 메서드의 시그니처만 정의하기 때문에, **동일한 인터페이스를 구현하는 여러 클래스 사이에서 공통 로직을 공유하려면 추상 클래스를 사용**해야 한다.

**정말정말 다를 여지가 없고, 변해도 같이 변해야 하는 경우가 아니라면 나는 인터페이스를 이용할 것 같다.**

---

## 예제 코드

- 추상 클래스와 이를 통한 템플릿 메서드 패턴 구현

```java
public abstract class BreadMaker {

	final void makeBread() {
		mixIngredients();
		ferment();
		bake();
		if (isTopped()) {
			topSomeIngredients();
		}
	}

	abstract void mixIngredients();

	abstract void ferment();

	abstract void bake();

	final void topSomeIngredients() {
		System.out.println("토핑 더 많이 얹기~");
	}

	boolean isTopped() {
		return false;
	}
}
/*-----------------------------------------------------------*/
public class BaguetteMaker extends BreadMaker {

	@Override
	void mixIngredients() {
		System.out.println("반죽 배합하기 ~");
	}

	@Override
	void ferment() {
		System.out.println("숙성하기~");
	}

	@Override
	void bake() {
		System.out.println("오븐에 집어넣기~");
	}

	@Override
	boolean isTopped() {
		return true;
	}
}
/*-----------------------------------------------------------*/
@Test
	void 템플릿_메서드_패턴() {
		BaguetteMaker baguetteMaker = new BaguetteMaker();

		baguetteMaker.makeBread();
		//반죽 배합하기 ~
		//숙성하기~
		//오븐에 집어넣기~
		//토핑 더 많이 얹기~
	}
```

- 골격 구현 클래스와 이를 통한 구현

```java
public interface Washing {

	void toothBrush();

	void bodyWash();

	void hairDry();

	void faceWash();

	void doWashing();
}
/*-----------------------------------------------------------*/
public class WashingSkeleton implements Washing {
	@Override // 양치는 반드시 세게 해야 함
	public void toothBrush() {
		System.out.println("양치 구석구석 꼼꼼하게 해버리기~");
	}

	@Override
	public void bodyWash() {
		throw new UnsupportedOperationException();
	}

	@Override
	public void hairDry() {
		throw new UnsupportedOperationException();
	}

	@Override
	public void faceWash() {
		throw new UnsupportedOperationException();
	}

	@Override
	public void doWashing() {
		toothBrush();
		bodyWash();
		hairDry();
		faceWash();
	}
}
/*-----------------------------------------------------------*/
@Test
	void 스켈레톤() {
		Washing myWashing = new WashingSkeleton() {
			@Override
			public void bodyWash() {
				System.out.println("몸을 씻기는 것도 꼼꼼하게~");
			}

			@Override
			public void hairDry() {
				System.out.println("머리도 대충 말리기~");
			}

			@Override
			public void faceWash() {
				System.out.println("얼굴은 무조건 미온수로 살살 씻기~");
			}
		};

		myWashing.doWashing();
		//양치 구석구석 꼼꼼하게 해버리기~
		//몸을 씻기는 것도 꼼꼼하게~
		//머리도 대충 말리기~
		//얼굴은 무조건 미온수로 살살 씻기~
	}
```
