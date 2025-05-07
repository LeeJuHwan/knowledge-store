# 좌표 계산기 미션 구현

## <mark style="color:blue;">"Good Game Well Played"</mark>

[Mission PR 보러가기](https://github.com/java-playground-hiking/java-coordinate-playground/pull/17)



## <mark style="color:purple;">Goal</mark>

**미션을 진행 하기 전에 손이 먼저 나가지 않는다**. 항상 버릇처럼 손이 먼저 튀어나와 코드 부터 작성하려 하는 습관이 있다. 요구사항을 분석하고, 어떤 기능을 설계할지 명세서를 작성하는 것을 습관화 한다.

**테스트 주도 개발 사이클을 활용한다**. 테스트 코드 없이 절대 프로덕션 코드를 작성하지 않는다. 모래주머니를 차고 달리기 훈련을 하듯 본래 수준보다 살짝 높은 환경에서 최고의 몰입도를 만들어내자.

**getter 메서드를 지양하자**. 가능한 최대한 객체의 속성 값에 직접 접근 하지 않고, 값을 물어보거나 객체가 직접 처리하도록 하자.



## <mark style="color:green;">Goods</mark>

> _"TDD 방식에서 또 다른 시선을 얻다"_
>
>
>
> **"일단 시작은 `create` 부터 만든다?**" ❌
>
> 객체를 생성하는 테스트 사이클을 만들면 어쩔 수 없이 getter를 이용해서 속성값에 의존한 테스트를 진행하게 되더라구요. 그러다보니 생성할 때 필요한 인자 값을 기준으로 객체를 생각하게 되어 객체 행동 중심의 사고가 떨어졌어요.
>
> "생성은 잘 되었다고 가정하고 해당하는 객체가 어떤 행동을 할 것인가?" 를 먼저 고려하면 행동 위주의 사고가 가능했습니다. 그렇게 작성된 테스트 코드는 해당 기능에 대한 구현을 확실하게 만들어주었어요.
>
> \
> 항상 테스트 코드의 시작점을 어떻게 해야할지 고민 해왔던터라 `create` 부터 만드는 습관을 버리게 되었어요.



TDD 를 시작할 때 내 모습을 떠올려보자면 항상 객체의 생성 부분을 테스트 하고 있었다. 예를 들면 아래와 같은 코드이다.

```java
 @Test
 @DisplayName("두개의 좌표를 입력 하면 X와 Y는 10이어야한다.")
 void coordinateValues() {
     Coordinate coordinate = Coordinate.of(10, 10);

     assertThat(coordinate.getX()).isEqaulTo(10);
     assertThat(coordinate.getY()).isEqaulTo(10);
}
```

이렇게 작성한 코드의 문제점은 일단 생성자를 가지고 테스트할 수 있는게 없다. 그렇기 때문에 객체 생성과 동시에 `getter` 메서드가 생성된다. 그럴거면 캡슐화가 왜 필요한가?

그리고, 생성자 테스트 이후 어떤 행동을 만들어낼지 감이 오지 않는다.&#x20;



아래 예시를 들면, `Line` 이라는 객체에 대한 TDD를 작성할 때 `create()` 로 먼저 객체가 잘 생성되는지 테스트하는 것 보다, 객체에 가장 핵심적인 비즈니스 로직인 두 점 사이의 거리를 구하는 메서드를 먼저 테스트 하는 것이 훨씬 사고 확장에 도움이 된다.

```java
@Test
@DisplayName("두개의 좌표를 입력 하면 두 점 사이의 길이를 반환한다")
void lineLength() {
     Coordinate coordinate1 = Coordinate.of(10, 10);
     Coordinate coordinate2 = Coordinate.of(14, 15);
     Points points = Points.from(List.of(Point.of(coordinate1), Point.of(coordinate2)));

     assertThat(Line.from(points).calculate()).isEqualTo(6.403124, offset(0.00099));
}
```



## <mark style="color:red;">Weaknesses</mark>

구현하기 힘든 요구사항을 빠트리고 개발 했었다. 그 것을 알고도 리팩터링 단계에 반영하지 않았다.

실제 사용자의 요구사항도 아니었고 이 개발이 주는 보상이 없었기 때문에 간과 했었다. 미션을 임하는 자세가 시간이 지나면서 점차 흐트러지는 것 같다.

> "네 점이 뒤틀어진 사다리꼴이나 마름모는 제외하고 직사각형만 허용하도록 검사한다."

이 요구사항을 개발 했었어야 했으나 까먹고 개발하지 못한 점, 이 것이 스노우볼을 굴려 리팩터링 단계에서 코드가 귀찮다는 이 무슨 오만한 심정을 만들어내었다.

두 번 다시는 이런 감정을 갖지 않도록 하려면 어떻게 해야할까?

{% hint style="info" %}
콘솔 게임 개발을 단면에 그치지 않고 확장 가능성과 도메인 설계에 대한 철학을 배우는 학습으로 생각해야한다.

그 당시 시간에 급급해 미션을 풀어나가기에 그쳤던 과거와 달리, 이 속에서의 인사이트를 뽑아내기 위해 의도적으로 연습해야한다.
{% endhint %}



## <mark style="color:yellow;">Progresses</mark>

**코드를 단순하게 바라보지 말고, 서비스라는 관점에서 확장성을 고려한 개발을 한다.** 내가 적었던 가장 못했던 점에 언급 했듯이 코드 작성에 귀찮음을 느끼면 절대 안된다는 것이다. 그 지루함을 어떻게든 부숴야한다.

**객체를 설계하고 계층을 구조화 할 수록 근거가 필요하다.** 근거 없이 누군가의 패턴을 그대로 모방하는 것은 같은 문제를 만났을 때 응용할 수 없다. 만약 내게 특정 조건에 대한 조건문을 줄이기 위해 Map 구조를 썼다면, Enum 은 어떤가? 또 다른 객체에선 어떨까? 등 사고의 확장이 더 필요하다.



## <mark style="background-color:orange;">Feedbacks</mark>

당연 첫번째 피드백의 내용은 요구사항 미준수다. 이건 내가 봐도 할 말이 없고 최악의 행동을 저질렀다.&#x20;

두 번째는 조건문을 어떻게 효율적으로 처리할 수 있었는지에 대한 긍정적인 코멘트가 따랐다.

{% tabs %}
{% tab title="FigureFactory.java" %}
```java
package model.figure;

import java.util.Arrays;
import model.coordinate.Coordinates;
import model.point.Points;

public enum FigureFactory {

    LINE(2) {
        @Override
        public Figure createFigure(Points points) {
            return Line.from(points);
        }
    },
    TRIANGLE(3) {
        @Override
        public Figure createFigure(Points points) {
            return Triangle.from(points);
        }
    },
    RECTANGLE(4) {
        @Override
        public Figure createFigure(Points points) {
            return Rectangle.from(points);
        }
    };

    private final int figureSize;

    FigureFactory(int figureSize) {
        this.figureSize = figureSize;
    }

    public abstract Figure createFigure(Points points);

    public static Figure create(Coordinates coordinates) {
        return Arrays.stream(values())
                .filter(type -> coordinates.isEqualSize(type.figureSize))
                .findFirst()
                .orElseThrow(IllegalArgumentException::new)
                .createFigure(coordinates.toPoints());
    }
}
```
{% endtab %}
{% endtabs %}

아무래도 조건문 대신 가독성 좋은 객체에서 관리했기 때문에 좋은 평을 받았다.

> "Map 구조로 작성 하면 어땠을까?"

당연히 O(n) 방식으로 데이터를 찾는 것이 아닌 O(1) 방식을 활용하면 어떨까란 관점이었는데, 관리하는 데이터가 많지 않을 것 같기도하다.



미션을 진행 하면서 시간이 오래걸리는 것은 각자의 환경이 있기 때문에 어쩔 수 없다. 하지만, 본인 스스로가 일부러 학습을 놓고 있는 것은 아닌지 점검해야 할 때가 왔다.

그 이유는 단순한 이 미션만 1달 반 정도 시간을 썼던 것 같다. 그리고 이 미션이 끝난 뒤 스터디가 그대로 끝나는 느낌도 있었는데 아주 간신히 연휴를 맞이해 연명하고 있다.

