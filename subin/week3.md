# 4장. 불변 활용하기 : 안정적으로 동작하게 만들기

## 재할당

재할당은 **변수에 값을 다시 할당하는 것**으로 파괴적 할당이라고도 합니다.
재할당은 **변수의 의미를 바꿔 추측하기 어렵게 만듭니다.**

```java
int damage() {
    int tmp = member.power() + member.weaponAttack();
    tmp = (int) (tmp * (1f + member.speed() / 100f));
    tmp = tmp - (int)(enemy.defence / 2);
    tmp = Math.max(0, tmp);
    
    return tmp;
}
```

위와 같이 재할당을 계속하면, 중간에 의미가 바뀌기 때문에 읽는 사람이 헷갈리게 됩니다.
헷갈리면 버그를 만들어 낼 가능성이 높습니다.
재할당을 피하기 위해서는 **변수 하나를 재활용하지 않고, 계속해서 새로운 변수를 만들어 사용**하면 됩니다.

- 불변 변수로 만들어서 재할당 막기
- 매개변수도 불변으로 만들기

## 가변으로 인해 발생하는 의도하지 않은 영향

인스턴스가 가변이면 다른 부분에 의도하지 않은 영향을 주기 쉽습니다.
의도하지 않게 영향을 끼치는 경우 두 가지 및 문제를 개선하는 설계 방법에 대해 알아보겠습니다.

### 사례 1 : 가변 인스턴스 재사용하기

다음 코드는 무기의 공격력을 나타내는 AttackPower 클래스입니다.
공격력 값을 저장하는 인스턴스 변수 `value`는 final 수식자가 따로 붙어있지 않으므로 가변입니다.

```java
class AttackPower {
    static final int MIN = 0;
    int value; // 가변
    
    AttackPower(int value) {
        if (value < MIN) {
            throw new IllegalArgumentException();
        }
        
        this.value = value;
    }
}
```

다음은 무기를 나타내는 Weapon 클래스이며, AttackPower를 인스턴스 변수로 갖는 구조입니다.

```java
class Weapon {
    final AttackPower attackPower;
    
    Weapon(AttackPower attackPower) {
        this.attackPower = attackPower;
    }
}
```

다음과 같이 `AttackPower` 인스턴스를 재사용하는 코드를 작성하도록 코드를 구성하였습니다.

```java
AttackPower attackPower = new AttackPower(20);

Weapon weaponA = new Weapon(attackPower);
Weapon weaponB = new Weapon(attackPower);

weaponA.attackPower.value = 25;

System.out.println("Weapon A attack power : " + weaponA.attackPower.value);
System.out.println("Weapon B attack power : " + weaponB.attackPower.value);
```

분명 weaponA에 대한 공격력을 변경하였는데, 실행해보면 weaponB에 대한 공격력도 변경된 것을 확인할 수 있습니다.

```text
Weapon A attack power : 25
Weapon B attack power : 25
```

이러한 상황을 예방하려면, AttackPower 인스턴스를 개별적으로 생상하고 재사용하지 않는 로직으로 변경하면 됩니다.

```java
AttackPower attackPowerA = new AttackPower(20);
AttackPower attackPowerB = new AttackPower(20);

Weapon weaponA = new Weapon(attackPowerA);
Weapon weaponB = new Weapon(attackPowerB);

weaponA.attackPower.value = 25;

System.out.println("Weapon A attack power : " + weaponA.attackPower.value);
System.out.println("Weapon B attack power : " + weaponB.attackPower.value);
```

이렇게 하면 weaponA 공격력 변경이 weaponB에 영향을 주지 않습니다.

```text
Weapon A attack power : 25
Weapon B attack power : 20
```

### 사례 2 : 함수로 가변 인스턴스 조작하기

AttackPower 클래스에 공격력을 변화시키는 reinforce 메서드와 disable 메서드를 추가했습니다.

```java
class AttackPower {
    static final int MIN = 0;
    int value;
    
    AttackPower(int value) {
        if (value < MIN) {
            throw new IllegalArgumentException();
        }
        
        this.value = value;
    }
    
    void reinforce(int increment) {
        value += increment;
    }
    
    void disable() {
        value = MIN;
    }
}
```

한 스레드에서는 AttackPower.reinforce 메서드를 호출하고, 다른 스레드에서는 AttackPower.disable 메서드를 호출하면 예상과 다른 공격력이 설정될 수 있습니다.
이러한 경우를 **부수 효과** 라고 합니다.

### 부수 효과

부수 효과를 주요 작용과 함께 정리하면 다음과 같습니다.

- 주요 작용 : 함수(메서드)가 매개변수를 전달받고, 값을 리턴하는 것
- **부수 효과 : 주요 작용 이외의 상태 변경을 일으키는 것**

여기서 상태 변경이란 **함수 밖에 있는 상태를 변경하는 것**이며, 예를 들면 다음과 같습니다.

- 인스턴스 변수 변경
- 전역 변수 변경
- 매개변수 변경
- 파일 읽고 쓰기 같은 I/O 조작

부수 효과가 있는 함수는 영향 범위를 예측하기 힘듭니다.
따라서 함수는 다음 항목을 만족하도록 설계하는 것이 좋습니다.

- 데이터(상태)는 매개변수로 받습니다.
- 상태를 변경하지 않습니다.
- 값은 함수의 리턴 값으로 돌려줍니다.

이를 고려하여 AttackPower 클래스에 부수 효과가 발생하지 않도록 수정해보겠습니다.

```java
class AttackPower {
    static final int MIN = 0;
    final int value; // 불변으로 변경
    
    AttackPower(final int value) {
        if (value < MIN) {
            throw new IllegalArgumentException();
        }
        
        this.value = value;
    }
    
    AttackPower reinforce(final AttackPower increment) {
        return new AttackPower(this.value + increment.value);
    }
    
    AttackPower disable() {
        return new AttackPower(MIN);
    }
}
```

## 불변과 가변은 어떻게 다루어야 할까

변수를 **불변**으로 만들면 다음과 같은 **장점**이 있습니다.

- 변수의 의미가 변하지 않으므로, 혼란을 줄일 수 있습니다.
- 동작이 안정적이게 되므로, 결과를 예측하기 쉽습니다.
- 코드의 영향 범위가 한정적이므로, 유지 보수가 편리해집니다.

그러나 다음과 같은 경우에는 **가변이 필요하거나 더 좋을 수도** 있습니다.

- 성능이 중요한 경우
  - 대량의 데이터를 빠르게 처리해야 하는 경우
  - 이미지를 처리하는 경우
  - 리소스에 제약이 큰 임베디드 소프트웨어를 다루는 경우
- 스코프가 국소적인 경우

또한 불변을 활용해서 아무리 신중하게 설계하더라도, 코드 외부와의 데이터 교환은 주의해야 합니다.
코드 내부에서는 외부 동작을 제어할 수 없기 때문입니다.


