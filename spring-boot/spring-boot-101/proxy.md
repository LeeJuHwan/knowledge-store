---
description: 스프링이 제공하는 동적 프록시와 CGLib 기술 이해하기
---

# 프록시 기술

[프록시](https://1eejuhwany.gitbook.io/studylog/booklog/undefined/6-.#undefined-3) 는 호출하고자 하는 행위를 직접 참조하지 않고 중계 역할을 하며, 행위 호출 이전에 흐름을 제어하는 역할을 한다.

스프링에서 이러한 기술을 직접 정의하고 사용하게 되면 불필요한 빈의 등록과 DI 과정에서 의도치 않은 오류를 범할 수 있다.

### 수동으로 프록시를 구현하는 경우

***

이 예시는 [해당 블로그](https://mangkyu.tistory.com/175)에서 참고 하였으며 할인 정보를 제공하는 서비스 계층을 통해 프록시를 직접 구현하는 경우 겪을 수 있는 단점을 살펴본다.

{% tabs %}
{% tab title="DiscountService" %}
```java
public interface DiscountService {

    int discount();

}
```
{% endtab %}

{% tab title="RateDiscountService" %}
```java
@Service
public class RateDiscountService implements DiscountService {

    @Override
    public int discount() {
        return 0;
    }
}
```
{% endtab %}

{% tab title="RateDiscountServiceProxy" %}
```java
@Service
public class RateDiscountServiceProxy implements DiscountService{

    private DiscountService discountService;

    public RateDiscountServiceProxy(DiscountService discountService) {
        this.discountService = discountService;
    }

    @Override
    public int discount() {
        // NOTE: workflow started at

        this.discountService.discount();

        // NOTE: workflow ended at

        return 0;
    }


}
```
{% endtab %}
{% endtabs %}

프록시 예시를 살펴보면 서비스 계층을 구현하는 두 코드에 어노테이션이 붙어있다. 그리고, 동일한 인터페이스를 서로 구현하고 있다.

여기서 알 수 있는 불편사항은 "**동일한 인터페이스를 구현 했기 때문에 DI 과정에서 우선순위를 정해주어야한다.**", "**두 서비스 구현 코드가 빈에 등록된다.**" 이다.

그래서 스프링은 직접 구현하여 프록시에 대한 소스코드 파일이 남지 않고 런타임 중 프록시 객체를 생성하는 동적 프록시 방식을 지원한다.



### 동적 프록시 기술

***

위 직접 구현한 프록시에서 단점들을 여럿 소개 했지만 핵심적인 부분은 서비스 계층을 구현하는 코드가 100개 이상이라면, 프록시 코드도 100개 이상이 등장하게 된다.&#x20;

이 문제를 해결 하는 것이 바로 동적 프록시 기술이며, 이는 개발자가 직접 프록시 클래스를 만들지 않고 런타임에 대신 만들어주는 기술이다.

#### JDK Dynamic Proxy

JDK 동적 프록시 기술은 기본적으로 인터페이스를 기반으로 동작하며, 자바에서 제공하는 `InvocationHandler` 를 구현하여 적용할 수 있다.

> 실행 시간을 측정하는 프록시 예제

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

{% tabs %}
{% tab title="AInterface" %}
```java
package jhlee.springrecipes.proxytest.jdkdynamic;

public interface AInterface {

    String call();
}
```
{% endtab %}

{% tab title="TimeInvocationHandler" %}
```java
package jhlee.springrecipes.proxytest.jdkdynamic;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class TimeInvocationHandler implements InvocationHandler {

    private final Object target;

    public TimeInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = method.invoke(target, args);

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료 resultTime = {}", resultTime);
        return result;
    }
}
```
{% endtab %}

{% tab title="JdkDynamicProxyTest" %}
```java
class JdkDynamicProxyTest {

    @Test
    void dynamicA() {
        // Arrange
        AInterface target = new AImpl();
        TimeInvocationHandler timeInvocationHandler = new TimeInvocationHandler(target);
        AInterface proxy = (AInterface) Proxy.newProxyInstance(AInterface.class.getClassLoader(), new Class[]{AInterface.class}, timeInvocationHandler);
        
        // Act
        proxy.call();

        // Assert
        assertThat(target).isInstanceOf(AInterface.class);
        assertThat(proxy).isInstanceOf(Proxy.class);
    }

    @Test
    void dynamicB() {
        // Arrange
        BInterface target = new BImpl();
        TimeInvocationHandler timeInvocationHandler = new TimeInvocationHandler(target);
        BInterface proxy = (BInterface) Proxy.newProxyInstance(BInterface.class.getClassLoader(), new Class[]{BInterface.class}, timeInvocationHandler);
        
        // Act
        proxy.call();

        // Assert
        assertThat(target).isInstanceOf(BInterface.class);
        assertThat(proxy).isInstanceOf(Proxy.class);
    }
```
{% endtab %}
{% endtabs %}

각 서비스 계층이 구현하고 있는 프록시 객체를 더 이상 1대1 비율로 생성하지 않아도 된다. 공통으로 처리해야 하는 로직을 프록시 구현체로 생성하고, 서비스 코드는 이를 활용하여 타겟으로 넘겨주기만 하면 되는 편리함이 생긴다.

{% hint style="info" %}
#### JDK 동적 프록시의 한계

1. 프록시를 적용하기 위해 반드시 인터페이스를 사용해야 한다.
2. 구현 클래스로 타겟으로 활용할 수 없고, 반드시 인터페이스로만 주입 받아야한다.
{% endhint %}



#### CGLib

기존 JDK 동적 프록시가 갖고 있던 문제 중 반드시 인터페이스를 사용해야 한다는 부분을 해결할 수 있는 기술이다.

{% hint style="info" %}
#### CGLIB - Code Generator Library

CGLIB 는 바이트코드를 조작하여 동적으로 클래스를 생성하는 기술을 제공하는 라이브러리이다.

CGLIB 는 사용자가 정의한 타겟 클래스를 부모 클래스로 만든 뒤 프록시 객체를 생성할 때 타겟 클래스를 상속하여 메서드를 호출한다.
{% endhint %}

> 예제

{% tabs %}
{% tab title="TimeMethodInterceptor" %}
```java
@Slf4j
public class TimeMethodInterceptor implements MethodInterceptor {

    private Object target;

    public TimeMethodInterceptor(Object target) {
        this.target = target;
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = proxy.invoke(target, args);

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료 resultTime = {}", resultTime);
        return result;
    }
}
```
{% endtab %}

{% tab title="CglibTest" %}
```java
@Slf4j
class CglibTest {

    @Test
    void cglib() {
        // Arrange
        ConcreteService target = new ConcreteService();

        // Act
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(ConcreteService.class);
        enhancer.setCallback(new TimeMethodInterceptor(target));
        ConcreteService proxy = (ConcreteService) enhancer.create();

        log.info("targetclass={}", target.getClass());
        log.info("proxyclass={}", proxy.getClass());

        // Act
        proxy.call();

        // Assert
        assertThat(target).isInstanceOf(ConcreteService.class);
        assertThat(proxy).isInstanceOf(ConcreteService.class);

    }

}
// targetclass=class ConcreteService
// proxyclass=class ConcreteService$$EnhancerByCGLIB$$16cc60ac
```
{% endtab %}
{% endtabs %}

CGLIB 기술로 생성한 프록시 객체는 `Enhancer$CGLIB$객체참조 주소`로 생성 되며, 구체 클래스로 타입 검증을 했을 때 통과하는 것을 볼 수 있다.

{% hint style="info" %}
#### CGLIB 의 제약

1. 기본 생성자를 필요로 하며, 원본 타겟 클래스의 생성자 호출, 프록시 객체의 슈퍼 클래스 생성자 호출로 총 2번의 생성자를 호출한다.
2. `final` 로 명시된 클래스 또는 메서드를 사용할 수 없다.
3. 메서드의 접근 제어자는 `private` 을 이용할 수 없다.
{% endhint %}

#### 프록시 기술 정리

|         | JDK 동적 프록시   | CGLIB Proxy |
| ------- | ------------ | ----------- |
| 구현 원리   | 인터페이스 구현     | 슈퍼클래스 상속    |
| 대표적인 제약 | 구체 클래스 주입 불가 | 생성자 두 번 호출  |



### 스프링이 해결한 동적 프록시 기술

***

스프링은 3.2 버전 부터 별도의 CGLIB 라이브러리 추가 없이 사용할 수 있도록 내부에 함께 패키징 되었다.

#### CGLIB 기본 생성자 필수 문제 & 생성자 2번 호출 문제

스프링 4.0 부터 `objenesis` 라이브러리를 사용해서 기본 생성자 없이 객체 생성이 가능하도록 하였다.

* `objenesis`: 생성자 호출 없이 객체를 생성할 수 있는 기술 지원

#### final 키워드 제약 문제

스프링 부트 2.0 버전 부터 CGLIB 을 기본 프록시 기술로 선정 되었으며, 구체 클래스를 의존 관계로 주입하는 문제를 해결하였다.

* 스프링 부트는 별도의 설정이 없다면 AOP 를 적용할 때 기본적으로 `proxyTargetClass=true` 로 설정하여 사용한다.

따라서, 인터페이스가 있더라도 JDK 동적 프록시를 사용하지 않고 항상 CGLIB 을 사용하여 구체 클래스를 기반으로 프록시를 생성한다.

추가적으로 JDK 동적 프록시도 사용할 수 있도록 옵션 값을 설정할 수 있다.



<details>

<summary>참고 자료</summary>

* [https://mangkyu.tistory.com/175](https://mangkyu.tistory.com/175)
* [https://www.inflearn.com/courses/lecture?courseId=327901\&tab=curriculum\&type=LECTURE\&unitId=94541\&subtitleLanguage=ko](https://www.inflearn.com/courses/lecture?courseId=327901\&tab=curriculum\&type=LECTURE\&unitId=94541\&subtitleLanguage=ko)

</details>

