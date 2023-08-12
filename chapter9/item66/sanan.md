## 울트라 복잡한 계산아니면 네이티브 어쩌구 쓰지 마라.

~~→ 울트라 복잡한 계산인데 왜 자바 씀;;~~

## 핵심어

### JNI와 네이티브 메서드

- JNI(Java Native Interface)
- Native Method
- Exmaples
    - 플랫폼 특화 기능 사용
        
        Java에서 제공하지 않는 운영 체제 또는 하드웨어 특정 기능에 접근해야 할 수 있다. 이런 경우에 자바 네이티브 인터페이스(JNI)를 사용해 네이티브 메서드를 작성하게 되는데, 이를 통해 운영 체제의 레지스트리(시스템 구성정보 DB)를 조작하거나 특정 하드웨어 장치에 접근하는 작업을 할 수 있다.
        
    - 기존 라이브러리 사용
        
        C 라이브러리 또는 C++ 라이브러리를 자바에서 사용하는 것은 JNI를 통해 가능하다.
        예를 들어, 복잡한 수학적 계산을 수행하는 라이브러리나, 이미지 변환 처리를 위한 라이브러리 등을 자바에서 사용하려면 JNI를 이용하면 된다.
        
    - 성능 개선 예시
        
        자바의 성능이 충분하지 않아 특정 부분의 성능을 개선하고자 할 때, 그 부분을 C/C++ 등의 다른 언어로 작성할 수 있다.
        종종 이런 경우에도 JNI를 사용하게 되는데, 고성능의 수치 계산을 필요로 하는 과학 계산 응용 프로그램 등에서 일반적으로 이런 방법을 사용한다.
        ~~과학 계산을 왜 자바로 함?~~
        

---

## 요약

- Java가 업데이트 되면서 네이티브 메서드를 사용할 일은 줄고 있다.
→ 하지만 대체재가 없다면 네이티브 라이브러리를 사용해야 할 수 있다.
- 성능 개선을 목적으로 네이티브 라이브러리는 웬만하면 쓰지마라.
→ 순수 자바로도 좋은 성능 나올 수 있다.
→ 정말 고성능의 다중 정밀 연산이 필요하면 고려해봐라.
- 하지~만! 네이티브 언어 자체가 **플랫폼 종속적**이고, **인터프리트로 인한 비용**도 발생하고, **메모리 관리가 어렵다**(GC가 바라 보는 것이 아니므로).

---

## 내 생각

![sadf](https://github.com/TightJava/effective_java/assets/105692206/0aac6512-8ab6-4ca3-a273-d5d672c83352)


---

## 예제 코드

- JNI 써보기

```java
package my.effective.java.chapter9.item66;

public class HelloJni {
	static {
		System.loadLibrary("helloJni");
	}

	public static void main(String[] args) {
		new HelloJni().sayHello();
	}

	private native void sayHello();
}

//...

#include <jni.h>
#include <stdio.h>
#include "helloJni.c"

JNIEXPORT void JNICALL Java_com_example_hellojni_sayHello(JNIEnv *env, jobject thisObj)
{
    printf("쓰지 말라니까?\n");
    return;
}
```

javah 등을 이용해서 헤더를 별도로 생성하고.. 사용해야하지만 여백이 부족해 이만 줄인다.
