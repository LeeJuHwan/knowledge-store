# 객체 지향 원리 적용

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

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="warning" %}
#### OCP, DIP 같은 객체지향 설계 원칙을 충실히 준수했다고 볼 수 있지만 그렇지 않다.

위 이미지에 나와있듯이 코드에서 `DiscountPolicy` 인터페이스만 참조하고 있다고 알고 있지만, 사실은 인터페이스를 구현한 구현 클래스도 직접 참조하고 있다.



:bulb: **추상도 의존을 하고, 구체도 의존을 하는 관계가 형성 되어 DIP 를 위반하고 있다**

:bulb: **할인 정책을 바꾸는 순간 서비스 계층의 참조 객체를 변경 하며 코드가 수정된다. 이 때 OCP 또한 위반된다.**
{% endhint %}

> _**이러한 설계는 마치 이런 상황과도 같다.**_
>
> "공연" 에서 배역을 맡을 배우를 정하는데, 공연 기획자가 배우를 정하는 것이 아닌 남자 주연 배우가 자신에게 맞는 여자 주연 배우를 고르는 격과 같다.&#x20;
>
> 이러한 문제를 해결하기 위해서 남자 주연 배우와, 여자 주연 배우를 섭외 하는 담당자인 "공연 기획자" 즉, 서비스의 구현 객체가 아닌 다른 객체가 이 책임을 맡아야 한다.



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



**관심사 분리 - App Config 등장**

구현 객체를 생성하고, 연결 하는 책임을 갖는 별도의 설정 클래스를 만들어서 구현체가 인터페이스만 의존할 수 있도록 문제를 해결 해보자.



<figure><img src="../../../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

{% tabs %}
{% tab title="AppConfig.java" %}
```java
package hello.core;

import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.member.MemberRepository;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import hello.core.member.MemoryMemberRepository;
import hello.core.order.OrderService;
import hello.core.order.OrderServiceImpl;

public class AppConfig {

    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    public DiscountPolicy discountPolicy() {
        return new FixDiscountPolicy();
    }
}

```
{% endtab %}

{% tab title="OrderServiceImpl.java" %}
```java
package hello.core.order;

import hello.core.discount.DiscountPolicy;
import hello.core.member.Member;
import hello.core.member.MemberRepository;

public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }

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

더 이상 `OrderServiceImpl` 구현체는 스스로 할인 정책의 구현체를 의존하지 않게 되었다. 이로서 얻을 수 있는 이점은 구현체의 소스코드를 수정하지 않고 할인 정책을 바꿀 수 있다는 것이다.

이러한 원리는`DIP` 가 잘 준수 되었다고 보며, 추후 유지보수를 위해 할인 정책을 또 바꿔야한다면 `AppConfig` 에서 할인 정책만 갈아끼우면 된다.&#x20;

<mark style="color:red;">**이로써**</mark><mark style="color:red;">**&#x20;**</mark><mark style="color:red;">**`MemberServiceImpl`**</mark><mark style="color:red;">**&#x20;**</mark><mark style="color:red;">**은 의존관계에 대한 고민은 "외부(**</mark><mark style="color:red;">**`App Config`**</mark><mark style="color:red;">**)" 에게 맡기고 오로지 실행에만 집중하게된다.**</mark>



#### :bulb: 좋은 객체 지향 설계의 5가지 원칙이 어떻게 적용 되었는가?



{% stepper %}
{% step %}
#### SRP&#x20;

**한 클래스는 하나의 책임만 가져야한다.**

`OrderServiceImpl` 이 개선되기 전 상황을 돌이켜보면, `DiscoutPolicy` 인터페이스를 참조하고 있었지만 그와 동시에 `FixDiscountPolicy` 를 같이 참조하며 "인터페이스" 와 "구체"를 동시에 참조하는 상황이 발생했다.

이러한 설계는 할인 정책을 고르는 주체는 `OrderServiceImpl` 이 되기 때문에 여러 책임이 생기게된다.

그래서, 할인 정책에 대한 구현 객체를 생성하고 연결하는 `AppConfig` 에게 관심사를 분리시키고, `OrderServiceImpl` 은 실제 주문과 관련된 기능만 집중하게 될 수 있다.
{% endstep %}

{% step %}
#### DIP

**프로그래머는 추상화에 의존해야지, 구체화에 의존하면 안된다.**

SRP 에서 겪었던 문제와 동일하다.

구현 객체를 생성하고 연결하는 AppConfig 가 생성됨에 따라, OrderServiceImpl 이 필요한 할인정책은 앞으로 AppConfig 내부에서 생성 하여 직접 의존관계를 주입하는 설계를 따랐다.

이 방법의 이점은 당연하게도 클라이언트 코드를 수정하지 않고 요구사항을 반영할 수 있는 것이다.
{% endstep %}

{% step %}
#### OCP

**소프트웨어 요소는 확장은 열려있고 변경은 닫혀있어야한다.**

DIP 규칙에 의해 클라이언트 코드가 변경되지 않고도 할인 정책을 새롭게 확장시킬 수 있었다.
{% endstep %}
{% endstepper %}





### IoC, DI, 컨테이너

***

{% stepper %}
{% step %}
#### 제어의 역전 IoC

개발자 입장에서 자신의 프로그램을 직접 제어하는 것은 당연하다. 위 예제 처럼 `OrderServiceImpl` 을 직접 생성해서, 주문을 만들고 하는 과정은 모두 개발자가 제어하여 논리적 흐름을 만들어낸다.

하지만, `AppConfig` 가 등장하면서 `OrderServiceImpl` 은 개발자가 더이상 제어하지 못하게 된다. 이는, `MemberRepository`, `DiscountPolicy` 라는 두 인터페이스가 `AppConfig` 에 의해 직접 의존 관계를 주입 하기 때문에 어떠한 로직이 실행될지는 `AppConfig` 가 없이 예측하기 어렵다.

또한, `AppConfig` 에서 `orderService` 라는 메서드의 반환 값으로 `OrderServiceImpl` 을 생성 해주었지만, 만약 `OrderLongCustomerService` 라는 장기고객 전용 주문 서비스가 새롭게 개발되어 해당 객체가 `orderService` 메서드로 반환된다면 어떨까? 이 모든 과정은 개발자가 제어하던 프로그램이 `AppConfig` 라는 구성 요소가 제어하게 되며 생긴 제어 방향의 역전이 일어난 것이다.

> _"프레임워크 vs 라이브러리"_
>
> 내가 작성한 코드를 프레임워크가 제어하고, 대신 실행 한다 - 프레임워크
>
> 내가 작성한 코드의 흐름을 내가 직접 제어한다 - 라이브러리


{% endstep %}

{% step %}
#### 의존관계 주입

의존관계는 **정적인 클래스 의존 관계**와, 실행 시점에서 결정 되는 **동적인 객체 의존 관계**를 분리해서 생각해야한다.

> "정적인 클래스 의존관계"

클래스가 사용하는 코드만 보고 의존관계를 쉽게 파악할 수 있는 관계로, 애플리케이션을 직접 실행하지 않아도 어떤 의존관계를 띄고 있는지 분석할 수 있다.

{% tabs %}
{% tab title="예시" %}
```java
package hello.core.order;

import hello.core.discount.DiscountPolicy;
import hello.core.member.Member;
import hello.core.member.MemberRepository;

public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
```
{% endtab %}
{% endtabs %}

위 코드에서 `OrderServiceImpl` 은 `MemberRepository`, `DiscountPolicy` 와 의존관계를 맺고 있다는 것을 굳이 애플리케이션을 실행하지 않고도 알아볼 수 있는 것이 "**정적 의존 관계**" 이다.

> "동적인 객체 인스턴스 의존 관계"
>
> 애플리케이션 실행 시점에 실제 생성된 객체 인스턴스가 연결된 의존 관계로, 실제 외부에서 생성 하여 클라이언트에게 주입 하며 연결 되는 의존 관계이다.

{% tabs %}
{% tab title="실행 시점(런타임) 에서 의존관계 주입" %}
```java
public class AppConfig {

    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }
    
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }

```
{% endtab %}
{% endtabs %}

:bulb: **의존 관계 주입을 사용하게 되면, 실제 정적인 클래스 의존관계(인터페이스 참조 등)을 변경하지 않고도 동적인 객체 인스턴스 의존관계(구현체 생성)를 변경할 수 있다.**
{% endstep %}

{% step %}
#### **DI 컨테이너**

AppConfig 처럼 실행 시점에서 객체를 관리하고 의존성을 주입하는 구성을 IoC 컨테이너 또는 **DI 컨테이너** 라고 표현한다.

과거에는 IoC 컨테이너라고 불렀지만, 시간이 지나며 의존 관계를 주입하는 역할에 맞는 명칭을 사용하기 위해 DI 컨테이너라고 명칭이 생겼고 이런 컨테이너를 부를 땐 DI 컨테이너라고 부르는것이 올바르다.
{% endstep %}
{% endstepper %}



### AppConfig 를 스프링 컨테이너로 전환하기

***

스프링 컨테이너를 사용하기 위해 필요한 객체를 어노테이션을 활용 하여 생성하게 된다. 그렇게 생성 된 구성 정보는 `ApplicationContext` 객체에서 사용할 수 있다.\
아래 어노테이션을 작성한 메서드는 스프링 컨테이너에 등록 되며, 이렇게 등록된 객체를 **스프링 빈** 이라고 한다.

* `@Configuration`
* `@Bean`



#### 코드에 적용 하기

{% tabs %}
{% tab title="MemberApp" %}
```java
package hello.core;

import hello.core.member.Grade;
import hello.core.member.Member;
import hello.core.member.MemberService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class MemberApp {

    public static void main(String[] args) {
//        AppConfig appConfig = new AppConfig();
//        MemberService memberService = appConfig.memberService();

        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService = applicationContext.getBean("memberService", MemberService.class);

        Member member = new Member(1L, "A", Grade.VIP);
        memberService.join(member);

        Member findMember = memberService.findMember(1L);
        System.out.println("new member = " + member.getName());
        System.out.println("findMember = " + findMember.getName());
    }
}
```
{% endtab %}

{% tab title="OrderApp" %}
```java
package hello.core;

import hello.core.member.Grade;
import hello.core.member.Member;
import hello.core.member.MemberService;
import hello.core.order.Order;
import hello.core.order.OrderService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class OrderApp {

    public static void main(String[] args) {
//        AppConfig appConfig = new AppConfig();
//
//        MemberService memberService = appConfig.memberService();
//        OrderService orderService = appConfig.orderService();

        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService = applicationContext.getBean("memberService", MemberService.class);
        OrderService orderService = applicationContext.getBean("orderService", OrderService.class);

        Long memberId = 1L;
        Member member = new Member(memberId, "A", Grade.VIP);
        memberService.join(member);

        Order order = orderService.createOrder(memberId, "itemA", 20000);

        System.out.println("order = " + order);

    }

}
```
{% endtab %}
{% endtabs %}







