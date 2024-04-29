# 5장. 응집도 : 흩어져 있는 것들

## static 메서드 오용

### 왜 static 메서드를 사용할까?

static 메서드를 사용하는 이유는 **객체 지향 언어를 사용할 때, C언어 같은 절차 지향 언어의 접근 방법을 사용하려 하기 때문**입니다.
static 메서드는 **클래스의 인스턴스를 만들지 않아도 되므로** 간단하게 사용할 수 있습니다.
하지만 **응집도가 낮아지는 문제**를 일으키므로 남용하지 않는 것이 좋습니다.

- 응집도 : 클래스 내부에서 데이터와 로직의 변수가 얼마나 강한지 나타내는 지표

### 어떠한 상황에서 static 메서드를 사용해야 좋을까?

다음과 같이 **응집도에 영향을 받지 않은 경우** static 메서드를 사용해도 괜찮습니다.

- 로그 출력 전용 메서드
- 포맷 변환 전용 메서드

## 초기화 로직 분산

초기화 로직이 분산되면 **응집도가 낮은 구조**가 됩니다.
초기화 로직 분산을 막기 위해서는 **생성자를 private**로 만들고, 대신 **목적에 따라 팩토리 메서드**를 만듭니다.
그렇게 하면 클래스 내부에서만 인스턴스를 생성할 수 있으며, 인스턴스 생성을 위한 static 팩토리 메서드에서 생성자를 호출합니다.
만약 생성 로직이 너무 많아지는 것 같다면, **생성 전용 팩토리 클래스**를 분리하는 방법을 고려하는 것이 좋습니다.

## 범용 처리 클래스 (Common/Unit)

범용 처리 클래스는 **static 메서드를 빈번하게 볼 수 있는 클래스**로, 범용 처리를 위한 클래스입니다.
일반적으로 이러한 클래스에는 Common, Util이라는 이름이 붙습니다.
범용 처리 클래스는 결국 static 메서드이기 때문에 **응집도가 낮은 구조**라는 문제를 가지고 있습니다.
그러나 다음과 같은 **횡단 괌심사**(다양한 상황에서 넓게 활용되는 기능)에 해당하는 기능이라면, 범용 코드로 만들어도 괜찮습니다.

- 로그 출력
- 오류 확인
- 디버깅
- 예외 처리
- 캐시
- 동기화
- 분산 처리

## 결과를 리턴하는 데 매개변수 사용하지 않기

다음과 같이 출력으로 사용되는 매개변수를 **출력 매개변수**라고 부릅니다.

```java
class ActorManager {
    void shift(Location location, int shiftX, int shiftY) {
        location.x += shiftX;
        location.y += shiftY;
    }
}
```

이렇게 출력 매개변수를 사용하면 응집도가 낮은 구조가 됩니다.
또한, 매개변수가 입력인지 출력인지 메서드 내부의 로직을 확인해야 합니다.

매개변수는 출력 매개변수로 설계하지 말고, **'데이터'와 '데이터를 조작하는 논리'를 같은 클래스에 배치**합시다.

```java
class Location {
    final int x;
    final int y;
    
    Location(final int x, final int y) {
        this.x = x;
        this.y = y;
    }
    
    Location shift(final int shiftX, final int shiftY) {
        final int nextX = x + shiftX;
        final int nextY = y + shiftY;
        return new Location(nextX, nextY);
    }
}
```

## 매개변수가 너무 많은 경우

매개변수가 너무 많은 메서드는 응집도가 낮아지기 쉽습니다. 그리고 실수로 잘못된 값을 대입할 가능성이 높습니다.
매개변수가 너무 많아지는 문제를 피하려면, **개념적으로 의미 있는 클래스**를 만들어야 합니다.

## 메서드 체인

다음과 같이 .(점)으로 여러 메서드를 연결해서 리턴 값의 요소에 차례차례 접근하는 방법을 **메서드 체인**이라고 합니다,

```java
void equipArmor(int memberId, Armor newArmor) {
    if (party.members[memberId].equipments.canChange) {
        party.members[memberId].equipments.armor = newArmor;
    }
}
```

이렇게 구성하면 영향을 미치는 범위가 커질 수 있습니다. 또한, 메서드 체인으로 내부 구조를 돌아다닐 수 있는 설계가 됩니다.
이는 사용하는 객체 내부를 알아서는 안된다는 법칙인 데메테르의 법칙을 위반하게 됩니다.

소프트웨어 설계에는 '묻지 말고, 명령하기 (Tell, Don't Ask)'라는 유명한 격언이 있습니다.
이는 다른 객체의 내부 상태(변수)를 기반으로 판단하거나 제어하려고 하지 말고, 메서드로 명령해서 객체가 알아서 판단하고 제어하도록 설계하라는 의미입니다.

```java
class Equipments {
    // ...
    void equipArmor(final Equipment newArmor) {
        if (canChange) {
            armor = newArmor;
        }
    }
}
```