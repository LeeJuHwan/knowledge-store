# 자동차 경주 미션 구현

## <mark style="color:blue;">"Good Game Well Played"</mark>

[Mission PR 보러가기](https://github.com/java-playground-hiking/java-racing-car/pull/6)

### <mark style="background-color:green;">Goal</mark>

**객체의 행동을 중심으로 상태를 설계 하는 방식을 적용한다**. 이는 상태를 먼저 설계 했을 때 캡슐화가 저해 될 수 있으며 행동을 상태에 맞추는 과정에서 객체간 협력이 아닌 스스로 모두 해결 하는 객체를 설계 할 수 있기 때문에 피해보려 한다.

**테스트 주도 개발 사이클을 적극 활용한다**. 프로덕션 코드를 작성 하기 전 테스트 코드를 먼저 작성 하며 요구사항을 이해 하는 데 집중 하고 테스트 하기 편리한 메서드 시그니처를 설계 하는 과정에 집중한다.

**일급 컬렉션을 활용한다**. 객체를 포장 하고 있는 사용자 정의 컬렉션을 사용 하며 객체에게 메시지를 던져 상태를 변경 하거나 조회 하는 객체지향적 사고 방식을 적극 활용한다.

**모든 원시 값을 포장한다**. 모든 원시 자료형에 대해 의미를 부여 하여 코드의 추상화 레벨을 동등한 선으로 설정 하고, 코드를 읽는 데 방해 되지 않도록 코드에서 의미를 설명한다.



### <mark style="background-color:green;">Goods</mark>

_<mark style="color:green;">goals 3 done</mark>_

#### "**객체를 설계 할 때 행동을 중심으로 상태를 설계 하는 방식 잘 적용하기**"

자동차 객체는 "4" 이상이 나오면 전진 해야한다 라는 요구사항에 따르면 자동차가 앞으로 나아갈 행동이 필요하다. 이를 `moveFoward` 라고 만들 것이다.

그럼 자동차가 전진 해야 할 때 마다 위치가 변해야 하기 때문에 자동차 객체는 자연스레 `Name`과 `Position`를 갖게 된다.&#x20;

#### **"일급 컬렉션을 활용 하고, 모든 원시 값을 포장하기"**

자동차 객체, 여러 자동차를 관리 하는 객체, 게임을 승리 한 자동차를 관리 하는 객체, 스코어를 보여주는 객체 등 객체간 협력을 이루기 위해 많은 원시 값을 포장 하고 있는 객체들이 탄생했다.&#x20;

이 객체들의 존재 덕분에 어플리케이션에서 원시 자료형이 갑자기 튀어나오지 않는다. 덕분에 코드를 읽는 데 있어 많은 도움을 주고 있다. 예를 들면 `StringBuilder`에 의해 생성 된 자동차 위치 값 표기 방식인 `String Type`은 `Car`객체에서 `carPosition`을 기준으로 변환 하여 반환한다.&#x20;



### <mark style="background-color:red;">Weaknesses</mark>

_<mark style="color:red;">goals 1 fail</mark>_

#### **"테스트 주도 개발 사이클 적극 활용하기"**

이하 TDD 방법론을 적용 하는 미션을 앞서 몇 번 진행 했었고 큰 무리 없었다. 매번 초반 설계를 잘 했을 때 순조롭게 흘러갔었으나, 해당 미션은 순조롭지 못했다.&#x20;

**TDD 장애물**

요구사항을 살펴 보면 "자동차는 0과 9사이 중 4이상이 나오면 앞으로 전진한다." 처음은 자동차 객체를 구현 하기 전 테스트 코드를 순조롭게 작성 할 수 있었다.

시간이 흘러 해당 미션에 대한 핵심 로직인 자동차 경주를 구현 할 때 장애물을 만났다.

자동차 경주를 1번 수행 할 때 생성 되어 있는 모든 자동차가 각자 다른 난수를 생성 하여 해당 값이 4 이상일 때 전진 시켜야한다.

특정 값이 모든 자동차에게 영향을 주는 것이 아니었기 때문에 외부에서 값을 주입 할 수 없었다.&#x20;

개인적으로 이 문제를 3일 정도 고민 했었고 외부 코드를 참고 하고 싶지 않았다. 어차피 해당 미션 끝엔 피드백 영상이 있기 때문에 구현 하며 고통이 따를 것이라 짐작 했기 때문이다.



### <mark style="background-color:blue;">Progresses</mark>

**원시 값 포장과 객체를 설계 하는 방식에 대한 수준을 더 높여야한다**. 아직은 작은 콘솔 토이 프로젝트에서 추출된 도메인을 기반으로 하고 있지만 더 큰 프로젝트에서는 수 많은 도메인과 여러 비즈니스 요구사항이 존재 할 것이기 때문에 기반을 다져야한다.

**테스트를 먼저 할 수 없다고 판단 되면 일단 프로덕션 코드를 작성하고 단위 테스트를 구현한다**. 이번 미션을 진행 하면서 테스트를 작성하지 못 해 미션을 진행하지 못하여 시간을 많이 보냈다. 만약 회사에서 개발 마감일정을 이러한 이유로 못 맞췄다고 한다면 과연 개발자로서 옳은 판단이었을까 생각 하게 되었다.

**메서드를 더 작은 기능으로 분리한다**. 우승자를 가리는 메서드에서 현재 가장 멀리 간 자동차의 위치 값을 가져오고, 그 값과 같은 자동차가 더 있는지 판단하는 메서드에서 두 가지 행동이 일어났다. 이 부분을 메서드 2개로 나누어 호출 했었어야 했는데 그 부분이 통합 되었던게 너무 아쉬웠기 때문에 메서드를 분리 하는 연습을 더 의도적으로 해야겠다.

***



### Feedbacks

{% tabs %}
{% tab title="As-Is" %}
{% code lineNumbers="true" fullWidth="true" %}
```java
public class Winners {

    private final List<String> winners;

    public static Winners from(List<String> winners) {
        return new Winners(winners);
    }
}

public class ConsoleOutputHandler {
    ...

    private String getWinnerNames(Winners winners) {
        return String.join(", ", winners.getWinners()) + GameMessage.finalWinnerSuffix;
    }
}

// 문제의 호출 코드
return Winners.from(winners.stream().map(Car::getCarName).collect(Collectors.toList()));
```
{% endcode %}
{% endtab %}

{% tab title="To-Be" %}
{% code lineNumbers="true" fullWidth="true" %}
```java
public class Winners {

    private final String winners;

    public static Winners from(Cars winnerCars) {
        return new Winners(
                winnerCars.getCars().stream()
                        .map(Car::getCarName)
                        .collect(Collectors.joining(", "))
        );
    }
    
    public String getWinners() {
        return winners;
    }
}

public class ConsoleOutputHandler {
    ...

    public void printRaceResult(Winners winners) {
        System.out.println(winners.getWinners() + GameMessage.finalWinnerSuffix);
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}



#### 피드백을 통해 배운 점

**Stream API 내에서 문자열 Join을 지원 한다는 것을 배웠다**. 기존 AS-IS 코드에서 `List<String>` 자료형으로 게임의 승자를 알려주는 객체에게 전달 했었다. 그리고 그 값을 `ConsoleOutputHandler`에서 출력하게 했었는데 이 때 해당 객체의 의도와 달리 출력만 하지 않고 서비스 로직이 포함 되었다는 것이 피드백 대상이었다.

**Stream API 내에서** `max()` **체이닝을 하면 Optional Value이기 때문에** `OrElseThrow`**를 사용 했지만** OrElse(0) **으로 처리하는 더 간단한 방법을 배웠다**. 기존 코드는 불필요한 Exception이 생겨났다. 하지만 개선 할 수 있는 부분에서는 숫자형으로 관리 하기 때문에 별도의 작업이 필요 없었다.

