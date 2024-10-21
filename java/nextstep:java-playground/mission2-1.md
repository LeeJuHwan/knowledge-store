# Mission1. Baseball - Game Implementation

<details>

<summary>Properties</summary>

:pencil:2024.08.23

:computer: [source\_code](https://github.com/LeeJuHwan/java-baseball/tree/mission2-jhlee/src/main/java)

</details>

## 숫자야구게임 구현

### 요구사항 이해하기

> 용어 정리

* 플레이어: 나
* 상대방: 컴퓨터
* 같은 위치의 숫자: 713 중 1번째 요소는 "7", 사용자가 713을 입력 했다면 1번째 요소가 같음.

1. 컴퓨터는 1 부터 9까지 랜덤한 숫자로 3자리를 구성 해야한다.
2. 플레이어는 컴퓨터가 생성한 3자리 난수를 맞춰야한다.
3. 플레이어가 정답을 입력 할 때 마다 결과로 힌트가 제공 된다.
   * 플레이어가 입력한 숫자와 컴퓨터의 임의 숫자 중 같은 위치의 값이 같다면 "스트라이크"
   * 플레이어가 입력한 숫자가 같은 위치에 없지만 컴퓨터의 숫자 중 속해있다면 "볼"
   * 플레이어가 입력한 숫자가 아무 것도 포함되지 않는다면 "낫싱"
   * 예시
     * 컴퓨터 정답: 713
     * 플레이어 입력: 123 -> 결과: 1볼 1스트라이크
     * 플레이어 입력: 671 -> 결과: 2볼
     * 플레이어 입력: 456 -> 결과: 낫싱
     * 플레이어 입력: 713 -> 결과: 3스트라이크
4. 게임이 종료 되면 다시 실행 할지, 완전히 종료 할지 사용자에게 입력을 받는다.

\
\


### 도메인 추출하기

요구사항에 따른 도메인을 먼저 분석 해서, 해당 도메인 객체가 행동 할 수 있는 메서드를 만드는게 먼저라고 생각했다. 위 요구사항을 보면 가장 거시적으로 있어야 할 도메인 객체는 `유저`, `컴퓨터`가 있고 유저와 컴퓨터 사이의 연결을 해줄 객체가 필요해보인다.

> 왜 연결을 해줄 객체가 필요할까?

프로그램이 빌드 한 시점 이후로 `컴퓨터` 객체는 난수를 생성 해서 본인만 알고있다. 그 후, 플레이어가 정답을 입력 하여 그 숫자를 맞추는 과정이 진행 되는데 이 때 정답인지 아닌지 판별은 어떻게 할 것인가? 다짜고짜 컴퓨터에게 "이거 정답이야?" 라고 물어본다면, 컴퓨터는 당연히 "맞아" 또는 "아니야"가 대답이 될 것이다.

> 그렇다면 내가 필요한건 정답 여부인가?

정답 여부도 필요한 것은 맞지만 위 요구사항은 점수를 기록 해서 플레이어가 접근 하기 쉽도록 힌트를 제공 하고 있다. 그렇다면 컴퓨터에게 점수도 물어볼 것인가?

이 도메인을 추출하는 데 모호한 이유는 흔히 `SRP` 라는 단일 책임의 원칙 때문일 것이다.

컴퓨터 객체가 해야 할 일은 무엇일까? 고민 해본다면 정답을 생성 하는 것 이외에 행동 하는 것이 없다. 이 프로그램의 주체는 플레이어이기 때문이다. 다시 질문으로 돌아와서, 내가 필요한건 정답 여부는 아니라는 것이다. 정답을 판별 해줄 객체가 필요한데 지금 프로그램은 "야구 게임" 이니까 `심판`이 등장하면 어떨까?

**`컴퓨터`, `유저`, `심판` 그리고 `점수`가 도메인으로 추출되다.**

심판은 어떤 책임을 갖고 있어야할까? 개인적으로 유저가 입력한 정답에 대해 판정을 내려줬으면 좋겠다. 다시 돌아와, 정답 여부만 결정 되는 것이 아닐까? 그렇다면 유저의 정답을 판정 할 때 점수를 측정하면 좋을 듯 하다. 심판이 점수를 주는건 현실에서도 납득이 가는 행동이고, 판정을 내리는 것도 납득이간다.

심판이라는 객체는 점수를 측정하고, 유저가 정답을 입력하면 판정을 내리는 객체가 되었다. 심판이 혼자 힘들게 점수도 입력하고 게임도 진행하고 하면 너무 많은 일을 준 것 아닐까? 내가 만약 게임 속 심판이라면 점수 올리러 뛰어 갔다가 다시 뛰어서 유저가 던진 공이 볼인지, 스트라이크 인지 말해주고, 아웃인지 아닌지도 말 해줘야된다고 생각 하니 아찔하다.

자주 언급 되었던 `점수`도 하나의 객체가 될 수 있을 것 같다. 점수는 앞으로 심판과 협력하는 객체로 심판이 볼이라고 하면 점수 객체는 볼 카운트를 올리고, 스트라이크 라고 하면 스트라이크 카운트를 올리면 서로가 바라보는 관심사는 다르지만 정중하게 요구할 수 있다.

### 객체를 존중하기

객체지향의 캡슐화를 이용하면 속성이 대부분 숨겨지게 된다. 그렇게 숨겨진 속성에 접근하기 위해서 공개 메서드인 게터, 세터가 많이 등장 하게 되는데 **게터와 세터는 매우 폭력적인 존재**라는 것을 배운 적 있다. 게터를 이용하는 경우 내가 만든 객체에게 정중하게 묻는 것이 아닌 강제로 탈취하는 개념이 폭력적이라는 내용을 담았다. 하지만 요구 사항에 따른 개발을 하다보면 게터를 사용 할 수 밖에 없다. 그럴 때만 사용 할 수 있도록 최소화 하는 것이 이번 미션을 해결 하면서 신경 써야 할 문제였다. 게터를 사용 하는 것은 private 속성을 public으로 사용하는 것과 다름 없기 때문에 캡슐화를 지키고자 한다면 게터를 최소화하자.

> 그럼에도 불구하고 `Getter` 를 사용 할 수 밖에 없었던 이유

{% code title="User.java" overflow="wrap" lineNumbers="true" %}
```java
public class User {

    private final static int NUMBER_LIMIT_LENGTH = INPUT_LIMIT_LENGTH;
    private final ArrayList<String> userInputNumbers;

    public ArrayList<String> getUserInputNumbers() {
        return userInputNumbers;
    }

    ...
}
```
{% endcode %}

지극히 개인적인 생각으로 유저 객체가 갖고 있는 숫자는 정답과 비교해야 하기 때문에 게터를 안 쓸 수가 없었다. 게터를 보호하고 싶다면 해당 객체에게 정중하게 물어봐야한다. 그렇다면 유저에게 정답을 묻는 것이 올바른 행동일까 라는 생각이 들었는데, 유저는 문제를 맞추는 사람인데 정답을 알고있다? 굉장히 모순적인 것 같았다. 그렇기 때문에 유저는 어쩔 수 없이 게터를 사용했다.

{% code title="Score.java" overflow="wrap" lineNumbers="true" %}
```java
public class Score {

    private int strikeCount;
    private int ballCount;

    private static final String BALL_MESSAGE = "볼";
    private static final String STRIKE_MESSAGE = "스트라이크";
    private static final String UN_HANDLE_MESSAGE = "낫싱";

    public String getScoreRecordResult() {
        if (isBallAndStrike()) {
            return ballCount + BALL_MESSAGE +  " " + strikeCount + STRIKE_MESSAGE;
        }

        if (isOnlyBall()) {
            return ballCount + BALL_MESSAGE;
        }

        if (isOnlyStrike()) {
            return strikeCount + STRIKE_MESSAGE;
        }

        return UN_HANDLE_MESSAGE;
    }

    ...
}
```
{% endcode %}

스코어 객체에서도 결과를 게터를 통해 가져오고 있다. 앞서 도메인을 추출하는 과정에서 `심판`이 `스코어` 객체를 통해 점수를 산출하고 결과를 받아오는데 이 때 필요한 게터 역할이었다. 만약 여기서도 게터를 보호해야 한다면 어떤 방향으로 흘러갔을까? 게터가 없었다면 결과를 출력하기 위해 `스코어` 객체의 공개 메서드를 통해 결과를 출력 해줬어야 했다. 스코어 객체는 점수를 기록하고 반환하면 되는 객체로 활용 되는데 출력을 담당하는 객체와 또 협력을 맺어야 할 필요가 있을까란 생각이 들어 출력을 빼고 스코어 객체의 속성 값을 가공하여 반환 하기로 변경 하였다.

> `Getter`를 사용하지 않고 외부와 소통한 경우

{% code title="Judgement.java" overflow="wrap" lineNumbers="true" %}
```java
public class Judgment {

    private final Score score = new Score();
    private final ArrayList<String> computerRandomNumbers;

    public Score judge(ArrayList<String> userInputNumbers) {
        newScore();

        for (int seq = 0; seq < userInputNumbers.size(); seq++) {
            scoreRecord(userInputNumbers, seq);
        }
        return score;
    }

    private void newScore() {
        score.clean();
    }

    ...
}
```
{% endcode %}

`심판`은 점수를 산정한 후 `스코어` 객체를 반환한다. 심판이 점수를 산정 했으니 `getScore`를 통해 점수를 반환 했더라면 `Getter`를 사용하게 되는 경우에 속하게 되는데, 위 메서드에서 내부 상태를 변경하고 그 객체 자체를 반환 하고있다. 이렇게 반환 된 객체는 객체의 메서드를 통해 결과를 받아올 수 있어 캡슐화를 지키고 있다고 생각이 들었다.

### 추상화 레벨을 동등하게 유지하기

인간은 추상적인 개념을 이해하는 데 굉장히 뛰어나다. 아무래도 "컴퓨터" 라는 단어를 이해하는 것과 "0과 1로 이루어진 바이너리 명령어를 처리하는 기계" 라는 것을 이해하는건 전자가 훨씬 쉽기 때문에 소스코드에서도 읽기 좋은 코드를 작성할 때 가장 중요시 되는게 추상이라는 개념이다.

그 중, 추상화 레벨을 동등하게 유지하여 "컴퓨터" 라고 작성하던 코드에서 갑자기 "명령어 처리 기계" 라고 하지 않는 습관을 들이기 위해 의도적인 연습을 하고있다.

지금까지 추출한 도메인 내부는 추상적인 메서드를 사용하기 위해 만들어 놓은 구현부이다. 지금 확인 해볼 위치는 `Application` 의 게임 실행 부분인데, 이 때 뜬금 없이 구현 코드가 등장하지는 않는지 메서드명은 이해하기 쉬운 내용인지를 고민 해보면 좋다.

{% code title="BaseballApplication.java" overflow="wrap" lineNumbers="true" %}
```java

public class BaseballApplication implements Application {

    private final InputHandler inputHandler = new InputHandler();
    private final OutputHandler outputHandler = new OutputHandler();


    public void startGame() {
        Judgment judgment = readyToGame();
        outputHandler.gameStartCommentPrint();

        boolean isGameRunning = true;
        while (isGameRunning) {
            run(judgment);

            isGameRunning = pauseForUserGameRunOptionSelect();
        }
    }

    private void run(Judgment judgment) {
        try {
            actionToGameStartByJudgement(judgment);

        } catch (AppException e) {
            outputHandler.printAppExceptionMessage(e);

            run(judgment);
        }
    }

    private void actionToGameStartByJudgement(Judgment judgment) {
        String userInputValue = userInput();
        User user = new User(userInputValue);
        ArrayList<String> getUserInputArrayStringNumbers = user.getUserInputNumbers();

        Score score = judgment.judge(getUserInputArrayStringNumbers);

        String scoreResultMessage = score.getScoreRecordResult();
        outputHandler.printMessage(scoreResultMessage);

        boolean isGameSet = score.isStrikeCountEqualToWinningStrikeCount();

        if (stillGameRunning(isGameSet)) {
            actionToGameStartByJudgement(judgment);
        }
    }

    private boolean pauseForUserGameRunOptionSelect() {
        try {
            String gameFlag = userSelectGameRestartOrStop();

            return isGameContinue(gameFlag);

        } catch (AppException e) {
            outputHandler.printAppExceptionMessage(e);
            return pauseForUserGameRunOptionSelect();
        }
    }
    ...
}
```
{% endcode %}

가장 대표적으로 확인 해볼 수 있는건 게임을 시작하는 로직인데 위 작성된 코드를 살펴보면서 어떤 내용을 의도적인 수련을 하고자 했던건지 알아보려고 한다. 일단 코드를 위에서 부터 아래로 읽으면서 내려오면 이러한 해석이 가능하다.

1. `startGame` 을 실행

* 심판은 게임 할 준비가 되었다고 알려준다.
* 게임을 시작한다는 내용이 출력된다.
* 게임이 시작되고 있다고 기록한다.
* 게임이 시작되고 있을 땐 `run` 이라는 메서드를 통해 게임을 실행 시킨다.
* 게임이 종료되면 잠시 멈춘 뒤 유저가 게임 실행 옵션에 대해 선택하고 이 내용을 기록한다.

2. `run` 을 실행

* `actionToGameStartByJudgement` 라는 메서드가 실행 되는데 직역 해보면 "심판이 게임 시작을 위한 행동을 게시했다." 라고 해석할 수 있다.
* 게임 실행 도중 문제가 생기면 문제를 투명하게 공개하고, 다시 시작한다.

3. `actionToGameStartByJudgement` 를 실행

* 유저가 숫자를 입력한다.
* 유저가 입력한 숫자를 심판에게 전달한다.
* 심판이 판정을 내린다.
* 점수판에 점수가 기록된다.
* 점수판에서 "3스트라이크"인지 확인한다.
* "3스트라이크"가 맞는지 아닌지 확인 한 후 게임을 재개하거나 종료한다.

4. `pauseForUserGameRunOptionSelect` 를 실행

* 유저가 게임을 재시작 할 것인지 종료 할 것인지 고른다.
* 게임을 고르다가 문제가 발생 하면 문제 이유를 알려주고, 다시 고른다.
* 게임을 계속 시작 할 것인지 알려준다.

이렇게 긴 내용을 풀어서 해석 할 수 있다. 추상의 개념을 통해 메서드를 쉽게 이해할 수 있었으며 다소 불편한 부분도 존재 했지만 이런식으로 이해하기 쉬워진다. 만약, 게임을 다시 시작 할 것인지 고르는 부분에서 `userInput.equals(1)` 이라고 한다면 1이 무엇을 의미하고 어떤 단계인지 알기 어렵다. **이렇게 해석하기 어려운 코드는 구현부에 위치하고 해석하기 쉬운 이름들을 지어주는 행위를 추상화 레벨을 맞춰준다** 라고 표현 할 수 있다.

## 테스트 하기

### 단위 테스트와 통합 테스트 간략하게 알아보기

단위 테스트는 작은 기능이 빠르게 동작하기 위해 테스트 하는 것을 목적으로 하며 비즈니스 로직의 핵심 구현부를 의존성과 상관 없이 테스트 하는 환경을 만든다.

단위 테스트는 통합 테스트 보다 빠른 실행 시간을 갖고 있기 때문에 **자주 실행**을 할 수 있다는 것이 큰 장점이다.

통합 테스트는 의존성과 함께 넓은 커버리지를 담당한 테스트를 목적으로 하며 이 때 단위 테스트로 구현 할 수 없던 Private Method와 Branch Coverage 등 Reflection를 사용 하지 않고 직접적인 테스트가 어려운 부분을 커버하여 테스트한다.

통합 테스트는 실행 속도는 느리지만 넓은 커버리지를 맡고 있기 때문에 회귀 케이스를 예방 할 수 있도록 도움을 얻기 효과적인 것이 큰 장점이다.

### 단위 테스트를 구현하기

게임 어플리케이션의 실행 부분은 상태 값을 보존하는 무한 루프 코드와 사용자 입력을 기다리는 코드가 존재 하기 때문에 통합 테스트는 적합하지 않다고 생각이 들었다. 그렇다면 의존성 없이 메서드를 테스트 할 수 있는 환경을 만드는 단위 테스트가 적합하다고 생각이 든다.

> 어떻게 테스트 할 것인가?

위에서 구현 할 때 나는 도메인을 기준으로 나누었기 때문에 서비스 계층이 도메인에 속해있다. 도메인에서 추출 할 수 있는 서비스 계층은 아무래도 검증 하는 로직, 점수 판별 등 다양하게 존재 하는데 이 것을 도메인 내부에서 모든 것을 해결했는데 아키텍처 패턴을 이렇게 사용 해본적이 처음이었다.

그렇기 때문에 도메인을 테스트 할텐데 최대한 Branch Coverage를 높게 달성 하는 것을 목표로 할 것이다.

:bulb: Branch Coverage란?

보통 메서드 테스트는 메서드를 테스트 했는가, 안 했는가로 나뉜다면 브랜치 커버리지는 메서드 내부에 조건 분기까지 커버리지에서 측정한다. 그렇기 때문에 메서드 내부에서 일어나는 행동들을 추적할 수 있다.

> Branch Coverage가 오르지 않았던 도메인

유일하게 브랜치 커버리지가 100%가 되지 않았던 도메인은 `컴퓨터` 도메인이었는데 이 도메인의 메서드를 살펴보자.

{% code title="BaseballApplication.java" overflow="wrap" lineNumbers="true" %}
```java
public ArrayList<String> readyToGameStart() {

    if (isGenerateRandomNumberSizeEqualToLimit()) {
        return randomNumbers;
    }
    String number = getRandomNumberToString();

    if (doesNotDuplicate(number)) {
        randomNumbers.add(number);
    }
    return readyToGameStart();
}
```
{% endcode %}

{% code title="ComputerTest.java" overflow="wrap" lineNumbers="true" %}
```java
class ComputerTest {

    @DisplayName("컴퓨터 플레이어의 난수 생성 메서드는 3개여야한다")
    @Test
    void 길이_검증() {
        // Given
        Computer computer = new Computer();

        // When
        int randomNumberGeneratorSize = computer.readyToGameStart().size();

        // Then
        assertThat(randomNumberGeneratorSize).isEqualTo(3);
    }

    @DisplayName("컴퓨터 플레이어의 난수 생성 메서드는 중복을 포함할 수 없다")
    @Test
    void 요소_중복_검사() {
        // Given
        Computer computer = new Computer();
        ArrayList<String> fixture = computer.readyToGameStart();

        // When
        int randomNumberGeneratorSize = (int) fixture.stream()
                .distinct()
                .count();

        // Then
        assertThat(randomNumberGeneratorSize).isEqualTo(3);
    }

}

```
{% endcode %}

![image](../../.gitbook/assets/branch\_coverage.png)

* 해당 도메인의 브랜치 커버리지는 83%이다.

> 브랜치 커버리지 원인 분석

![image](../../.gitbook/assets/branch\_coverage\_reason.png)

중복을 검사 하는 메서드에서 실패 케이스가 검증 되고 있지 않다. 그렇다면 스스로에게 질문 해볼 수 있다. **Random을 어떻게 테스트 할 것인가?**

### `Random` 은 어떻게 테스트 할 것인가?

[> 참고 블로그](https://yonghwankim-dev.tistory.com/576)

`Mocking` 을 이용 하여 `Random` 모듈의 반환 값을 테스트 할 예정이다. 아무래도 알 수 없는 임의의 값을 테스트 하는 것은 의미가 없으니 알고 있는 값을 테스트 한다면 기능 동작에 대한 단위 테스트 검증이 가능하기 때문이다.

테스트를 하다보면 이미 구현 되어있던 객체가 수정되어야 테스트가 가능해지는 경우가 종종 발생 하는데 이번 컴퓨터 도메인은 그런 케이스에 해당한다.

> 테스트 하기 위한 컴퓨터 도메인 수정하기

{% code title="Computer.java" overflow="wrap" lineNumbers="true" %}
```java
public class Computer {

    public static final int BOUND = 9;
    private final RandomGenerator randomGenerator;
    private final ArrayList<String> randomNumbers = new ArrayList<>();

    public Computer(RandomGenerator randomGenerator) {
        this.randomGenerator = randomGenerator;
    }
}
```
{% endcode %}

기존에는 합성을 사용해서 `Random` 클래스를 직접 생성 하고 있었다면 현재는 외부에서 주입하는 방식으로 변경 하였다. 그렇게 하여금 테스트 할 때 컴퓨터 객체를 생성 하기 위해 클래스를 주입 하면 되는데, 이 때 `Mocking` 을 이용하여 값을 예상 할 수 있다.

> 모킹을 이용한 테스트 케이스 만들기

{% code title="ComputerTest.java" overflow="wrap" lineNumbers="true" %}
```java
    @DisplayName("컴퓨터 플레이어의 난수 생성 메서드는 중복을 포함할 수 없다")
    @Test
    void 요소_중복_검사() {
        // Given by Mocking
        RandomGenerator mockRandom = Mockito.mock(RandomGenerator.class);
        Mockito.when(mockRandom.getRandomNumberToString(9))
                .thenReturn("1", "1", "2", "3");

        Computer computer = new Computer(mockRandom);

        // When
        ArrayList<String> fixture = computer.readyToGameStart();
        System.out.println("fixture = " + fixture);
        int randomNumberGeneratorSize = (int) fixture.stream()
                .distinct()
                .count();

        // Then
        assertThat(randomNumberGeneratorSize).isEqualTo(3);
        assertThat(fixture).containsExactly("1", "2", "3");
    }
```
{% endcode %}

![image](../../.gitbook/assets/branch\_coverage\_result.png)

결과적으로 브랜치 커버리지가 총 83%까지 올랐고 문제가 되었던 컴퓨터 도메인의 브랜치 커버리지가 100%가 되었다. 이렇게 테스트를 위한 환경을 구성 하면서 객체가 수정 되는 일도 발생 하고, 모킹을 이용 해야 하는 경우도 있다. 하지만 알아두어야 할 것은 그 사용이 적재적소에 잘 사용 되고 있는지를 점검 해봐야 한다.

만약, 테스트를 하는 데 프라이빗 메서드가 퍼블릭 메서드에서 호출 되는게 너무 많고 그 내용이 제대로 테스트 되지 않을 땐 리팩터링을 고려 해봐야한다. 그 외에도 의존성이 실버뷸렛이 되지 않기 때문에 테스트를 어떠한 관점에서 하는지, 어떤 로직이 핵심적으로 테스트가 되어야 하는지, 단위 테스트를 해야 하는지 아니면 통합 테스트로 넓게 회귀 케이스를 방지 해야하는지 등 상황에 맞게 잘 활용 하는 것이 핵심이다.
