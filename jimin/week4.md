## 5장. 응집도: 흩어져 있는 것들
## 5.1 static 메서드 오용

### static 메서드 개선
```java
class OrderManager {
	static int add(int moneyAmount1, int moneyAmount2) {
		return moneyAmount1 + moneyAmount2;
	}
}

//static 호출, 데이터 클래스와 함께 사용 
moneyData1.amount = OrderManager.add(moneyData1.amount, moneyData2.amount);
```
* 데이터와 로직이 서로 다른 클래스에 작성되면서 **응집도가 낮아진다.**
* static 메서드는 **인스턴스 변수를 사용할 수 없으므로** 등집도가 낮아질 수 밖에 없다. 
  * 따라서 인스턴스 변수를 사용하도록 설계를 변경하는 것이 좋다.  
    ```java
    class Money {
        final int amount; //인스턴스 변수로 이동

        Money add(final Money other) { 
            final int added = amount + other.amount;
            return new Money(added); 
        }
    }
    ```

### 왜 사용할까? 
* 객체 지향 언어를 사용할 때, 절차 지향 언어의 접근 방법을 사용하려 하기 때문
  * 절차 지향 언어에서는 데이터와 로직이 따로 존재하도록 설계한다.
* static 메서드는 인스턴스를 만들지 않아도 되므로, 간단하게 사용할 수 있지만 응집도가 낮아지는 문제를 일으키므로 남용하지 않는 것이 좋다.


### 언제 사용하면 좋을까?
* 로그 출력 전용 메서드, 포맷 변환 전용 메서드처럼 응집도와 관계없는 기능을 staitc 메서드로 설계하면 좋다.

## 5.2 초기화 로직 분산
상황에 따라 초기화 값(ex. 표준회원인 경우 3000포인트, 프리미엄 회원인 경우 10000포인트)이 다른 경우, 로직이 분산되므로 유지보수가 어려워진다. 

### private 생성자 + 정적 팩토리 메서드 사용해 목적에 따라 초기화

```java
class GiftPoint {
  private static final int MIN_POINT = 0;
  private static final int STANDARD_MEMBERSHIP_POINT = 3000;
  private static final int PREMIUM_MEMBERSHIP_POINT = 10000;
  final int value;

  private GiftPoint(final int point) { //생성자는 private으로 정의
    if (point < MIN_POINT) {
      throw new IllegalArgumentException("포인트는 0 이상");
    }

    value = point;
  }

  static GiftPoint forStandardMembership() { //정적 팩토리 메서드
    return new GiftPoint(STANDARD_MEMBERSHIP_POINT);
  }

  static GiftPoint forPremiumMembership() {
    return new GiftPoint(PREMIUM_MEMBERSHIP_POINT);
  }
}

//값으로 초기화
GiftPoint standardMemberShipPoint = new GiftPoint(3000);

//정적팩토리메서드 사용
 GiftPoint standardMemberShipPoint = GiftPoint.forStandardMembership();
```

## 5.3 범용 처리 클래스
Common, Util 이라는 이름으로 범용 처리 클래스를 만들곤 하는데, 관련 없는 범용 처리가 한 클래스에 모이게 된다.
재사용을 위해 범용 처리 클래스에 넣기보다는, **응집도를 높여 재사용을 높이자**,,

### 횡단 관심사
다양한 상황에서 넓게 활용되는 기능을 횡단 관심사라고 하는데, 이러한 코드라면 범용 처리 클래스로 빼는 것은 나쁘지 않다.

횡단 관심사 코드는 다음과 같다.
* 로그 출력, 오류 확인, 디버깅, 예외처리, 캐시, 동기화, 분산처리

## 5.4 결과를 리턴하는데 매개변수 사용하지 않기

```java
class ActorManager {
  void shift(Location location, int shiftX, int shiftY) {
    location.x += shiftX;
    location.y += shiftY;
  }
}
```
매개변수를 출력으로 사용해 버리면, 메서드 내부를 확인해야 한다. 따라서 객체 지향 설계답게..! 데이터와 데이터를 조작하는 논리를 같은 클래스에 배치하는 것이 좋다.

```java
class Location {
  final int x;
  final int y;

  Location shift(final int shiftX, final int shiftY) {
    final int nextX = x + shiftX;
    final int nextY = y + shiftY;
    return new Location(nextX, nextY);
  }
}
```

## 5.5 매개변수가 너무 많은 경우
매개변수가 많다는 것은 많은 기능을 처리하고 싶다는 것이다. 처리할게 많아지면 로직이 복잡해지고 중복 코드가 많아진다.

### 기본 자료형에 대한 집착
기본 자료형으로만 구현하면, 중복 코드가 많이 발생할 수 있다.

```java
class Common {
  int discountedPrice(int regularPrice, float discountRate) {
    if (regularPrice < 0) {
      throw new IllegalArgumentException(); //중복
    }
    ...
  }
}

class Util {
  boolean isFairPrice(int regularPrice) {
    if (regularPrice < 0) {
      throw new IllegalArgumentException(); //중복
    }
    return true;
  }
}
```

따라서 클래스에 캡슐화 하면 중복 코드를 줄일 수 있다.

```java
class RegularPrice {
  final int amount;

  RegularPrice(final int amount) {
    if (amount < 0) {
      throw new IllegalArgumentException();
    }
    this.amount = amount;
  }
}
```

### 의미 있는 단위는 모두 클래스로 만들기
매개변수가 많다면 의미 있는 단위를 클래스로 빼서 잘 설계해보자..

## 5.6 메서드 체인

```java
void equipArmor(int memberId, String newArmor) {
    if (party.members(memberId).equipments(memberId).canChange(memberId)) {
        party.members(memberId).equipments(memberId).armor = newArmor;
    }
}
```
메서드 체인으로 클래스의 꽤 깊은 요소까지 접근하고 있다. 만약 저 체인 중 일부가 변경된다면 해당 요소에 접근하고 있던 코드들을 모두 수정해야 한다. 

메서드 체인보다는, 객체 내부를 모르게 작성하는 것이 좋다.

### 묻지 말고 명령하기 
다른 객체의 내부 상태를 기반으로 판단하거나 제어하려고 하지말고, **메서드로 명령**해서 객체가 알아서 판단하고 제어하도록 설계하는 것이 좋다.

```java
class Equipments {
  private boolean canChange;
  private Equipment head;
  private Equipment armor;
  private Equipment arm;

  void equipArmor(final Equipment newArmor) { //갑옷 장비 
    if (canChange) {
      armor = newArmor;
    }
  }

  void deactivateAll() { //전체 장비 해제
    head = Equipment.Empty;
    armor = Equipment.Empty;
    arm = Equipment.Empty;
  }
}
```
