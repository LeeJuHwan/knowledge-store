# 새로운 기능 개발하기

{% hint style="info" %}
앞서 만들어두었던 예제에서 간단한 주문시스템과 VIP 고객일 경우 고정 금액 1,000 원을 할인 해주는 정책을 적용하였다.

이번 섹션에서는 기획자에 의해 할인 정책이 변경 되었을 때 코드를 변경 해야 하는 개발자는 어떻게 보다 나은 방향으로 유지보수 할 수 있는지 배울 수 있는 시간이다.
{% endhint %}

### 할인 정책 구현체 만들기

***

{% tabs %}
{% tab title="RateDiscountPolicy.java" %}
```java
package hello.core.discount;

import hello.core.member.Grade;
import hello.core.member.Member;

public class RateDiscountPolicy implements DiscountPolicy {

    private int discountPercent = 10;

    @Override
    public int discount(Member member, int price) {
        if (member.getGrade() == Grade.VIP) {
            return price * discountPercent / 100;
        }
        return 0;
    }
}

```
{% endtab %}

{% tab title="RateDiscountPolicyTest.java" %}
```java
package hello.core.discount;

import static org.assertj.core.api.Assertions.assertThat;

import hello.core.member.Grade;
import hello.core.member.Member;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

class RateDiscountPolicyTest {

    RateDiscountPolicy discountPolicy = new RateDiscountPolicy();

    @Test
    @DisplayName("VIP는 10% 할인이 적용되어야 한다")
    void vip_o() {
        // given
        Member member = new Member(1L, "memberVIP", Grade.VIP);

        // when
        int discount = discountPolicy.discount(member, 10000);

        // then
        assertThat(discount).isEqualTo(1000);
    }

    @Test
    @DisplayName("VIP가 아니면 할인이 적용되지 않아야 한다")
    void vip_x() {
        // given
        Member member = new Member(1L, "memberBASIC", Grade.BASIC);

        // when
        int discount = discountPolicy.discount(member, 10000);

        // then
        assertThat(discount).isZero();
    }
}
```
{% endtab %}
{% endtabs %}

할인 정책이라는 인터페이스를 참조 하고 있는 서비스 계층은, 사용해야 할 구현체만 바꾸면 "고정 금액 할인" 이든, "변동 금액 할인" 이든 자유롭게 관리할 수 있다.

#### 문제점

***

{% tabs %}
{% tab title="OrderServiceImpl.java" %}
```java
package hello.core.order;

import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.discount.RateDiscountPolicy;
import hello.core.member.Member;
import hello.core.member.MemberRepository;
import hello.core.member.MemoryMemberRepository;

public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository = new MemoryMemberRepository();
//    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
    private final DiscountPolicy discountPolicy = new RateDiscountPolicy();


    @Override
    public Order createOrder(Long memberId, String itemName, int itemPrice) {
        Member member = memberRepository.findById(memberId);
        int discountPrice = discountPolicy.discount(member, itemPrice);

        return new Order(memberId, itemName, itemPrice, discountPrice);
    }
}

```
{% endtab %}
{% endtabs %}

위 코드를 보면, 서비스 계층에서 의존성을 갖고 있던 할인 정책 객체를 바꿈으로써 손쉽게 요구사항을 충족할 수 있다.&#x20;

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

{% hint style="warning" %}
#### OCP, DIP 같은 객체지향 설계 원칙을 충실히 준수했다고 볼 수 있지만 그렇지 않다.

위 이미지에 나와있듯이 코드에서 `DiscountPolicy` 인터페이스만 참조하고 있다고 알고 있지만, 사실은 인터페이스를 구현한 구현 클래스도 직접 참조하고 있다.



:bulb: **추상도 의존을 하고, 구체도 의존을 하는 관계가 형성 되어 DIP 를 위반하고 있다**

:bulb: **할인 정책을 바꾸는 순간 서비스 계층의 참조 객체를 변경 하며 코드가 수정된다. 이 때 OCP 또한 위반된다.**
{% endhint %}



#### 해결방안

***

{% tabs %}
{% tab title="OrderServiceImpl.java" %}
```java
package hello.core.order;

import hello.core.discount.DiscountPolicy;
import hello.core.member.Member;
import hello.core.member.MemberRepository;
import hello.core.member.MemoryMemberRepository;

public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository = new MemoryMemberRepository();
//    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
//    private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
    private DiscountPolicy discountPolicy;


    @Override
    public Order createOrder(Long memberId, String itemName, int itemPrice) {
        Member member = memberRepository.findById(memberId);
        int discountPrice = discountPolicy.discount(member, itemPrice);

        return new Order(memberId, itemName, itemPrice, discountPrice);
    }
}
```
{% endtab %}
{% endtabs %}

구체를 직접적으로 참조하고 있던 값을 제거하고, 오로지 인터페이스만 의존하도록 코드를 변경하였다.

이렇게 설계된 코드가 위에서 말했던 문제점을 상쇄시킬 수 있다. 하지만, 이 코드에서 해당 인터페이스에 대한 구현체를 할당하지 않으면 이 코드는 NPE 가 발생한다.

> :white\_check\_mark: 이 문제를 해결하려면, 누군가 클라이언트인 OrderServiceImpl 에 DiscountPolicy 의 구현 객체를 대신 생성하고 주입 해주어야한다.

