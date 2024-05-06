# 6장 조건 분기: 미궁처럼 복잡한 분기 처리를 무너뜨리는 방법

- 조건 분기는 조건에 따라 처리 내용을 전환하는 데 사용되는 프로그래밍 기본 제어 구조이다.
- 조건 분기를 사용하면 복잡한 판단을 빠르고 정확하게 할 수 있다.

## 6.1 조건 분기가 중첩되어 낮아지는 가독성

- 중첩 if문을 여러개 사용하면 코드의 가독성이 크게 떨어지는 문제가 있다.
- 가독성이 나쁜 코드는 팀 전체의 개발 생산성을 저하 시키며, 코드가 복잡하고 길어지면서 코드을 변경하는데 큰 어려움이 있다.
- 중첩 문을 제거하는 방법 중 조기 리턴이 있다.
- 조기 리턴은 조건을 만족하지 않는 경우 곧바로 리턴하는 방법이다.

    ``` java
    if (member.hitPoint <= 0) return;
    if (member.canAct().not()) return;
    if (member.magicPoint < magic.constMagicPoint) return;

    // 비즈니스 로직 수행
    ...
    ```

- 조기 리턴은 가독성을 좋게해서 로직을 빠르게 이해할 수 있게 해준다.
- else 구문도 가독성을 나쁘게 만드는 원인 중 하나이다.
- 간단한 코드임에도 불구하고 중첩된 if 조건문 내부에 else 구문이 섞이면 가독성이 현저히 낮아져 코드를 이해하기 힘들다. 런 경우 조기 리턴을  사용하여 if문 내부에 return을 배치한다.

## 6.2 switch 조건문 중복

- 값 종류에 따라 다른 처리를 하고 싶을 때 우리는 switch 조건문을 많이 사용한다.
- switch 조건문을 사용시에는 요구 사항 변경 시 case 구문에 조건을 추가하는 과정에서 누락이 발생하여 만족되는 요구사항이 누락 될 수 있다.
- switch문의 경우 조건문의 중복이 많아지면 실수가 발생할 수 밖에 없고 case가 추가 됨에 따라 버그가 발생할 가능성이 있다.
- switch 조건문의 중복을 해소하기 위해선 **단일 책임 선택의 원칙**을 생각하여, switch 조건문을 한번에 작성하는 것도 방법이지만, 변경이 필요한 부분이 많아 지는 경우 switch문에 대한 조건 로직이 점점 많아 지면서 클래스가 거대해지게 된다.
- 이런 경우 interface을 사용하여 처리하는 방법도 있다.

> interface를 switch 조건문 중복에 응용하기
>
> - 종류별로 다르게 처리해야 하는 기능을 인터페이스의 메서드로 정의
> - 인터페이스의 이름을 어떤 부류에 속하는가? 기준으로 정하기
> - 종류별로 클래스 만들기
> - 각각의, 클래스에 인터페이스 구현하기
> - switch 조건문이 아니라 Map으로 변경하기
> - 메서드를 구현하지 않으면 오류로 인식하게 만들기
> - 값 객체화하기

## 6.3 조건 분기 중복과 중첩

- interface는 switch 조건문의 중복을 제거할 수 있을 뿐만 아니라, 다중 중첩된 복잡한 분기를제거하는 데 활용할 수 있다.
- 정책 패턴 (policy pattern): 조건을 부품처러 만들고, 부품으로 만든 조건을 조합해서 사용하는 패턴
  - interface 선언

    ``` java
    // interface 선언
    interface ExcellentCustomerRule {
        boolean ok(final PurchaseHistory history)
    }
    ```

  - 정책에 대한 검증 클래스 선언

    ```java
    class GoldCustomerPurchaseAmountRule implements ExcellentCustomerRule {
        public boolean ok(final PurchaseHistory history) {
            return 100_000 <= history.totalAmoint;
        }
    }
    ```

    ```java
    class PurchaseFrequencyRule implements ExcellentCustomerRule {
        public boolean ok(final PurchaseHistory history) {
            return 10 <= history.purchaseFrequencyPerMonth;
        }
    }
    ```

    ```java
    class ReturnRateRule implements ExcellentCustomerRule {
        public boolean ok(final PurchaseHistory history) {
            return history.returnRate <= 0.001;
        }
    }
    ```

- 정책 클래스 선언

    ``` java
    class ExcellentCustomerPolicy {
        private final Ser<ExcellentCustomerRule> rules;


        ExcellentCustomerPolicy() {
            rules = new HashSet();
        }

        /**
         * 규칙추가
         */
        void add(final ExcellentCustomerRule rule) {
            rules.add(rule);
        }
        
        boolean complyWithAll(final PurchaseHistory history) {
            for (ExcellentCustomerRule each : rules) {
                if(each.ok(history).not()) return false;
            }
            return true;
        } 
    }
    ```

## 6.4 자료형 확인에 조건 분기 사용하지 않기

- java에서 `instanceof`는 자료형을 판정하는 연산자이다.
- 자료형을 판정하는 연산자를 사용하여 데이터를 확인하는 경우 조건 분기를 사용하면 요구사항이 추가 되는 경우마다 조건 분기가 계속 추가된다. -> 리스코프 치환 원칙 위반
- 그러기 때문에 `instanceof`처럼 차료형을 판정하는 연산자를 사용할 때는 코드의 구조를 변경하여 사용하도록 해애한다.

## 6.5 인터페이스 사용 능력이 중급으로 올라가는 첫걸음

- 여기는 그냥 인터페이스를 잘 사용하자라는거군...

## 6.6 플래그 매개변수

- booelan 타입의 매개 변수를 플래그 매개변수라고 하며, 플래그 매개변수는 어떤 일을 하는지 예측하기 어렵다는 단점이 있다.
- 플래그 매개변수를 받는 메서드는 기능별로 분리하는 것이 좋다.
