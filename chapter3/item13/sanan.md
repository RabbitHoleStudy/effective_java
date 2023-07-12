## 핵심어

- Mix-in Interface(from 위키)
    
    [객체 지향 프로그래밍 언어](https://ko.wikipedia.org/wiki/%EA%B0%9D%EC%B2%B4_%EC%A7%80%ED%96%A5_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D)에서 믹스인(**mixin** 또는 **mix-in**)은 다른 클래스의 부모클래스가 되지 않으면서 다른 클래스에서 사용할 수 있는 메서드를 포함하는 [클래스](https://ko.wikipedia.org/wiki/%ED%81%B4%EB%9E%98%EC%8A%A4_(%EC%BB%B4%ED%93%A8%ED%84%B0_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D))이다. 다른 클래스가 믹스인의 메소드에 액세스하는 방법은 언어에 따라 다르다. 믹스인은 때때로 "상속"이 아니라 "포함"으로 설명된다.
    
    믹스인은 [코드재사용성](https://ko.wikipedia.org/wiki/%EC%BD%94%EB%93%9C_%EC%9E%AC%EC%82%AC%EC%9A%A9)을 높이고 다중상속으로 인해 발생할 수 있는 상속의 모호성 문제("[다이아몬드 문제](https://ko.wikipedia.org/wiki/%EB%8B%A4%EC%A4%91_%EC%83%81%EC%86%8D)")를 제거하거나 언어에서 다중상속에 대한 지원부족을 해결하기 위해 사용될 수 있다. 믹스인은 구현된 [메서드](https://ko.wikipedia.org/wiki/%EB%A9%94%EC%86%8C%EB%93%9C_(%EC%BB%B4%ED%93%A8%ED%84%B0_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D))가 포함된 인터페이스로 볼 수도 있다. 이 패턴은 [종속성 역전 원칙](https://ko.wikipedia.org/wiki/%EC%9D%98%EC%A1%B4%EA%B4%80%EA%B3%84_%EC%97%AD%EC%A0%84_%EC%9B%90%EC%B9%99)을 적용하는 예가 되기도 한다.
    
- Marker Interface
Serialiazble, Cloneable, Parcelable등 메서드는 없더라도 해당 기능을 구현한다는 의미를 알려주는(Marking)의도의 인터페이스들.

---

## 요약

- clone은 원본 객체에 아무런 해를 끼치지 않는 동시에(참조, 얕은 복사X), 복제된 객체의 불변식을 보장해야 한다.
- public인 clone 메서드에서는 throws를 없애야한다. - Checked Exception X
- 상속용 클래스는 Cloneable을 구현하면 안 된다.
- 어쨌든 복제 기능은 생성자와 팩터리를 이용하게는 최고다. - 단 배열은 clone이 좋을 수 있다.

---

## 내 생각

- .clone()을 쓰기 위해서 이렇게 생각해야 할 지점도 많고, 거추장하게 사용하는 거라면, 정적 팩터리를 명확하게 쓰는게 훨씬 나을 것 같다. 팀원들 끼리도 서로 알아보기 좋을 것 같기도 하고..

---

## 예제 코드

```java

```
