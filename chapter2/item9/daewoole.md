## 설명 - try-with-resources

- try-with-resources
  - try-finally 의 문제점을 상당 부분 해결함
  - 이 구조를 사용하려면 해당 자원이 AutoCloseable 인터페이스를 구현해야 함
  - try 블록을 벗어날 때 (자원을 회수하는) close 메서드가 자동으로 호출됨
  - try 문에서 실행한 코드와 close 메서드 호출 양쪽에서 예외가 발생하면, close 메서드에서 발생한 예외는 숨겨지고, try 문에서 발생한 예외가 기록되기 때문에 디버깅이 수월해진다.
  - 위에서 숨겨진 예외도 stack trace에 '숨겨졌다'는 꼬리표를 달고 출력된다.
  - catch 절을 같이 쓸 수 있다.


- try-finnaly 문의 문제점
  1. 자원을 여러 개 사용하는 경우에 코드가 지저분해진다.
  2. try 문에서 예외가 발생하고, finally 문에서도 예외가 발생하였을 경우, 두 번째 예외가 첫 번째 예외를 완전히 집어삼키게 되기 때문에 stack trace에 첫 번째 예외에 관한 정보가 남지 않아서 디버깅이 어려워진다.


- Autocloseable 인터페이스
  - close 메서드를 호출해 직접 닫아줘야 하는 자원을 나타내는 인터페이스
  - void 반환 타입의 close 메서드 하나만 정의되어 있는 인터페이스

## 예제 코드

    import item2.builderPattern.Job;

    public class TryWithResources {
        public static void makeDeveloper() {
            try (Job job = new Job.Builder("developer", 50000000)
                .setWorkingHours(8).setVacationDays(15).build()) {
                System.out.println("makeDeveloper");
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
