## 3장. 클래스 설계: 모든 것과 연결되는 설계 기반
### 클래스 단위로 잘 동작하도록 설계하기
* **잘 만들어진 클래스의 구성요소**
  1. 인스턴스 변수
  2. 인스턴스 변수에 잘못된 값이 할당되지 않게 막고, 정상적으로 조작하는 메서드

-> **굳이 다른 클래스를 사용해서 초기화와 유효성 검사**를 해야 하는 클래스는 그 자체로는 안전하게 사용할 수 없는 미성숙한 클래스이다. 응집도 높게 작성하는 것이 좋다. 

### 성숙한 클래스로 성장시키는 설계 기법

1. 생성자에서 유효성 검사
2. 계산 로직도 데이터가 가진 쪽에 구현
3. 인스턴스변수, 매개변수, 지역변수는 불변 변수로 정의
4. 변경하고 싶다면 새로운 인스턴스 생성
5. 엉뚱한 값을 전달하지 않도록 새로운 자료형 만들어 사용
6. 의미없는 메서드를 만들지 말고, 시스템 사양에 필요한 메서드만 정의


```java
class Money {
    final int amount;
    final Currency currency; 

    Money(final int amount, final Currency currency) { 
        //인스턴스 변수 초기화 강제, 유효성 확인 
        if (amount < 0) {
            throw new IllegalArgumentException("금액은 0 이상의 값을 지정해 주세요.");
        }

        if (currency == null) {
            throw new IllegalArgumentException("통화 단위를 지정해 주세요.");
        }

        this.amount = amount;
        this.currency = currency;
    }

    Money add(final Money other) { //잘못된 값 전달 방지를 위해 Money 전달, 부수효과 방지를 위한 fianl
        if (!currency.equals(other.currency)) {
            throw new IllegalArgumentException("통화 단위가 다릅니다.");
        }

        final int added = amount + other.amount;
        return new Money(added, currency); //값 변경을 원하면 새로운 인스턴스 생성
    }
}
```

### 프로그램 구조의 문제 해결에 도움을 주는 디자인 패턴
위의 설계는 (1) 완전 생성자, (2) 값 객체 라는 두 가지 디자인 패턴을 적용하였다.

1. 완전 생성자
    * **인스턴스 변수를 모두 초기화**해야만 객체를 생성할 수 있게, 매개변수를 가진 생성자를 만들게 함
    * 생성자 내부에서 가드를 사용해 **잘못된 값 방지**
    * 인스턴스 변수에 **final 로 불변으로 만들면, 생성 후에도 잘못된 상태로부터 방어**할 수 있음
2. 값 객체
    * 위 예시의 Money 클래스와 같이 값 객체로 생성해 **다른 값과 섞이는 상황을 원천적으로 차단**
