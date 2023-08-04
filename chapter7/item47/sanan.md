**한 줄 요약 : Stream이나 Iterable말고 Collection으로 반환해라.**

## 핵심어

---

## 요약

- API 사용자에게 Collection이나 그 하위 타입인 List, Set 등을 반환하도록 노력해라.
→ Stream이든 Iterable이든 원하는대로 사용할 수 있기 때문!
- 반환 전부터 이미 원소들을 컬렉션에 담아 관리하고 있거나 컬렉션을 하나 더 만들어도 될 정도로 원소 개수가 적다면, ArrayList 같은 표준 컬렉션에 담아 반환하라. 
그렇지 않으면 전용 컬렉션을 구현할지 고민하라.
- 컬렉션을 반환하는 게 불가능하면 스트림과Iterable중 더 자연스러운 것을 반환하라.

---

## 내 생각

- 결국 현재는 Iterable을 사용하거나, Stream을 사용할 수 있는 방법을 선택할 때 양쪽 다 가능한 Collection을 사용하지만, Iterable이 Stream을 통해서도 가능해지면 Stream을 사용하라는 얘기가 무슨 말인지 알 것 같다.
→ 결국 Iterable을 이용한 ForEach를 쓸지, Stream을 이용한 유연한 데이터 처리를 사용할지는 사용자의 마음이니까.

---

## 예제 코드

```java

```
