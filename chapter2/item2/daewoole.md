## 설명 - 점층적 생성자 패턴

- 필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, ... 필수 매개변수와 선택 매개변수 n개를 받는 생성자 와 같은 방식으로 생성자를 만들어
  나가는 방식


- 단점
    1. 매개변수가 많아지면 클라이언트 코드를 작성하거나 읽기 어려워진다.
    2. 매개변수가 몇 개이고 각 값의 의미가 무엇인지를 항상 파악해야 한다.
    3. 매개변수의 순서를 실수로 바꿔도 컴파일러는 알아채지 못하고 런타임에서야 발견할 수 있다.

## 예제 코드

    public class Job {
      private final String name;
      private final int salary;
      private final int workingHours;
      private final int vacationDays;

	  public Job(String name, int salary) {
        this(name, salary, 0);
	  }

	  public Job(String name, int salary, int workingHours) {
        this(name, salary, workingHours, 0);
	  }

	  public Job(String name, int salary, int workingHours, int vacationDays) {
		this.name = name;
		this.salary = salary;
		this.workingHours = workingHours;
		this.vacationDays = vacationDays;
      }
    }

## 설명 - 자바빈즈 패턴

- 매개변수가 없는 생성자로 객체를 만든 후, setter 메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식

- 단점
  1. 객체 하나를 만들려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 된다. 
  2. 클래스를 불변으로 만들 수 없으며, 스레드 안전성을 얻으려면 프로그래머가 추가 작업을 해줘야 한다.

## 질문
  1. 객체가 완전히 생성되기 전까지 일관성이 무너진다는 것이 무슨 의미인지 잘 모르겠다.

## 예제 코드

    public class Job {
	  // (선택) 매개변수들은 (필요하다면) 기본값으로 초기화
	  private String name;
      private int salary;
      private int workingHours = 0;
      private int vacationDays = 0;

      public Job() {
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public void setSalary(int salary) {
          this.salary = salary;
      }
  
      public void setWorkingHours(int workingHours) {
          this.workingHours = workingHours;
      }
  
      public void setVacationDays(int vacationDays) {
          this.vacationDays = vacationDays;
      }
    }

## 설명 - 빌더 패턴

- 필수 매개변수만으로 생성자를 호출해 빌더 객체를 만든 후, 빌더 객체가 제공하는 setter 메소드들로 원하는 선택 매개변수의 값을 설정하고, 매개변수가 없는 build 메소드로 최종적으로 (일반적으로 불변인) 객체를 생성하는 방식
- 빌더는 생성할 클래스 안에 정적 멤버 클래스로 만들어두는 것이 일반적이다.
- 빌더의 setter 메소드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다. - fluent API or method chaining 이라 부른다.
- 빌더 패턴은 명명된 선택적 매개변수(named optional parameters)를 흉내 낸 것이다.
- 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.


- 빌더 패턴에서의 유효성 검사
  - 빌더의 생성자와 setter 메서드에서 입력 매개변수를 검사하고, build 메서드가 호출하는 생성자에서 여러 매개변수에 대한 불변식을 검사한다.
  - 공격에 대비해 이런 불변식을 보장하려면 빌더로부터 매개변수를 복사한 후 해당 객체 필드들도 검사해야 한다.


- 불변식(invariant) : 객체의 생성과 파괴가 일어날 때까지 항상 참인 조건
  - 변경을 허용할 수는 있으나 주어진 조건 내에서만 허용한다는 의미
  - ex) 리스트의 크기는 반드시 0 이상이어야 한다. / 기간을 표현하는 Period 클래스에서 start 필드의 값은 반드시 end 필드의 값보다 앞서야(작아야) 한다.


- 공변 반환 타이핑 (covariant return typing)
  - 하위 클래스의 메서드가 상위 클래스의 메서드가 정의한 반환 타입이 아닌, 그 하위 타입을 반환하는 기능


- 변성(variance) - 타입 간에 서로 어떤 관계가 있는지를 나타내는 개념
  - java generic에서는 3가지 가변성 성질을 제공한다.
  - 공변(covariance) : 타입의 계층 관계가 그대로 유지되는 성질 ex) A가 B의 하위 타입일 때, T<A>는 T<B>의 하위 타입이다.
  - 반공변(contravariance) : 타입의 계층 관계가 반대로 유지되는 성질 ex) A가 B의 하위 타입일 때, T<B>는 T<A>의 하위 타입이다.
  - 무공변(invariance) : 타입의 계층 관계가 유지되지 않는 성질 ex) A가 B의 하위 타입이라고 해도, T<A>와 T<B>는 아무 관계가 없다.
  - 이러한 3가지 공변들로 메소드의 인자로 들어오는 파라미터의 타입에 제한을 걸 수 있다.
  - 공변은 upperbound로 그 객체의 하위 타입만 허용하는 것이고, 반공변은 lowerbound로 그 객체의 상위 타입만 허용하는 것이다.
  - 이러한 타입 관계는 객체지향 프로그래밍 원칙 중 하나인 "리스코프 치환 원칙"에 해당하며, 이 원칙은 상위 타입이 사용되는 곳에는 하위 타입의 인스턴스를 넣어도 이상 없이 동작해야 함을 의미한다.


- 빌더 패턴의 장점
  1. 빌더를 이용하면 가변인수(varargs) 매개변수를 여러개 사용할 수 있다. (가변인수는 타입이 같은 매개변수를 0개 이상 받을 수 있게 해준다.) 각각을 적절한 메서드로 나눠 선언하거나 같은 메서드를 여러 번 호출하면서 넘겨진 매개변수들을 하나의 필드로 모을 수도 있다.
  2. 유연하다 -> 빌더 하나로 여러 객체를 순회하면서 만들 수 있고, 빌더에 넘기는 매개변수에 따라 다른 객체를 만들수도 있다.


- 빌더 패턴의 단점
  1. 객체를 생성하려면 먼저 그 객체를 만들 빌더부터 생성해야 하기 때문에 빌더 생성 비용이 클 경우, 성능 저하의 원인이 될 수 있다.
  2. 매개변수가 일정 개수 이상 (보통 4개) 되어야 코드가 길어지는 비용을 감수하고 사용하는 가치가 있다. - API는 시간이 지날수록 매개변수가 많아지는 경향이 있기 때문에 builder pattern을 사용하는 것이 좋다.


- 질문
  1. 빌더의 생성자와 setter 메서드에서 입력 매개변수에 대한 불변식을 검사하면 되지 않나? 왜 1차적으로 검사하고, 이후에 다시 한번 build 메서드가 호출하는 생성자에서 불변식을 검사해야 하는지 모르겠다.
  2. 공변 반환 타이핑은 결국 하위 클래스의 메서드가 상위 클래스의 메서드를 overriding 하였다는 의미 아닌가? 이 외의 추가적인 의의가 있는가?
  3. 빌더 하나로 여러 객체를 순회하면서 만들 수도 있다는 것이 어떤 의미인지 잘 모르겠다.

## 예제 코드

    public class Job implements AutoCloseable {
        private final String name;
        private final int salary;
        private final int workingHours;
        private final int vacationDays;

        public static class Builder {
            // 필수 매개변수
            private final String name;
            private final int salary;
    
            // 선택 매개변수 - 기본값으로 초기화
            private int workingHours = 0;
            private int vacationDays = 0;
    
            public Builder(String name, int salary) {
                this.name = name;
                this.salary = salary;
            }
    
            public Builder setWorkingHours(int workingHours) {
                this.workingHours = workingHours;
                return this;
            }
    
            public Builder setVacationDays(int vacationDays) {
                this.vacationDays = vacationDays;
                return this;
            }
    
            public Job build() {
                return new Job(this);
            }
        }
    
        private Job(Builder builder) {
            name = builder.name;
            salary = builder.salary;
            workingHours = builder.workingHours;
            vacationDays = builder.vacationDays;
        }
    
        @Override
        public void close() throws Exception {
            System.out.println("Job is closed.");
        }
    }






