---
description: 테스트 코드를 작성하기 전 기본 개념 이해하기
---

# 테스트 사전 지식

### <mark style="color:blue;">TDD</mark>, Red - Green - Refactor

***

<figure><img src="../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

{% stepper %}
{% step %}
### <mark style="color:red;">RED</mark>

구현해야 할 메서드를 생성하지 않고 예상대로 흘러갈 수 있도록 골조만 작성한다.

이 때, 컴파일 에러가 나는 것을 두려워하지 말고 테스트 코드를 동작시켜 실패하는 것을 유도해야한다.

{% code overflow="wrap" %}
```java
    @Test
    @DisplayName("주문 목록에 담긴 상품들의 총 금액을 계산할 수 있다.")
    void calculateTotalPrice() {
        // given
        CafeKiosk cafeKiosk = new CafeKiosk();
        Americano americano = new Americano();
        Latte latte = new Latte();

        cafeKiosk.add(americano);
        cafeKiosk.add(latte);

        // when
        int totalPrice = cafeKiosk.calculateTotalPrice();  // compile error!

        // then
        assertThat(totalPrice).isEqualTo(8500);
    }
```
{% endcode %}


{% endstep %}

{% step %}
### <mark style="color:green;">GREEN</mark>

실패한 테스트 코드를 최대한 빠르게 통과 시키는 것이 핵심이다.&#x20;

이 때, 어떠한 행위도 허용된다 마치 아래 처럼 하드코딩 하는 것도 마찬가지다.

{% code overflow="wrap" %}
```java
    public int calculateTotalPrice() {
        return 8500
    }
```
{% endcode %}


{% endstep %}

{% step %}
### <mark style="color:blue;">REFACTOR</mark>

테스트 코드가 빠른 피드백을 제공하기 때문에 해당 메서드는 어떠한 방식으로도 리팩터링을 할 수 있다.

맨 처음 `For Each` 구문을 사용해서 향상된 반복문으로 금액을 더하는 코드였고 그 코드가 정상적으로 통과 되었다면 반복문 보다 컬렉션을 사용할 수 있는 API를 활용하는 구조로 변경하는 것은 훨씬 쉽기 때문이다.

```java
    public int calculateTotalPrice() {
        return beverages
                .stream()
                .mapToInt(Beverage::getPrice)
                .sum();
    }

```
{% endstep %}
{% endstepper %}

{% hint style="success" %}
TDD의 핵심은 _**빠른 피드백을 통해 코드를 점진적으로 리팩터링 가능하다**_ 는 것이다.

TDD의 선순환은 _**메서드 시그니처를 설계할 때 외부에서 주입 가능하도록 사고를 하기 비교적 쉽다**_.

결론적으로 TDD는 관점의 차이를 가져온 설계 방식이다.
{% endhint %}

{% hint style="info" %}
_**더 알아보기**_

* 애자일 방법론
* 폭포수 방법론
* 익스트림 프로그래밍
* 스크럼, 칸반
{% endhint %}



### 테스트 코드는 문서이다

***

{% hint style="info" %}
* 프로덕션 기능을 설명하는 테스트 코드 문서
* 다양한 테스트 케이스를 통해 프로덕션 코드를 이해하는 시각과 관점을 보완
  * 해피 케이스의 시야만 갖지 않고 엣지 케이스의 시야를 얻을 수 있음
* 어느 한 사람이 과거에 경험했던 고민의 결과물을 팀 차원으로 승격시켜서 모두의 자산으로 공유
{% endhint %}



#### :writing\_hand: <mark style="color:blue;">Display Name</mark> 을 섬세하게 작성하자

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
*   도메인 용어를 사용하여 한층 추상화된 내용을 담기

    ↪ 메서드 자체의 관점보다 도메인 정책 관점으로
* 테스트의 현상을 중점으로 기술하지 말 것
  * 테스트가 실패한다, 성공한다 또는 "\~테스트"  지양하기
{% endhint %}



#### _<mark style="color:blue;">BDD</mark>_, Given - When - Then

{% hint style="info" %}
<mark style="color:green;">**Given**</mark>: 시나리오 진행에 필요한 모든 준비 과정(객체, 값, 상태 등)&#x20;

<mark style="color:green;">**When**</mark>: 시나리오 행동 수행

<mark style="color:green;">**Then**</mark>: 시나리오 진행에 대한 결과 명시, 검증



1. TDD에서 파생된 개발 방법
2. 함수 단위의 테스트에 집중하기보다, 시나리오에 기반한 테스트케이스(TC) 자체에 집중하여 테스트한다.
3. 개발자가 아닌 사람이 봐도 이해할 수 있을 정도의 추상화 수준(레벨)을 권장
{% endhint %}







