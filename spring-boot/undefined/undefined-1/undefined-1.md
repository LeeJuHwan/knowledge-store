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
