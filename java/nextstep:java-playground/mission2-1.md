# Mission1. Baseball-Unit Test

<details>

<summary>Properties</summary>

:pencil:2024.08.23

:computer: [source\_code](https://github.com/LeeJuHwan/java-baseball/tree/mission2-jhlee/src/main/java)

</details>

## 숫자야구게임 구현


### 요구사항 이해하기


> 용어 정리

- 플레이어: 나

- 상대방: 컴퓨터

- 같은 위치의 숫자: 713 중 1번째 요소는 "7", 사용자가 713을 입력 했다면 1번째 요소가 같음.

1. 컴퓨터는 1 부터 9까지 랜덤한 숫자로 3자리를 구성 해야한다.

2. 플레이어는 컴퓨터가 생성한 3자리 난수를 맞춰야한다.

3. 플레이어가 정답을 입력 할 때 마다 결과로 힌트가 제공 된다.

    - 플레이어가 입력한 숫자와 컴퓨터의 임의 숫자 중 같은 위치의 값이 같다면 "스트라이크"

    - 플레이어가 입력한 숫자가 같은 위치에 없지만 컴퓨터의 숫자 중 속해있다면 "볼"

    - 플레이어가 입력한 숫자가 아무 것도 포함되지 않는다면 "낫싱"

    - 예시
        - 컴퓨터 정답: 713

        - 1. 플레이어 입력: 123 -> 결과: 1볼 1스트라이크

        - 2. 플레이어 입력: 671 -> 결과: 2볼

        - 3. 플레이어 입력: 456 -> 결과: 낫싱

        - 4. 플레이어 입력: 713 -> 결과: 3스트라이크

4. 게임이 종료 되면 다시 실행 할지, 완전히 종료 할지 사용자에게 입력을 받는다.

<br></br>

### 도메인 추출하기

요구사항에 따른 도메인을 먼저 분석 해서, 해당 도메인 객체가 행동 할 수 있는 메서드를 만드는게 먼저라고 생각했다. 위 요구사항을 보면 가장 거시적으로 있어야 할 도메인 객체는 `유저`, `컴퓨터`가 있고 유저와 컴퓨터 사이의 연결을 해줄 객체가 필요해보인다.

> 왜 연결을 해줄 객체가 필요할까?

프로그램이 빌드 한 시점 이후로 `컴퓨터` 객체는 난수를 생성 해서 본인만 알고있다. 그 후, 플레이어가 정답을 입력 하여 그 숫자를 맞추는 과정이 진행 되는데 이 때 정답인지 아닌지 판별은 어떻게 할 것인가? 다짜고짜 컴퓨터에게 "이거 정답이야?" 라고 물어본다면, 컴퓨터는 당연히 "맞아" 또는 "아니야"가 대답이 될 것이다.

> 그렇다면 내가 필요한건 정답 여부인가?

정답 여부도 필요한 것은 맞지만 위 요구사항은 점수를 기록 해서 플레이어가 접근 하기 쉽도록 힌트를 제공 하고 있다. 그렇다면 컴퓨터에게 점수도 물어볼 것인가?

이 도메인을 추출하는 데 모호한 이유는 흔히 `SRP` 라는 단일 책임의 원칙 때문일 것이다.

컴퓨터 객체가 해야 할 일은 무엇일까? 고민 해본다면 정답을 생성 하는 것 이외에 행동 하는 것이 없다. 이 프로그램의 주체는 플레이어이기 때문이다.
다시 질문으로 돌아와서, 내가 필요한건 정답 여부는 아니라는 것이다. 정답을 판별 해줄 객체가 필요한데 지금 프로그램은 "야구 게임" 이니까 `심판`이 등장하면 어떨까?

**`컴퓨터`, `유저`, `심판` 그리고 `점수`가 도메인으로 추출되다.**

심판은 어떤 책임을 갖고 있어야할까? 개인적으로 유저가 입력한 정답에 대해 판정을 내려줬으면 좋겠다. 다시 돌아와, 정답 여부만 결정 되는 것이 아닐까?
그렇다면 유저의 정답을 판정 할 때 점수를 측정하면 좋을 듯 하다. 심판이 점수를 주는건 현실에서도 납득이 가는 행동이고, 판정을 내리는 것도 납득이간다.

심판이라는 객체는 점수를 측정하고, 유저가 정답을 입력하면 판정을 내리는 객체가 되었다. 심판이 혼자 힘들게 점수도 입력하고 게임도 진행하고 하면 너무 많은 일을 준 것 아닐까? 내가 만약 게임 속 심판이라면 점수 올리러 뛰어 갔다가 다시 뛰어서 유저가 던진 공이 볼인지, 스트라이크 인지 말해주고, 아웃인지 아닌지도 말 해줘야된다고 생각 하니 아찔하다.

자주 언급 되었던 `점수`도 하나의 객체가 될 수 있을 것 같다. 점수는 앞으로 심판과 협력하는 객체로 심판이 볼이라고 하면 점수 객체는 볼 카운트를 올리고, 스트라이크 라고 하면 스트라이크 카운트를 올리면 서로가 바라보는 관심사는 다르지만 정중하게 요구할 수 있다.


### 객체를 존중하기

객체지향의 캡슐화를 이용하면 속성이 대부분 숨겨지게 된다. 그렇게 숨겨진 속성에 접근하기 위해서 공개 메서드인 게터, 세터가 많이 등장 하게 되는데 **게터와 세터는 매우 폭력적인 존재**라는 것을 배운 적 있다.
게터를 이용하는 경우 내가 만든 객체에게 정중하게 묻는 것이 아닌 강제로 탈취하는 개념이 폭력적이라는 내용을 담았다. 하지만 요구 사항에 따른 개발을 하다보면 게터를 사용 할 수 밖에 없다. 그럴 때만 사용 할 수 있도록 최소화 하는 것이 이번 미션을 해결 하면서 신경 써야 할 문제였다. 게터를 사용 하는 것은 private 속성을 public으로 사용하는 것과 다름 없기 때문에 캡슐화를 지키고자 한다면 게터를 최소화하자.

> 그럼에도 불구하고 `Getter` 를 사용 할 수 밖에 없었던 이유

**User Domain**

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

지극히 개인적인 생각으로 유저 객체가 갖고 있는 숫자는 정답과 비교해야 하기 때문에 게터를 안 쓸 수가 없었다. 게터를 보호하고 싶다면 해당 객체에게 정중하게 물어봐야한다. 그렇다면 유저에게 정답을 묻는 것이 올바른 행동일까 라는 생각이 들었는데, 유저는 문제를 맞추는 사람인데 정답을 알고있다? 굉장히 모순적인 것 같았다. 그렇기 때문에 유저는 어쩔 수 없이 게터를 사용했다.

**Score Domain**

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

스코어 객체에서도 결과를 게터를 통해 가져오고 있다. 앞서 도메인을 추출하는 과정에서 `심판`이 `스코어` 객체를 통해 점수를 산출하고 결과를 받아오는데 이 때 필요한 게터 역할이었다. 만약 여기서도 게터를 보호해야 한다면 어떤 방향으로 흘러갔을까? 게터가 없었다면 결과를 출력하기 위해 `스코어` 객체의 공개 메서드를 통해 결과를 출력 해줬어야 했다. 스코어 객체는 점수를 기록하고 반환하면 되는 객체로 활용 되는데 출력을 담당하는 객체와 또 협력을 맺어야 할 필요가 있을까란 생각이 들어 출력을 빼고 스코어 객체의 속성 값을 가공하여 반환 하기로 변경 하였다.


> `Getter`를 사용하지 않고 외부와 소통한 경우

**Judgement Domain**

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

`심판`은 점수를 산정한 후 `스코어` 객체를 반환한다. 심판이 점수를 산정 했으니 `getScore`를 통해 점수를 반환 했더라면 `Getter`를 사용하게 되는 경우에 속하게 되는데, 위 메서드에서 내부 상태를 변경하고 그 객체 자체를 반환 하고있다. 이렇게 반환 된 객체는 객체의 메서드를 통해 결과를 받아올 수 있어 캡슐화를 지키고 있다고 생각이 들었다. 


### 추상화 레벨을 동등하게 유지하기

인간은 추상적인 개념을 이해하는 데 굉장히 뛰어나다. 아무래도 "컴퓨터" 라는 단어를 이해하는 것과 "0과 1로 이루어진 바이너리 명령어를 처리하는 기계" 라는 것을 이해하는건 전자가 훨씬 쉽기 때문에 소스코드에서도 읽기 좋은 코드를 작성할 때 가장 중요시 되는게 추상이라는 개념이다. 

그 중, 추상화 레벨을 동등하게 유지하여 "컴퓨터" 라고 작성하던 코드에서 갑자기 "명령어 처리 기계" 라고 하지 않는 습관을 들이기 위해 의도적인 연습을 하고있다.

지금까지 추출한 도메인 내부는 추상적인 메서드를 사용하기 위해 만들어 놓은 구현부이다. 지금 확인 해볼 위치는 `Application` 의 게임 실행 부분인데, 이 때 뜬금 없이 구현 코드가 등장하지는 않는지 메서드명은 이해하기 쉬운 내용인지를 고민 해보면 좋다.

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
        String userInputNumber = userAction();
        ArrayList<String> userActionResult = userReadyComplete(userInputNumber);

        Score score = judgment.judge(userActionResult);

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

가장 대표적으로 확인 해볼 수 있는건 게임을 시작하는 로직인데 위 작성된 코드를 살펴보면서 어떤 내용을 의도적인 수련을 하고자 했던건지 알아보려고 한다. 일단 코드를 위에서 부터 아래로 읽으면서 내려오면 이러한 해석이 가능하다.

1. `startGame` 을 실행

- 심판은 게임 할 준비가 되었다고 알려준다.

- 게임을 시작한다는 내용이 출력된다.

- 게임이 시작되고 있다고 기록한다.

- 게임이 시작되고 있을 땐 `run` 이라는 메서드를 통해 게임을 실행 시킨다.

- 게임이 종료되면 잠시 멈춘 뒤 유저가 게임 실행 옵션에 대해 선택하고 이 내용을 기록한다.


2. `run` 을 실행

- `actionToGameStartByJudgement` 라는 메서드가 실행 되는데 직역 해보면 "심판이 게임 시작을 위한 행동을 게시했다." 라고 해석할 수 있다.

- 게임 실행 도중 문제가 생기면 문제를 투명하게 공개하고, 다시 시작한다.


3. `actionToGameStartByJudgement` 를 실행

- 유저가 어떠한 행동을 한다. -> 이 부분은 메서드 네이밍이 잘못되었다고 느껴지기 때문에 수정이 필요하다.

- 유저는 게임을 시작 할 준비가 완료 됐다고 알린다. -> 이부분 또한 유저가 시작할 준비가 완료 되고 바로 판정하는 것이 어색하다. 여기도 변경이 필요하다.

- 심판이 판정을 내린다.

- 점수판에 점수가 기록된다.

- 점수판에서 "3스트라이크"인지 확인한다.

- "3스트라이크"가 맞는지 아닌지 확인 한 후 게임을 재개하거나 종료한다.


4. `pauseForUserGameRunOptionSelect` 를 실행

- 유저가 게임을 재시작 할 것인지 종료 할 것인지 고른다.

- 게임을 고르다가 문제가 발생 하면 문제 이유를 알려주고, 다시 고른다.

- 게임을 계속 시작 할 것인지 알려준다.

이렇게 긴 내용을 풀어서 해석 할 수 있다. 추상의 개념을 통해 메서드를 쉽게 이해할 수 있었으며 다소 불편한 부분도 존재 했지만 이런식으로 이해하기 쉬워진다. 만약, 게임을 다시 시작 할 것인지 고르는 부분에서 `userInput.equals(1)` 이라고 한다면 1이 무엇을 의미하고 어떤 단계인지 알기 어렵다. **이렇게 해석하기 어려운 코드는 구현부에 위치하고 해석하기 쉬운 이름들을 지어주는 행위를 추상화 레벨을 맞춰준다** 라고 표현 할 수 있다.


### 테스트 하기