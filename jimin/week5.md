# 6장. 조건 분기: 미궁처럼 복잡한 분기 처리를 무너뜨리는 방법
## 조건 분기 중첩
### 문제점
* if 조건문의 블록이 어디까지인지 이해하기 어렵다.
* 중첩이 깊어질수록 이해하기 어려워지고, 사양 변경 또한 힘들어진다.

### 개선 방법
* early return
  * 조건을 반전시킨 후 곧바로 리턴

## switch 조건문 중복
### 문제점
* 같은 형태의 switch 조건문이 여러 개 사용되기 시작한다면, 새로운 타입을 추가할 때 누락될 가능성이 존재한다.
* 또 다른 switch 문이 계속해서 생겨날 수 있다.

### 개선 방법
* 조건 분기 모으기 
  * 한 클래스에 switch 조건문을 모은다.
* 인터페이스로 switch 조건문 해소 (전략패턴)
  * 인터페이스로 정의하고, `Map<타입, 구현체>`로 type에 대응하는 인스턴스를 추출하도록 한다.
    ```java
    //인터페이스 구현
    interface Magic {
        String name();
        int costMagicPoint();
        int attackPower();
    }

    class Fire implements Magic {...}
    ```

    ```java
    //필요한 구현체 추출
    final Map<MagicType, Magic> magics = new HashMap<>();
    void magicAttack(final MagicType magicType) {
        final Magic usingMagic = magics.get(magicType)
        
        showMagicName(usingMagic);
        consumeMagicPoint(usingMagic);
        magicDamage(usingMagic);
    }

    void showMagicName(final Magic magic) {
        System.out.println(magic.name());
    }
    ```

## 조건 분기 중복과 중첩

아래와 같이 a,b 판정 조건이 재사용되는 경우가 존재할 수 있는데, 이때 정책 패턴을 사용할 수 있다.
* a,b,c 조건에 해당하면 골드 회원
* a,b 조건에 해당하면 실버 회원

### 정책 패턴
* 정책 패턴: 조건을 부품처럼 만들고, 부품으로 만든 조건을 조합해서 사용하는 패턴 

```java
interface ExcellentCustomerRule {
    boolean ok(final PurchaseHistory history);
}

//규칙들 분리 
class PurchaseFrequencyRule implements ExcellentCustomerRule {
    public boolean ok(final PurchaseHistory history) {
        return 10 <= history.purchaseFrequencyPerMonth;
    }
}

class ReturnRateRule implements ExcellentCustomerRule {
    //...
}

//규칙 관리
class ExcellentCustomerPolicy {
    private final Set<ExcellentCustomerRule> rules;
    
    ExcellentCustomerPolicy() {
        rules = new HashSet();
    }

    void add(final ExcellentCustomerRule rule) {
        rules.add(rule);
    }

    boolean complyWithAll(final PurchaseHistory history) {
        // 규칙을 모두 만족하는 경우 true
        for (ExcellentCustomerRule each : rules) {
            if (!each.ok(history)) return false;
        }
        return true;
    }
}
```
* add 를 통해 조건에 해당하는 rule 만 추가하도록 하면 된다.

## 자료형 확인에 조건 분기 사용하지 않기
* `instanceof`와 같이 자료형 확인 후, 분기하는 로직을 사용하지 말고 인터페이스에 메서드를 추가하여 각 구현체 내에 정의하도록 하자

## 플래그 매개변수
플래그를 이용해 분기 처리한다면, 해당 메서드는 어떤 일을 하는지 예측하기 어렵다.

### 개선 방법
1. 메서드 분리
    * 플래그 매개변수를 받는 메서드는 기능별로 분리하는게 낫다.
2. 전환은 전략 패턴으로 구현
    * type을 넘겨서 `map`을 이용해 구현체를 꺼낸 후 해당 type 에 맞는 로직을 실행하도록 한다.

