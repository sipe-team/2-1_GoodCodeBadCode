# 6장. 조건 분기 : 미궁처럼 복잡한 분기 처리를 무너뜨리는 방법

조건 분기는 **조건에 따라 처리 내용을 전환하는 데 사용되는 프로그래밍의 기본 제어 구조**입니다.
이러한 조건 분기는 **조건이 복잡해지면 코드의 동작을 이해하기 어렵다**는 단점이 있습니다.

## 조건 분기 중첩

조건 분기를 중첩하면 **코드의 가독성이 크게 떨어지는 문제**가 생깁니다.
이러한 조건 분기의 중첩을 제거하기 위한 방법 중 하나는 **조기 리턴(early return)** 입니다.

조기 리턴은 **조건을 만족하지 않는 경우 곧바로 리턴하는 방법**입니다.
조기 리턴을 사용하면 중첩이 제거되어 가독성이 좋아지며, **조건 로직과 실행 로직을 분리**할 수 있다는 장점도 있습니다.
그로인해 매우 간단하게 로직을 추가할 수 있게 됩니다.

```java
// AS-IS
if (member.hitPoint) {
    if (member.canAct())  {
        if (magic.costMagicPoint <= member.magicPoint) {
            member.consumeMagicPoint(magic.costMagicPoint);
            member.chant(magic);
        }
    }
}

// TO-BE
if (member.hitPoint <= 0) return;
if (!member.catAct()) return;
if (member.magicPoint < magic.costMagicPoint) return;

member.consumeMagicPoint(magic.costMagicPoint);
member.chat(magic);
```

## switch 조건문 중복

switch 조건문 중복이 많아지면, **수정 누락**과 **개발 생산성 하락**이 발생할 수 있습니다.
switch 조건문 중복을 해소하려면, **단일 책임 선택의 원칙**을 생각해봐야 합니다.
단일 책임 선택 원칙은 <<객체 지향 소프트웨어 설계 2판 원칙과 개념>>에서 "소프트웨어 시스템이 선택지를 제공해야 한다면, 그 시스템 내부의 어떠한 모듈만으로 모든 선택지를 파악할 수 있어야 한다"고 설명하고 있습니다.

단일 책임 선택의 원칙에 따라 중복이 발생한 switch 조건문을 하나로 모읍니다.
이렇게 한 곳에 구현되어 있으면, **사양을 변경할 때 누락 실수를 줄일 수 있습니다.**

```java
class Magic {
    // 생략
    
    Magic(final MagicType magicType, final Member member) {
        switch (magicType) {
            case fire:
                name = "파이어";
                costMagicPoint = 2;
                attackPower = 20 + (int) (member.level * 0.5);
                costTechnicalPoint = 0;
                break;
            case lightning :
                // 생략
            case hellFire :
                // 생략
            default:
                throw new IllegalArgumentException();
        }
    }
}
```

하지만 클래스가 거대해지면 데이터와 로직의 관게를 알기 힘들어집니다.
그래서 **유지 보수와 변경이 어려운 코드**가 됩니다. 이러한 문제를 해결할 때는 **인터페이스를 사용**합니다.

인터페이스를 사용하면, 분기 로직을 작성하지 않고도 분기와 같은 기능을 구현할 수 있습니다.
switch 조건문 중복 문제를 인터페이스를 응용해 해결하는 방법(전략 패턴)은 다음과 같습니다.
1. 종류별로 다르게 처리해야 하는 기능을 인터페이스의 메서드로 정의 
   ```java
   String name();
   int costMagicPoint();
   int attackPower();
   int costTechnicalPoint();
   ```
2. 인터페이스의 이름을 결정하는 방법 : '어떤 부류에 속하는가?'
   - e.g. Rectangle, Circle, Triangle → Shape
   - e.g. Fire, Lightning, HellFire → Magic 
     ```java
     interface Magic {
       String name();
       int costMagicPoint();
       int attackPower();
       int costTechnicalPoint();
     }
     ```
3. 종류별로 클래스 만들기 : `Fire`, `Lightning`, `HellFire`
4. 각각의 클래스에 인터페이스 구현하기
    ```java
     class Fire implements Magic {
       // 생략
     }
     ```
5. switch 조건문이 아니라, Map으로 변경하기
    ```java
    final Map<MagicType, Magic> magics = new HashMap<>();
    // 생략
    final Fire fire = new Fire(member);
    magics.put(MagicType.fire, fire);
    // 같은 방법으로 Lightning, HellFire 추가
    ```
6. 메서드를 구현하지 않으면 오류로 인식하게 만들기
   - 인터페이스의 메서드는 반드시 구현되어야 컴파일 할 수 있음 → 구현하지 않는 실수 자체를 방지
7. 값 객체화하기
     ```java
     interface Magic {
       String name();
       MagicPoint costMagicPoint();
       AttackPower attackPower();
       TechnicalPoint costTechnicalPoint();
     }
     ```

## 조건 분기 중복과 중첩

인터페이스는 switch 조건문의 중복을 제거할 수 있을 뿐만 아니라, **다중 중첩된 복잡한 분기를 제거**하는 데 활용할 수 있습니다.

```java
// 조건 분기 중첩
boolean isGoldCustomer(PurchaseHistory history) {
    if (1000000 <= history.totalAmount) {
        if (10 <= history.purchaseFrequencyPerMonth) {
            if (history.returnRate <= 0.001) {
                return true;
            }
        }
    }
    return false;
}

// 위 조건 분기와 중복된 부분이 있음
boolean isSilverCustomer(PurchaseHistory history) {
    if (10 <= history.purchaseFrequencyPerMonth) {
        if (history.returnRate <= 0.001) {
            return true;
        }
    }
    return false;
}
```

이렇게 같은 로직을 재사용하려면 **정책 패턴(policy pattern)** 으로 조건을 집약하면 됩니다.
정책 패턴이란 조건을 부품처럼 만들고, 부품으로 만든 조건을 조합해서 사용하는 패턴입니다.

다음과 같이 하나하나의 규칙(조건)을 나타내는 인터페이스를 생성합니다.

```java
interface ExcellentCustomerRule {
    boolen ok(final PurchaseHistory history);
}
```

그리고 이 인터페이스를 구현한 여러 규칙 클래스를 생성합니다.

```java
class GoldCustomerPurchaseAmountRule implements ExcellentCustomerRule {
    public boolean ok(final PurchaseHistory history) {
        return 1000000 <= history.totalAmount;
    }
}
// PurchaseFrequencyRule, ReturnRateRule 생략
```

그 다음 `add` 메서드로 규칙을 집약하고, `complyWithAll` 메서드 내부에서 규칙을 모두 만족하는지 판정하는 **정책 클래스**를 만듭니다.

```java
class ExcellentCustomerPolicy {
    private final Set<ExcellentCustomerRule> rules;
    
    ExcellentCustomerPolicy(){
        rules = new HashSet();
    }
    
    void add(final ExcellentCustomerRule rule) {
        rules.add(rule);
    }
    
    boolean complyWithAll(final PurchaseHistory history) {
        for (ExcellentCustomerRule each : rules) {
            if (!each.ok(history)) return false;
        }
        return true;
    }
}
```

## 자료형 확인에 조건 분기 사용하지 않기

`RegularRates`와 `PremiumRates`라는 인터페이스 구현 클래스를 다음과 같이 자료형을 확인해 분기하는 것은 좋지 않은 코드입니다.
이렇게 작성하게 되면 비슷한 형태의 조건 분기가 계속 작성되어, 조건 분기 코드 중복 문제가 발생할 수 있습니다.

```java
if (hotelRates instanceof RegularRates) {
    busySeasonFee = hotelRates.fee().add(new Money(30000));
} else if (hotelRates instanceof PremiumRates) {
    busySeasonFee = hotelRates.fee().add(new Money(50000));
}
```

이와 같은 로직은 **리스코프 치완 원칙**을 위반합니다. 리스코프 치환 원칙은 클래스의 기반 자료형과 하위 자료형 사이에 성립하는 규칙입니다.
간단하게 말하면 "기반 자료형을 하위 자료형으로 변경해도, 코드는 문제없이 동작해야 한다"라는 의미입니다.

리스코프 치환 원칙을 위반하면 조건 분기 코드가 점점 많아져 **유지 보수하기 어려운 코드**가 됩니다.
이러한 코드는 인터페이스의 의미를 충분히 이해하지 못하고 사용하는 경우 자주 만들어집니다.

```java
interface HotelRates {
    Money fee(); // 기존에는 fee()만 존재했음
    Money busySeasonFee(); // 성수기 요금 추가
}
```

그리고 각 인터페이스 구현 클래스에서 `busySeasonFee`를 구현합니다. 그렇게 되면, 성수기 요금 자료형 판정 로직이 필요없어지게 됩니다.

```java
Money busySeasonFee = hotelRates.busySeasonFee();
```

## 인터페이스 사용 능력이 중급으로 올라가는 첫 걸음

**인터페이스를 잘 사용하는 것은 곧 설계 능력의 전환점**이라고 할 수 있습니디.
**조건 분기를 써야 하는 상황에는 일단 인터페이스 설계**를 떠올립시다.

## 플래그 매개변수

플래그 매개변수란 **메서드의 기능을 전환하는 boolean 자료형의 매개 변수**입니다.
플래그 매개변수를 받는 메서드는 어떤 일을 하는지 예측하기 굉장히 힘들어져 내부 로직 확인이 필수이며,
그로 인해 **가독성이 낮아지며 개발 생산성이 저하**됩니다.

플래그 매개변수를 받는 메서드는 기능별로 분리하는 것이 좋습니다.

```java
// AS-IS
damage(true, damageAmount); // 플래그 매개변수 사용

void damage (boolean damageFlag, int dagameAmount) {
    if (damageFlag == true) {
        // 물리 대미지 로직 (히트포인트 기반 대미지)
    }
    else {
        // 마법 대미지 로직 (매직포인트 기반 대미지)
    }
}

// TO-BE : 상황에 따라 hitPointDamage 또는 magicPointDamage 호출하여 사용
void hitPointDamage(final int damageAmount) {
    // 물리 대미지 로직
}

void magicPointDamage(fianl int damageAmout) {
    // 마법 대미지 로직
}
```

플래그 매개변수를 개선하기 위한 가장 좋은 방법은 전략 패턴을 사용하는 것입니다.
전략 패턴으로 설계하면, 대미지를 전환하거나 새로운 종류의 대미지 추가가 필요할 때 더 쉽게 대응할 수 있습니다.

```java
interface Damage {
    void execute(final int damageAmount);
}

class HitPointDamage implements Damage {
    // 생략
}

class MagicPointDamage implements Damage {
    // 생략
}
```

대미지를 적용하는 코드는 다음과 같이 구성합니다.

```java
enum DamageType {
    hitPoint,
   magicPoint
}

private final Map<DamageType, Damage> damages;

void applyDamage(final DamageType damageType, final int damageAmount) {
    final Damage damage = damages.get(damageType);
    damage.execute(damageAmount);
}
```

이렇게 하면, 실제로 어떤 종류의 대미지를 적용하려고 할 때는 다음과 같이 한 줄만 호출하면 됩니다.

```java
applyDamage(DamageType.magicPoint, damageAmount);
```