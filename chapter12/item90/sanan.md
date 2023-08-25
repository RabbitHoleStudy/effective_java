## 핵심어

- 직렬화 프록시 패턴(Serialization Proxy Pattern)
    
    Serializable 인터페이스를 구현하는 클래스는 직렬화 통해 파일에 저장하거나 네트워크를 통해 전송할 수 있다. 한편, 직렬화는 예상치 못한 문제를 일으킬 수 있다.
    
    예를 들어, 객체의 private 필드조차도 직렬화 할 수 있기 때문에 객체의 불변성을 깨트릴 수 있다는 단점이 있다. 그리고 이전에 이미 여러가지를 둘러 보았다.
    
    이를 해결하기 위해 직렬화 프록시 패턴을 사용할 수 있다. 이 패턴은 직접 객체를 직렬화하지 않고, 대신 별도의 private static 중첩 클래스인 프록시 객체를 만들어 이것을 직렬화한다.
    이 프록시 클래스는 **원본 객체의 논리적 상태만을 복제해, 상태만 저장**한다.
    
    따라서 원본 객체의 불변성은 적절하게 유지되면서, 동시에 직렬화가 가능하게 된다.
    또한 다시 역직렬화 시에는 프록시가 원본 객체를 생성하여 리턴하게 된다.
    → 말그대로 대역(proxy)을 사용하는 것이다.
    
- 커스텀 직렬화
    
    <img width="1448" alt="스크린샷 2023-08-25 오후 12 41 36" src="https://github.com/TightJava/effective_java/assets/105692206/8a0ae991-6bb5-4d53-97b8-1b4bec7b3eaf">

    
    위 흐름에서 해당하는 메서드를 우리가 원하는 객체에 작성하면 직렬화를 커스텀 할 수 있다.
    
    **ObjectInputStream.readObject()**: 이 메서드를 호출하여 객체를 역직렬화한다.
    
    **readObject(ObjectInputStream in)**: 객체에 이 메서드가 정의되어 있다면, readObject()는
    
    ObjectInputStream.readObject()에 의해 호출되고, 커스텀한 역직렬화 로직을 수행한다.
    
    **readResolve()**: 이 메서드는 역직렬화 직후에 호출되며, 반환된 객체가 역직렬화한 결과로 반환된다. 직렬화 프록시 패턴에서 사용한다.
    
    ---
    
    **ObjectOutputStream.writeObject(Object obj)**: 이 메서드를 호출하여 **Object**를 직렬화한다.
    
    **writeObject(ObjectOutputStream out)**: 객체에 이 메서드가 정의되어 있다면, writeObject()는
    
    ObjectOutputStream.writeObject()에 의해 호출되며, 커스텀 직렬화 로직을 수행한다.
    
    **writeReplace()**: 이 메서드는 직렬화 전에 호출되어, 반환된 객체가 실제로 직렬화 과정에 사용된다. 마찬가지로 직렬화 프록시 패턴에서 사용한다.
    

---

## 요약

<img width="1229" alt="스크린샷 2023-08-25 오후 1 01 36" src="https://github.com/TightJava/effective_java/assets/105692206/70fe3327-a66d-40e2-a6bc-6430dafce89e">


- 직렬화를 하고 싶을 때 확장할 수 없는 클래스라면 웬만해서 직렬화 프록시 패턴을 사용해라.

---

## 참고자료

[The Serialization Proxy Pattern // nipafx](https://nipafx.dev/java-serialization-proxy-pattern/)
