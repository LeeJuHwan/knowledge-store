# Mission1. Baseball-String Calculator

<details>

<summary>Properties</summary>

:pencil:2024.06.17

:computer: [source\_code](https://github.com/java-playground-hiking/java-baseball/pull/8/commits/c9ced178a51f2856a80b5f02fb81c79d0811f02d)

</details>

### 넥스트 스텝: 자바 플레이그라운드 - 숫자 야구게임

#### 숫자야구게임 - 1. 문자열 계산기 구현 및 단위 테스트 작성

#### 문자열 계산기 기능 구현

✅ **요구사항**

1. 사용자가 입력한 문자열 값에 따라 사칙연산을 수행할 수 있는 계산기를 구현해야 한다.
2. 연산자 우선 순위는 적용 하지 않으며 왼쪽 부터 사칙연산을 수행한다.

> 문제 정의

1. 사용자가 입력할 수 있는 `Scanner`를 이용 할 `User`가 필요함.
2. 문자열을 입력 받아서 개별 요소에 접근이 쉬운 `Array` 배열로 변환이 필요함.
3. `Array`에서 개별 요소에 접근 할 때 마다 문자열 수식이 피연산자인지, 연산자인지 구분이 필요함.
4. 연산자에 따른 피연산자와의 연산을 수행 하는 핵심 비즈니스 로직이 필요함.

\


:point\_right: 필요한 내용을 코드로 옮길 때, 어떤 아키텍처 구조가 만들어져야 할까?

* 역할과 관심사 분리를 위한 계층 구조 설계
* 사용자의 입, 출력을 다루는 `Presentation Layer`
* 문자열 수식을 연산 할 `Service Layer`
* 어플리케이션을 실행 할 `Main`

:clap: 위 방식으로 가장 러프하게 코드가 작동 할 수 있도록 구현

\


> :man\_running: 작동하는 코드 구현하기

기능이 정상적으로 작동할 수 있는 가장 작은 단위의 개발을 시작 하고 클래스명을 기준으로 계층을 잠시 나눴다. `CalculatorService`, `CalculatorUser`, `StringCalculatorMain` 이렇게 파일을 생성 한 뒤 가장 쉽게 접근 할 수 있는 사용자의 입력 값을 받는 과정을 먼저 구현 하고 `split` 메서드를 활용한 `String -> Array` 변환 작업을 마무리 지었다.

문자열 수식이 배열로 변환 된 상태에서 배열의 요소에 접근 하여 숫자와 연산자를 분리 해야 했는데, 파이썬에서는 `type`으로 쉽게 검사 할 수 있었으나 아쉽게 자바에서는 레퍼런스가 대부분 직접 구현 하는 내용이거나, Apache의 패키지를 이용 하는 것이었는데 미션을 해결하는 데 중점을 뒀다면 당연히 외부 모듈을 사용하여 손 쉽게 풀었을 것 같다. 하지만 자바에 익숙해지는 것이 목표이기 때문에 직접 구현했다.

**직접 구현한 숫자 타입 검사**

```java
/**
 * 문자로 이루어진 데이터를 받아 숫자 여부 확인
 * @param string 문자열 데이터
 * @return boolean - true: 숫자, false: 문자
 */
public boolean isNumeric(String string) {
    try {
        Integer.parseInt(string);
        return true;
    } catch (NumberFormatException e) {
        return false;
    }
}
```

연산자 부분은 파이썬을 사용 하면서 딕셔너리의 키는 매핑 값, 밸류는 함수의 오브젝트 주소 값을 저장 한 후 사용을 많이 해 왔는데 자바에서도 람다가 지원되고 있어 비슷하게 구현 할 수 있었다.

```java
// NOTE: 연산자 옵션
public static final Map<String, BiFunction<Integer, Integer, Integer>> operator = new HashMap<>();
static {
    operator.put("+", (x, y) -> x + y);
    operator.put("-", (x, y) -> x - y);
    operator.put("*", (x, y) -> x * y);
    operator.put("/", (x, y) -> {
        if (y == 0) {
            throw new ArithmeticException("can't division by zero");
        }
        return x / y;
    });
}

```

위 코드를 작성 하면서 구글링을 정말 많이 했었는데 `BiFunction`을 알기 까지 시간을 정말 오래 쓴 것 같다. 무아지경으로 검색 하다보니 `BiFunction` 키워드를 얻었고 [해당 블로그](https://developer-talk.tistory.com/716) 에 있는 내용을 참고로 해서 코드를 작성 했다.

이렇게 피연산자와 연산자를 구분 할 수 있는 상황이 되었으니 바로 문자열 계산기의 핵심 비즈니스 로직인 연산 로직을 작성 했다.

문자열 수식을 연산 하기 위해 배열로 반환 받았던 데이터는 패턴이 존재하는데, 0번째 부터 짝수 인덱스는 피연산자이고, 홀 수는 연산자 기호이다.

그렇다면 인덱스 번호를 기준으로 반복 하면서 연산자만 뽑아도 되고, 피연산자만 뽑아도 되니 익숙하게 작성하기 위해 0번째 부터 반복 할 수 있는 피연산자를 타겟했다.

**문자열 수식 연산**

```java
/**
 * 문자열 수식 계산기
 * @param expressionValues `Array`로 변환 된 문자열 수식어
 * @return 연산 결과, 우선 순위 없이 왼쪽 부터 연산 수행
 */
public int calculate(String[] expressionValues) {
    if (!isNumeric(expressionValues[0])) {
        throw new IllegalArgumentException("Invalid math expression must be start with number");
    }

    String op = "";
    int expresionResult = Integer.parseInt(expressionValues[0]);
    int operandLast = 0;

    // NOTE: 짝수 인덱스는 항상 피연산자며, 홀수 인덱스는 항상 연산자가 되어야 정상 수식의 패턴이 됨
    for (int i = 0; i < expressionValues.length - 2; i += 2) {
        String expressionNumber = expressionValues[i + 2];

        if (isNumeric(expressionNumber)) {
            operandLast = Integer.parseInt(expressionValues[i + 2]);
        }

        if (isOperator(expressionValues[i + 1])) {
            op = expressionValues[i + 1];
            // NOTE: 연산 결과를 첫번째 인자로 넘기기 위해 `operandFirst` 변수에 반환 값을 할당
            expresionResult = operator.get(op).apply(expresionResult, operandLast);
        }
    }
    return expresionResult;
```

이전에 연산 했던 값을 다음 피연산자와 다시 연산을 해야 했기 때문에 재사용 하였고 이렇게 문자열 연산을 콘솔로 테스트 해 보았을 때 정상적으로 값이 반환 되는 것을 확인 할 수 있었다.

\


#### 리팩터링

***

정상적인 기능을 하는 코드는 작성이 완료 되었으니 이제 객체가 띄고 있는 특성들을 이용 해서 단일 책임 원칙, 역할과 관심사 분리 등 개발 방법론에 있어 추 후 테스트 시 단일 메서드 테스트가 가능할 수 있도록 유도 할 계획이며 메인 어플리케이션 로직은 절대 수정하지 않을 것이다.

일단, 비즈니스 로직이 담겨 있는 서비스 파일에 모든 코드가 담겨있다. 피연산자를 구분하는 방법, 연산자 기호를 기준으로 연산 하는 메서드, 문자열 연산, 배열로 변환 등 서비스 계층에 모두 의존하게 될 것 같으니 분리 시킬 필요가 있다.

> :worried: 연산자를 다루는 객체와 피연산자(문자열 수식의 수)는 인스턴스화가 필요한 객체일까?

연산자 기호를 담고 있는 객체는 아무래도 유틸성 클래스 느낌이 강하다. 그 이유는 하나의 수식에는 여러 개의 연산 기호들이 있는데, 이 때 마다 연산 기호를 넘겨 받은 클래스를 생성 하는 것은 너무 비효율적이다. 결론은 연산자 기호 객체는 유틸성 클래스로 만들어야한다.

피연산자를 다루는 객체는 문자열 수식을 상수 값으로 저장 하고 사용 하면 가능할 것 같다. 하지만, 문자열 수식을 여러번 입력 하는 상황이 온다면 이 또한 수식에 대한 처리를 하기 위해 여러 번 인스턴스화를 해야 하기 때문에 유틸성으로 판단하였다.

**분리된 연산 기호 객체**

```java
public class Operator {

    // NOTE: 연산자 옵션
    private static final Map<String, BiFunction<Integer, Integer, Integer>> operator = new HashMap<>();
    static {
        operator.put("+", (x, y) -> x + y);
        operator.put("-", (x, y) -> x - y);
        operator.put("*", (x, y) -> x * y);
        operator.put("/", (x, y) -> {
            if (y == 0) {
                throw new ArithmeticException("can't division by zero");
            }
            return x / y;
        });
    }

    public static BiFunction<Integer, Integer, Integer> getOperator(String symbol) {
        BiFunction<Integer, Integer, Integer> op = operator.get(symbol);

        if (op == null) {
            throw new IllegalArgumentException("does not matching operator symbol");
        };
        return op;
    }
}

```

**분리된 피연산자 객체**

```java
public class StringExpression {

    /**
     * 문자열 수식을 배열로 변환
     * @param mathExpression 문자열 수식
     * @return 문자열 수식을 공백을 기준으로 `split` 한 배열 반환 값
     */
    public static String[] convertExpressionArraysByBlank(String mathExpression) {
        return mathExpression.split(" ");
    }

    /**
     * 문자로 이루어진 데이터를 받아 숫자 여부 확인
     * @param string 문자열 데이터
     * @return boolean - true: 숫자, false: 문자
     */
    public static boolean isNumeric(String string) {
        try {
            Integer.parseInt(string);
            return true;
        } catch (NumberFormatException e) {
            return false;
        }
    }

}
```

이렇게 단일 책임의 원칙에 따라 객체를 분리 하고, 서로가 의존할 필요가 없는 구조가 되었다. 이렇게 설계를 했을 때 테스트 할 때도 편리하다. 결국 단일 메서드의 정상, 예외 케이스만 검증 하면 끝이기 때문이다.

문자열 연산을 수행 하는 서비스 계층은 이제 연산하는 메서드 하나만 존재 한다. 연산이 필요할 때만 서비스 계층에서 비즈니스 로직을 처리 하면 된다.

구조를 잡은 뒤 코드를 살펴보면 나름 예외 케이스를 설정 했다고 한들 빠져나갈 틈들이 보였다.

1. 수의 나눗셈 -> a / b 일 때, a는 앞서 연산 된 값을 받은 인자이기 때문에 0이 아닐수도 있고 맞을 수도 있지만, 대체적으로 b가 0인 경우 예외처리를 하였는데, a가 0일 수 있다는 내용을 빠트렸다.

```java
-             if (y == 0) {
->            if (x == 0 || y == 0) {
```

2. 문자열 수식을 전달 받아 배열로 변환 하는 과정은 공백을 기준으로 한다. 만약, 공백이 중간에 있지만 수식 다음 공백이 오지 않는 경우는 에러 상황을 만들어주는게 좋을 것 같다는 생각이 들었다.

```java
public static String[] convertExpressionArraysByBlank(String mathExpression) {
    for (int i = 1; i < mathExpression.length(); i += 2) {
        char nextChar = mathExpression.charAt(i);

        if (nextChar != ' ') {
            throw new IllegalArgumentException("Expression must be seperated by a space");
        }
    }

    return mathExpression.split(" ");
}
```

이렇게 숫자나 연산 기호 다음 공백이 없는 경우는 에러 처리를 해 주었다.

3. 연산자 기호가 공백을 구분 하여 연속으로 오는 경우가 있다면 에러로 유도를 해야한다.

```java

// NOTE: 짝수 인덱스는 항상 피연산자며, 홀수 인덱스는 항상 연산자가 되어야 정상 수식의 패턴이 됨
for (int i = 0; i < expressionValues.length - 2; i += 2) {
    String expressionNumber = expressionValues[i + 2];

-    if (isNumeric(expressionNumber)) {
-        operandLast = Integer.parseInt(expressionValues[i + 2]);
-    }

->   if (!StringExpression.isNumeric(expressionNumber)) {
->       throw new ArithmeticException("Math expression must be operated on numeric only");
->   }

    operandLast = Integer.parseInt(expressionValues[i + 2]);
    op = expressionValues[i + 1];

```

위에서 사용한 `calculate` 메서드에서 숫자가 아닌 경우는 에러로 유도하면 간단하게 처리 할 수 있다. 기존 소스 코드에서 숫자를 검사한 후 숫자일 때 피연산자 값을 넘겼는데 연산자가 연속으로 오는 경우는 이전 할당 된 값을 기준으로 연산을 하게 되니 의도치 않은 값이 나올 수 있다.

\


#### 테스트 케이스 작성

***

> 🤔 어떻게 테스트 할 것인가?

1. 유틸성 클래스가 제공 하는 메서드의 정상 케이스, 예외 케이스 검증
2. 핵심 비즈니스 로직의 결과 값 검증

그 외 어플리케이션 컨트롤러 단위도 해야 하지만, 콘솔 프로젝트이기 때문에 자주 사용되는 메서드 검증을 위주로 단위 테스트를 작성 할 계획이다. 테스트 코드를 작성 하면 좋은 이점은 굉장히 많은 것 같다. 그걸 알지만 굉장히 많은 시간을 쏟아야 하기 때문에 쉽게 놓치는 작업 중 하나인데, 그 중 이점을 뽑아 보자면 이런 내용들이 있을 것 같다.

* 코드 구현 단계에서 생각 하지 못 했던 예외 케이스
* 회귀 케이스 예방
* 잘못된 메서드 반환 값

\


> `Fixture` 생성

```java
public class StringCalculatorTest {

    String stringExpression;
    static StringCalculator service;

    @BeforeAll
    static void serviceInit() {
        service = new StringCalculator();
    }

    @BeforeEach
    void setUp() {
        stringExpression = "2 + 3 * 4 / 2";
    }
    ...

```

테스트 케이스를 실행 시킬 때 마다 사용 해야 할 문자열 수식과, 서비스 클래스를 인스턴스화 함으로 써 반복되는 코드를 줄일 수 있다. 하지만, 문자열 수식과 같은 경우에는 명확한 `Given`을 주기 위해 테스트 케이스 내부에서 작성해도 상관 없을 것 같다.

> 유틸 클래스 검증

```java
    @ParameterizedTest
    @DisplayName("`StringExpression` 숫자 여부 검증")
    @CsvSource(value = {"1:true", "+:false"}, delimiter = ':')
    void 숫자_여부_검증(String expectedNumericValue, boolean expectedResult) {
        // Given
        String stringExpressionValue = expectedNumericValue;
        boolean expect = expectedResult;

        // When
        boolean result = StringExpression.isNumeric(stringExpressionValue);

        // Then
        assertThat(result).isEqualTo(expect);
    }
```

파이썬에서 테스트 코드의 메서드 이름을 국문으로 짓는걸 선호하는 편인데, 자바에서도 마찬가지로 사용 했다. 하지만 `DisplayName`이 있다면 굳이 메서드를 국문으로 해야 됐을까란 생각도 들었다.

테스트 방법은 `Given-When-Then` 방법으로 사람이 이해하기 가장 쉬운 방법이기 때문에 사용 했고, 대부분의 케이스도 그렇게 사용 해 왔다. 이 방법은 주어진 데이터가 어떠한 행동을 했을 때 이러한 반응을 보일 것이다 라는게 워크플로우 자체를 나타내기 때문에 손 쉽게 이해할 수 있다.

하나의 메서드에서 정상, 이상 검증을 할 때 `ParameterizedTest` 를 이용 하면 여러 데이터를 순차적으로 처리하여 결과를 검증 할 수 있는 장점이 있기 때문에 1개의 메서드에서 여러가지 상황을 나타낼 수 있다.

> 에러 케이스 검증

```java

    @ParameterizedTest
    @DisplayName("`Operator` 연산자 나눗셈 예외 케이스 검증")
    @CsvSource({"0,1", "1,0"})
    void 연산자_나눗셈_예외_검증(int num1, int num2) {
        // Given
        String op = "/";

        // When
        assertThatThrownBy(() -> {
            Operator.getOperator(op).apply(num1, num2);
        })      // Then
                .isInstanceOf(ArithmeticException.class)
                .hasMessageContaining("can't division by zero");
    }
```

에러 케이스를 검증 한다는 것은 의도적으로 내가 해당 케이스를 예외 처리 했다는 내용과 같다. 하지만 의도적이지 않은 경우 이렇게 컴파일 에러를 만나 서비스가 종료되는 것을 미연에 방지 할 수 있다. `Mission1 - 학습 테스트 실습` 과정에서 사용 했던 `Exception`클래스 검증 + `ErrorMessage` 검증을 통해 해당 에러 케이스를 테스트 할 수 있다.

이렇게 테스트 하는 이유는 `Throw` 로 예외 처리를 했지만 `Try-Catch`에서 해당 에러 상황이 안 잡히는 경우를 방지 할 수 있다.

> 핵심 비즈니스 로직 검증

```java
    @Test
    @DisplayName("`StingCalculator` 문자열 연산 검증")
    void 문자열_연산_검증() {
        // Given
        String[] fixture = StringExpression.convertExpressionArraysByBlank(stringExpression);

        // When
        int result = service.calculate(fixture);

        // Then
        assertThat(result).isEqualTo(10);
    }
```

위에서 유틸성 메서드들을 테스트 했기 때문에 비즈니스 로직은 주어진 문자열 수식이 원하는 값을 반환 하는지 테스트만 하면 된다. 그 과정에서 여러 데이터를 `ParameterizedTest`로 넣어 데이터 정합성을 조금 더 요구할 수 있지만 일반적인 케이스이기 때문에 하나의 문자열 수식만 검증 했다.
