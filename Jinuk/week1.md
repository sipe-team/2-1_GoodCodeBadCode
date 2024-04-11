# 1. 잘못된 구조의 문제 깨닫기
- 의미를 알 수 없는 이름
    - 기술 중심 명명
    - 일련번호 명명
    - 위의 명명법들은 코드에서 의도를 읽어내기가 힘듦
    - 버그 발생 가능성이 올라감
        - 역할과 기능에 대한 일람표를 만들어서 해결? -> 유지보수 포인트 증가
    - 의도와 목적을 드러내는 이름을 사용하자

- 이해하기 어렵게 만드는 조건 분기
    - 조건문의 중첩 -> 어디까지가 조건문 처리 블록인지 확인하기 어려움
    - 조건이 복잡해질수록 코드를 읽고 이해하기 어려움

- 수많은 악마를 만들어 내는 데이터 클래스
    - 코틀린이나 파이썬의 data class랑은 다르다!
    - 말 그대로 데이터밖에 없는 클래스를 의미
    ```java
    public class ContractAmount {
        public int amountIncludingTax;
        public BigDecimal salesTaxRate;
    }
    ```
    - 데이터 클래스에 관련된 로직이 다른 곳에 있는 경우, 사양 변경 시 버그 발생, 코드 중복, 수정 누락, 가독성 저하, 가비지 객체, 유효성 문제 등을 유발할 수 있음

- 그러면... 이런 악마들을 퇴치하려면?
    - 나쁜 구조의 폐해 인지 필요
    - 객체지향의 기본인 클래스를 적절히 설계해야 함

# 2. 설계 첫걸음
- 의도를 분명히 전달할 수 있는 이름 설계
    - 데미지를 d, 플레이어의 공격력을 p1 이런식으로 개발한다면?
        ```java
        int d = 0;
        d = p1 + p2;
        d = d - ((d1 + d2) / 2);
        if (d < 0) {
            d = 0;
        }
        ```
        - 구현은 빠르지만, 이해하는데 시간 너무 오래 걸릴수밖에...
        - 결국 전체 개발 시간은 더 많이 늘어나게 됨
    - 의도를 알 수 있는 이름을 사용 필요
        - 변수 이름을 쉽게 붙이는 것도 괜찮다
        ```java
        int damageAmount = 0;
        damageAmount = playerArmPower + playerWeaponPower;
        damageAmount = damageAmount - ((enemyBodyDefence + enemyArmorDefence) / 2);
        if (damageAmount < 0) {
            damageAmount = 0;
        }
        ```

- 목적별로 변수를 따로 만들어 사용하기
    - 위 코드의 경우 이해하기는 확실히 쉬워졌지만, 문제가 있다!
        - `damageAmount` 변수에 값이 여러번 할당되고 있음
        - 재할당은 변수의 용도를 혼란스럽게 할 수 있음
        - 개념이 다른 값들은 다른 변수를 사용하자.
        ```java
        int totalPlayerAttackPower = playerArmPower + playerWeaponPower;
        int totalEnemyDefence = enemyBodyDefence + enemyArmorDefence;
        
        int damageAmount = totalPlayerAttackPower - (totalEnemyDefence) / 2
        
        if (damageAmount < 0) {
            damageAmount = 0;
        }
        ```

- 단순 나열이 아니라, 의미 있는 것들을 모아 메소드로 만들기
    - 계산 로직이 복잡해지면, 개발 과정에서 착각이 일어날 수 있음
    - 시작과 종료를 명확하게 만들어 주는 방법 -> 의미 있는 로직을 모아서 메소드로 만들기
        ```java
        
        int getPlayerAttackPower(int playerArmPower, int playerWeaponPower) {
            return playerArmPower + playerWeaponPower;
        }

        int getEnemyDefence(int enemyBodyDefence, int enemyArmorDefence) {
            return enemyBodyDefence + enemyArmorDefence;
        }

        int getDamage(int totalPlayerAttackPower, int totalenemyDefence) {
            int damageAmount = totalPlayerAttackPower - (totalEnemyDefence / 2);

            if (damageAmount < 0) {
                return 0;
            }

            return damageAmount;
        }
        
        
        int playerAttackPower = getPlayerAttackPower(playerBodyPower, playerWeaponPower);
        int enemyDefence = getEnemyDefence(enemyBodyDefence, enemyArmorDefence);
        
        int damageAmount = getDamage(playerAttackPower, enemyDefence);
        ```

    - 코드의 양은 많아지지만, 읽고 이해하기는 쉬워짐

- 관련된 데이터와 로직을 클래스로 모으기
    - 변수와 변수를 조작하는 로직을 산발해 두지 않고, 한 클래스로 모은다
        - validation, method, 인스턴스 변수 등을 클래스에 모아 둔다
    
    ```java
        Class Player {
            private static final int MAX = 999;
            private static final int MIN = 0;

            private int weaponPower = 0;
            private int armorDefence = 0;

            private final int armPower;
            private final int bodyDefence;

            Player(final int armPower, final int bodyDefence) {
                if (armPower < MIN || bodyDefence < MIN) {
                    throw new IllegalArgumentException();
                }

                if (armPower > MAX || bodyDefence > MAX) {
                    throw new IllegalArgumentException();
                }
                
                this.armPower = armPower;
                this.bodyDefence = bodyDefence;
            }

            public int getAttackPower() {
                return armPower + weaponPower;
            }
         
            public int getDefence() {
                return bodyDefence + armorDefence;
            }

            public static int getDamage(Player user, Player enemy) {
                int damageAmount = user.getAttackPower - (enemy.getDefence / 2);

                if (damageAmount < 0) {
                    return 0;
                }

                return damageAmount;
            }

        }
        
        Player user = new Player(20, 20);
        Player enemy = new Player(10, 10);
        
        int userAttackPower = user.getAttackPower();
        int enemyDefence = enemy.getDefence();
        int damageAmount = Player.getDamage(user, enemy);
    ```
