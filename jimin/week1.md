## 1장. 잘못된 구조의 문제 깨닫기

### **의미를 알 수 없는 이름**
```java
class Class001 {
    void method001(); //무슨 동작을 하는지 알기 어려움
}
```
### 이해하기 어렵게 만드는 **조건 분기 중첩**
```java
if (조건) {
    if (조건) {
        if (조건) {
            //로직 구현
        }

    }
 }
```
depth가 깊어질수록 코드는 파악하기 어려워진다.

### 수많은 악마를 만들어 내는 **데이터 클래스**
  * **낮은 응집도**: 데이터 클래스와 해당 데이터를 사용하는 계산 로직이 멀리 떨어져 있는 경우
  * **코드 중복**: 관련된 코드가 분산되어 있다면, 코드 위치 파악이 힘들어 여러 곳에 같은 로직을 구현할 수 있다.
  * **수정 누락**: 코드 중복이 많으면, 사양이 변경될 때 중복 코드를 모두 고쳐야 한다.
  * **초기화되지 않은 상태(쓰레기 객체)**: 초기 값이 들어가야 하는데 들어가지 않은 경우 NPE 발생, 초기 값이 필요한 클래스인지 모른다면 버그가 발생하기 쉬워진다.
  * **잘못된 값 할당**: 데이터 클래스에서 값 할당 시 유효성 검사할 필요가 있다.

  ## 2장. 설계 첫 걸음
  1. **의도를 분명히 전달**할 수 있는 이름 설계
  2. 같은 변수에 값을 재할당하기 보다는 **목적별로 변수**를 따로 만들어 사용
  3. 단순 나열이 아닌 **의미 있는 것을 모아 메서드**로 만들기 -> 의미있는 로직이라면 메서드로 추출
  4. 관련된 데이터와 로직을 **클래스로 모으기**
        
        ```java
        class HitPoint {
            private static final int MIN = 0;
            private static final int MAX = 999;
            final int value;

            HitPoint(final int value) {
                if (value < MIN) throw new IllegalArgumentException(MIN + "이상을 지정해 주세요.");
                if (MAX < value) throw new IllegalArgumentException(MAX + "이하를 지정해 주세요.");

                this.value = value;
            }

            HitPoint damage(final int damageAmount) {
                final int damaged = value - damageAmount;
                final int corrected = damaged < MIN ? MIN : damaged;
                return new HitPoint(corrected);
            }


            HitPoint recover(final int recoveryAmount) {
                final int recovered = value + recoveryAmount;
                final int corrected = MAX < recovered ? MAX : recovered;
                return new HitPoint(corrected);
            }
        }
        ```

        * 생성자에서 값 유효성 확인
        * 데이터와 연관된 메서드인 damage, recover 로직을 클래스 내부로 이동 

