# Readable code

<details>

<summary>Properties</summary>

:pencil:2024.09.10

:page_facing_up: [읽기 좋은 코드를 작성하는 사고법](https://www.inflearn.com/course/readable-code-%EC%9D%BD%EA%B8%B0%EC%A2%8B%EC%9D%80%EC%BD%94%EB%93%9C-%EC%9E%91%EC%84%B1%EC%82%AC%EA%B3%A0%EB%B2%95/dashboard)

:paperclip: 상속과 조합

</details>

## 객체 지향 이론을 코드로 적용하기


### 상속과 조합


{% hint style="info" %}

<mark style="color:orange;background-color:purple;">상속보다 조합을 사용하자</mark>

- 상속은 시멘트 처럼 굳어지는 구조로 수정이 어렵다. -> 부모 클래스와 자식 클래스의 결합도가 매우높음

- 조합과 인터페이스를 활용하는 것이 유연한 구조 -> **상속을 통한 코드의 중복 제거가 주는 이점 보다, 중복이 생기더라도 유연한 구조 설계가 주는 이점이 더 큼**

{% endhint %}


상속 구조는 과거에 코드 반복이 하드웨어 성능에 영향을 끼친다고 생각 했던 시절에 부모 클래스의 메서드를 자식 클래스에서 반복 없이 사용 할 수 있을 때 효율적이었다.

하지만 현대 하드웨어 성능이 과연 코드 몇 줄 줄였을 때 유의미한 성능 향상을 기대할 수 있을까? 단연 있다고 한들 굉장히 견고한 설계 구조가 아닌 이상 쉽게 상속 구조를 사용하기 어렵다.

유지보수 하는 개발자 입장에서 부모 클래스의 메서드 하나를 수정하기 위해 굉장히 많은 작업을 반복해야 하기 때문에 유연한 조합 설계 구조로 왠만한 문제는 풀어나갈 수 있다.

**Example use case**:

{% code title="상속을 이용하는 경우" overflow="wrap" lineNumbers="true" %}

```java
public class EmptyCell extends Cell {

    private static final String EMPTY_SIGN = "■";

    @Override
    public boolean isLandMine() {
        return false;
    }

    @Override
    public boolean hasLandMineCount() {
        return false;
    }

    @Override
    public String getSign() {
        if (isOpened) {  // 부모 메서드
            return EMPTY_SIGN;
        }

        if (isFlagged) {  // 부모 메서드
            return FLAG_SIGN;
        }

        return UNCHECKED_SIGN;
    }
}
```

{% endcode %}

위 처럼 상속을 이용하는 경우 `isOpened`와 `isFlagged` 가 수정이 발생 했다고 가정 했을 때 과연 `getSign` 메서드는 의도한 조건에 맞게 실행 되고 있는지 테스트가 필요하다.

또한, 자식 클래스가 부모 클래스의 캡슐화 된 데이터를 모든 정보를 다 알고있다. 이 경우는 캡슐화도 깨진 경우이기 때문에 조합을 이용해서 캡슐화를 지키고 유지보수 측면에서 유연한 설계를 보자면 아래와 같다.


{% code title="조합을 이용하는 경우" overflow="wrap" lineNumbers="true" %}

```java
public class EmptyCell implements Cell {

    private static final String EMPTY_SIGN = "■";

    private final CellState cellState = CellState.initialize();

    @Override
    public boolean isLandMine() {
        return false;
    }

    @Override
    public boolean hasLandMineCount() {
        return false;
    }

    @Override
    public void flag() {
        cellState.flag();
    }

    @Override
    public void open() {
        cellState.open();
    }

    @Override
    public boolean isOpened() {
        return cellState.isOpened();
    }

    @Override
    public boolean isChecked() {
        return cellState.isChecked();
    }

    @Override
    public String getSign() {
        if (cellState.isOpened()) {
            return EMPTY_SIGN;
        }

        if (cellState.isFlagged()) {
            return FLAG_SIGN;
        }

        return UNCHECKED_SIGN;
    }
}
```

{% endcode %}

위 코드는 조합과 인터페이스를 활용 한 경우이며 물론 상속 보다는 반복되는 코드 구조가 발생 하는 것은 사실이다. 하지만, `cell`과 관련 된 데이터 내부는 캡슐화 된 다른 객체로 대체 되었고 외부 객체와 협력하고 있다. 또한 인터페이스를 물려 받아 구현한 메서드들은 해당 객체의 고유한 행동을 나타낼 수 있기 때문에 수정 되더라도 메인 로직의 변함은 없다.

### Value Object(VO)

{% hint style="info" %}

<mark style="color:orange;background-color:purple;">Value Object</mark>

- 도메인의 어떤 개념을 추상화하여 표현한 값 객체

- 값으로 취급하기 위해서, 불변성, 동등성, 유효성 검증 등을 보장해야 한다.
  - 불변성: final 필드, setter 금지
  - 동등성: 서로 다른 인스턴스여도(Object ID가 달라도), 내부의 값이 같으면 같은 값 객체로 취급한다. `equals()` & `hashCode()` 재정의 필요

- 유효성 검증: 객체가 생성되는 시점에 값에 대한 유효성을 보장하기 

{% endhint %}

**Example use case**:

{% code title="Money 예시" overflow="wrap" lineNumbers="true" %}

```java
public class Money {
  private final long amount;
  
  public Money(long amount){
    if (amount < 0) {
      throw new IllegalArgumentException("금액은 0원 이상이어야 합니다.);
    }
    this.amount = amount;
  }
  
  // equals() && hashCode() 재정의
}

/*

Money money1 = new Money(1_000L);
Money money2 = new Money(1_000:);

assertThat(money1 == money2).isFalse();
assertThat(money1.equals(money2)).isTrue();
*/

```


#### VO vs Entity


{% tabs %}

{% tab title="Entity" %}

```java
class UserAccount {
  private String userId; // 식별자
  private String 이름;
  private String 생년월일;
  private Address 집주소;
}
```

{% endtab %}

{% tab title="VO" %}

```java
class Address {
  private String 시도;
  private String 시군구;
  private String 도로명;
  private String 건물번호;
}
```
 
{% endtab %}

{% endtabs %}

- Entity는 **식별자**가 존재한다. 식별자가 아닌 필드의 값이 달라도 식별자가 같으면 동등한 객체로 취급한다.
  - `equals()` & `hashCode()`도 식별자 필드만 가지고 재정의 할 수 있다.
  - 식별자가 같은데 식별자가 아닌 필드의 값이 서로 다른 두 인스턴스가 있다면 같은 Entity가 시간이 지남에 따라 값이 수정된 것으로 이해 할 수 있다.

- VO는 식별자 없이 **내부의 모든 값이 다 같아야 동등한 객체로 취급한다**
  - 개념적으로 전체 필드가 다 같이 식별자 역할을 한다고 생각해도 된다.

