# [아이템 26.5] 제네릭

![img](https://github.com/TightJava/effective_java/assets/13278955/41f65f3b-7036-4452-a66a-18a66916878b)

# 제네릭 그게 뭔데?

타입에 <T> 로 주어, 타입을 제네릭 하게 사용할 수 있다.

Object 로 쓰는것과 유사한 효과이며, 아래 왜 써야하는지 설명하겠다.

# 타입 안정성 & 코드 가독성

- 컴파일 타임에 ClassException 휴먼 에러를 잡을 수 있다
- 강제 형변환이 필요없다.
- 코드로 보자

## 시나리오

- 세상에는 총과 검 두개의 무기가있다.
- 내 주머니에는 총 홀스터와 검케이스 두개가 있다
- 그걸 각각 꺼내는 시나리오다

- MyPocket 클래스에서 getWeapon 이라는 메소드로 둘 다 꺼내는걸 하나의 메소드로 정의할것이다.  따라서 Sword , Gun 둘다 상위 타입인 Object 로 선언했다.

## 공통

```java
class Gun {
	void shot(){
		System.out.println("쏴");
	}
}

class Sword {

}

class MyPocket{
	private Object weapon;
	public MyPocket(Object weapon) {
		this.weapon = weapon;
	}
	public Object getWeapon() {
		return weapon;
	}
}
```

## Object Class 사용

```java
public class TypeSafeTest  {

	public static void main(String[] args) {
		MyPocket gunHolster = new MyPocket(new Gun());
		MyPocket swordCase = new MyPocket(new Sword());

		Gun gun = (Gun) gunHolster.getWeapon();
		Sword sword = (Sword) gunHolster.getWeapon(); // 컴파일 에러 
	}

}

```

- Object 를 썼을때 ⇒ 런타임 에러
- 건 홀스터에서 검을 꺼낸다고 선언했으니, 에러가 난다 하지만 런타임 에러

```java
Exception in thread "main" java.lang.ClassCastException: 
class test.generic.typesafe.Gun cannot be cast to class test.generic.typesafe.Sword (test.generic.typesafe.Gun and test.generic.typesafe.Sword are in unnamed module of loader 'app')
```

## Generic  Class 사용

```java
class MyDoramonPocket<T>{
	private T weapon;
	public MyDoramonPocket(T weapon) {
		this.weapon = weapon;
	}
	public T getWeapon() {
		return weapon;
	}
}
public class TypeSafeTest  {

	public static void main(String[] args) {
		MyDoramonPocket<Gun> gunHolster = new MyDoramonPocket<Gun>(new Gun());
		MyDoramonPocket<Sword> swordCase = new MyDoramonPocket<Sword>(new Sword());

		Gun gun = gunHolster.getWeapon();
		Sword sword = gunHolster.getWeapon();
	}
```

![스크린샷 2023-07-22 오전 1 54 26](https://github.com/TightJava/effective_java/assets/13278955/1f0058d8-ef65-4446-98f4-9a2390908cd1)

⇒ 컴파일 타임에 에러가 잡힌다

⇒ 강제 형변환 필요가 없다 

---

# 메서드에서 사용한 제네릭

- 사용 방법은 class 와 같이 <T> 를 붙여준다

```java
public class GenericMethods {
	class Printer<T> {
		public <T> void printClassType(T e) {
			System.out.println(e.getClass().getSimpleName());
		}
	}
	public static void main(String[] args) {
		GenericMethods m = new GenericMethods();
		Printer<String> printer = m.new Printer<String>();
		printer.printClassType(1234);
		printer.printClassType(3.14);
		printer.printClassType("HI");
	}
}
```

```java
// 실행결과
Integer
Double
String
```

- 제네릭 메소드의 타입 매개변수가 같다면 제네릭 메소드의 타입 매개변수를 우선한다.
    - Printer 는 String 선언했지만, 사용되는 메소드에서 Integer Double 로 사용된다.

---

# ? WildCard 와일드카드

```java
public class WildCard {

	public static void printing(List<Object> list){
		for (Object object : list) {
			System.out.println(object);
		}
		System.out.println();
	}
	public static void main(String[] args) {
		List<Object> objectList = new ArrayList<>();
		objectList.add("a");
		objectList.add("b");
		objectList.add("c");
		printing(objectList);

		List<String> stringList = new ArrayList<String>();
		printing(stringList); // 컴파일 에러 
//java: incompatible types: java.util.List<java.lang.String> cannot be converted to java.util.List<java.lang.Object>
	}
}
```

- String 은 Object 의 자식이지만 
List<String> 은 List<Object>의 자식이 아니다.

    ![a8175404b1b66a62f393f1270867ac71](https://github.com/TightJava/effective_java/assets/13278955/3964c9ce-f029-4ffa-ad70-56491d71c8cf)
    
- 따라서 매개변수가 안맞는다.
List<Object> 에 List<String> 은
Integer 매개변수에 String 을 넣은것과 다를바 없는것

```java
public static void printingWithWild(List<?> list){
		for (Object object : list) {
			System.out.println(object);
		}
		System.out.println();
	}

	public static void main(String[] args) {

		List<String> stringList = new ArrayList<String>();
		stringList.add("x");
		stringList.add("y");
		stringList.add("z");
		printingWithWild(stringList);
	}
```

- List<String> 혹은 List<T> 는 List<?> 의 자식이다.
- 제네릭 계의 Object 인 셈이다.
정확히는
몰?루 타입 혹은 모든 타입이라고 한다.
- List<?> 에서 get 한 객체는 Object 다 ⇒ ? 는 최상위 이기때문에

  
![3cAO5T](https://github.com/TightJava/effective_java/assets/13278955/2eea77b9-cebf-4dbb-8d9f-29b8aa802b87)
- 와일드 카드는 Getter 의형태로 사용되며, 값을 대입하는게 불가능하다 ⇒ 타입이 뭔지 모르니까 당연한것

---

# 제한된 제네릭

## 상한 경계 제네릭 upper bound

`<T extends Type>`

`Type` 과 `Type` 의 자손 타입만 가능하도록 제한한다.

### with Wildcard

`<? extends Type>`

`Type` 의 자손타입을 보장하나 어떤 타입인지 알 수 없다.

```java
<T> SimpleList<T> filterNegative(SimpleList<? extends Number> simpleList){
	SimpleList<T> tempList = new SimpleArrayList<>();
	for (int i = 0; i < simpleList.size(); i++) {
		if (simpleList.get(i).doubleValue() >= 0)
				tempList.add((T)simpleList.get(i));
	}
	return tempList;
}
```

- simpleList 는 read 만 가능 Number의 하위 타입 가능 ⇒ Integer, Double
- tempList 는 T 이면 add 가 가능

## 하한 경계 제네릭 lower bound

`<? super Type>`

`Type` 과 `Type`의 부모/조상 타입만 대입 가능하도록 제한한다.

```java
class Printer { }
class LaserPrinter extends Printer{}

List<Printer> printers = new ArrayList<>();
LaserPrinter laserPrinter = new LaserPrinter();
List<LaserPrinter> laserPrinters = new ArrayList<>(laserPrinter);

~.copy(laserPrinters, printers);

void copy(List<? extends T> laserPrinters, List<? super T> printers){
	for (int i = 0; i < laserPrinters.size(); i++)
		printers.add(laserPrinters.get(i));
}
```

하한과 상한을 모두 사용한 예제

하한에서는 T (Printer 클래스)의 add 사용

상한에서는 T (LaserPrinter 클래스 ) get사용

---
