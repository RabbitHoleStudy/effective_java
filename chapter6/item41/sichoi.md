# 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

- **마커 인터페이스(marker interface)란?**
    
    **마커 인터페이스(marker interface)**는 메소드 선언 없이 특정 클래스가 특정한 속성을 가진다는 것을 나타내는데 사용되는 인터페이스입니다. 이는 주로 메타데이터(metadata)의 형태로, 클래스가 특정 인터페이스를 구현함으로써 특정한 기능 또는 행동을 수행하도록 강제합니다.
    Java에서 가장 대표적인 마커 인터페이스의 예로는 **Serializable**과 **Cloneable**이 있습니다.
    예를 들어, **Serializable** 인터페이스를 구현한 객체는 JVM(JAVA Virtual Machine)에 의해 직렬화를 수행해서 네트워크로 전송하거나 파일 시스템에 저장할 수 있습니다.
    
    ```java
    import java.io.Serializable;
    
    public class MyClass implements Serializable {
        private int a;
    
        public int getA() {
            return this.a;
        }
    
        public void setA(int a) {
            this.a = a;
        }
    }
    ```
    
    이 예제에서 **MyClass**는 **Serializable**을 구현하여 직렬화를 수행할 수 있는 타입으로 간주됩니다. 따라서 이 클래스의 객체는 문자열로 변환되거나 파일에 저장되어 추후에 다시 객체로 복원할 수 있습니다.
    이처럼 마커 인터페이스는 특정 클래스가 특별한 속성을 가지고 있음을 나타내는 용도로 사용됩니다. 단순히 원하는 인터페이스를 구현한다고 해서 특정 기능이 자동으로 추가되는 것은 아닙니다, 구현하는 클래스가 실질적으로 그 기능을 제공해야 합니다.
    
1. 아무 메소드를 담고 있지 않고
2. 자신을 구현하는 클래스가 특정 속성을 가짐을 **표시**

표시만 하는 것 뿐이지, 마커 인터페이스를 implements 한다고하여, 해당 기능이 자동 추가되는 것은 아니다.

구현 클래스에서 해당 기능을 실질적으로 제공하는 코드를 작성해주어야 한다.

`마커 애너테이션`이 도입되면서 마커 인터페이스가 outdated된 방식이라고 착각할 수 있지만, 사실이 아니라고 한다.

### 마커 인터페이스 > 마커 애너테이션

1. **마커 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있지만, 마커 애너테이션을 그렇지 않다.**
    
    마커 인터페이스도 **타입이기 때문에**, 마커 애너테이션을 이용했다면 런타임에 발견될 수도 있는 에러를 컴파일 타임에 잡을 수 있다.
    
2. **마커 애너테이션보다 적용 대상을 더 정밀하게 지정할 수 있다.**

### 마커 인터페이스 < 마커 애너테이션

애너테이션을 적극 활용하는 프레임워크를 사용하는 경우 마커 애너테이션을 사용하는 쪽이 일관성을 지키는데 유리하다.

→ 너무 스프링 얘기이다. 스프링 프레임워크에서 `@Component`, `@Service`, `@Repository`, `@Controller` 등의 애너테이션들은 클래스에 특별한 의미를 부여하거나, 스프링 컨테이너에 의해 빈(Bean)으로 관리될 수 있도록 해준다.

**클래스나 인터페이스를 마킹해야 하는 경우, 마킹이 된 객체를 매개변수로 받는 메소드를 작성할 일이 있을 경우, 한 마디로 타입을 정의해야 하는 상황인 경우**

→ 마커 인터페이스를 사용 (클래스나 인터페이스만이 인터페이스를 확장할 수 있기 때문, 그것이 인터페이스니까..!)

**그 외에 요소들(모듈, 패키지, 필드, 지역변수)를 마킹해야 하는 경우**

→애너테이션 사용

### 저자의 핵심 정리

**마커 인터페이스**와 **마커 애너테이션**은 각자의 쓰임이 있다.

**새로 추가하는 메소드 없이 단지 타입 정의가 목적이라면 마커 인터페이스**를 선택하자.

**클래스나 인터페이스 외의 프로그램 요소에 마킹해야 하거나, 애너테이션을 적극 활용하는 프레임워크의 일부로 그 마커를 편입시키고자 한다면 마커 애너테이션이 올바른 선택**이다.

적용 대상이 **ElementType.TYPE**인 마커 애너테이션을 작성하고 있다면, 잠시 여유를 갖고 정말 애너테이션으로 구현하는게 옳은지, 혹은 마커 인터페이스가 더 낫지는 않을지 곰곰히 생각해보자.