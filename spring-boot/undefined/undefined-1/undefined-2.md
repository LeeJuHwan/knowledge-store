# 스프링 컨테이너와 스프링 빈

### 스프링 컨테이너 생성

***

```java
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
```

{% hint style="info" %}
`ApplicationContext`를 스프링 컨테이너라고 부르며, 이는 인터페이스이다.

스프링에서 XML 기반, 어노테이션 기반 등으로 자바 설정 클래스를 만들 수 있으며 현재는 XML 보다 어노테이션 방식이 훨씬 편리하기 때문에 더 많이 쓰인다.

* 위 코드로 인해 생성된 스프링 컨테이너는 `ApplicationContext` 의 구현체이다.

> 참고
>
> 스프링 컨테이너를 부를 때 `BeanFactory`, `ApplicationContext` 를 구분해서 이야기 했는데, `BeanFactory`를 직접 사용하는 경우는 거의 없기 때문에 일반적으로 `ApplicationContext`를 스프링 컨테이너라고 이해하면 된다.
{% endhint %}



#### 스프링 컨테이너의 생성 과정

{% stepper %}
{% step %}
#### **스프링 컨테이너 생성**

<figure><img src="../../../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

스프링 컨테이너를 사용하기 위해 인스턴스를 생성할 때 인자로 구성 정보를 지정해주어야 한다. 위 에서는 `AppConfig` 가 이에 해당하며, 이렇게 넘긴 인자는 별도의 "스프링 빈 저장소" 가 생성되게 되고, 설정 구성 정보를 관리하게 된다.
{% endstep %}

{% step %}
**스프링 빈 등록**

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

스프링 빈을 등록 하는 과정은 `@Bean` 이라는 어노테이션을 사용한 메서드는 모두 스프링 빈으로 저장된다. 이 때, 스프링 빈 저장소에 빈 이름으로 메서드 이름이 키 값으로 저장 되며, 인스턴스 객체를 밸류 값으로 저장한다.

만약, 빈 메서드 이름을 의도적으로 사용하고 싶다면 `@Bean(name="customBeanName")` 이런식으로 사용할 수 있다.

{% hint style="danger" %}
**스프링 빈 이름은 항상 다른 이름을 부여 해야한다**. 같은 이름을 부여하면, 설정이 덮어씌워지거나 오류가 발생할 수 있다.

이 원리는 `HashMap` 자료 구조 원리를 생각하면 빈의 이름이 같을 때 왜 무시하는 상황이 발생하는지 이해할 수 있다.
{% endhint %}


{% endstep %}

{% step %}
**스프링 빈 의존관계 설정**&#x20;

<figure><img src="../../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

스프링 빈은 라이프 사이클에 의해 빈을 먼저 생성하고, 의존 관계를 설정하는 두 단계로 나누어서 진행한다.&#x20;

지금 순수 자바 코드(AppConfig)에서는 메서드를 호출하면서 자연스레 의존성이 주입 되었지만 스프링 부트 프레임워크에서 의존관계 자동 주입 이라는 개념을 차용하기 때문에 이 내용이 단계별로 언급이 되었다.
{% endstep %}
{% endstepper %}

### 스프링 빈

***

개발자가 코드 상에서 빈을 직접 등록 하여 프로젝트에서 의존성을 주입 받아 쉽게 사용할 수 있는 컨테이너를 직접 "의도한대로 잘 등록 되었는지" 확인을 해본다.



> #### 스프링 빈 조회하기

모든 스프링 컨테이너에 등록된 빈을 조회 하면 스프링부트가 프레임워크를 제어하는데 필요한 빈 까지 같이 조회되게 된다.

기본적으로 애플리케이션에 등록된 빈만 출력해서 확인하면 실제 내가 직접 컨테이너에 등록 해둔 빈 목록만 확인할 수 있다.

{% tabs %}
{% tab title="모든 빈 출력하기" %}
```java
package hello.core.beanfind;

import hello.core.AppConfig;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

class ApplicationContextInfoTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("모든 빈 출력하기")
    void findAllBean() {
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();

        for (String beanDefinitionName : beanDefinitionNames) {
            Object bean = ac.getBean(beanDefinitionName);
            System.out.println("name = " + beanDefinitionName + " object = " + bean);
        }
    }
}
```
{% endtab %}

{% tab title="애플리케이션 빈 출력하기" %}
```javascript
package hello.core.beanfind;

import hello.core.AppConfig;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

class ApplicationContextInfoTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("애플리케이션 빈 출력하기")
    void findApplicationBean() {
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();

        for (String beanDefinitionName : beanDefinitionNames) {
            BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);

            if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
                Object bean = ac.getBean(beanDefinitionName);
                System.out.println("name = " + beanDefinitionName + " object = " + bean);
            }

        }
    }
}
```
{% endtab %}
{% endtabs %}



모든 빈을 확인해야 할 이유는 굉장히 드물 것이다. 프로젝트 규모가 커지면 커질수록 등록된 빈이 엄청 많아지게 될테니까, 그럴땐 특정 빈만 조회 하여 정보를 얻어야한다.

> 가장 기본적인 메서드를 이용하여 빈을 찾는 방법
>
> 만약 조회 하는 빈이 등록되어 있지 않다면 `NoSuchBeanDefinitionException` 에러가 발생한다.

{% tabs %}
{% tab title="기본적인 빈 조회 방법" %}
```java
package hello.core.beanfind;

import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;

import hello.core.AppConfig;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoSuchBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

class ApplicationContextBasicFindTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("빈 이름으로 조회")
    void findBeanByName() {
        MemberService memberService = ac.getBean("memberService", MemberService.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("구체 타입으로 조회")
    void findBeanByName2() {
        MemberService memberService = ac.getBean("memberService", MemberServiceImpl.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("빈 이름 없이 타입으로만 조회")
    void findBeanByType() {
        MemberService memberService = ac.getBean(MemberService.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("빈 이름으로 조회 X")
    void findBeanByNameX() {
        assertThatThrownBy(() -> {
            ac.getBean("asdfg", MemberService.class);
        }).isInstanceOf(NoSuchBeanDefinitionException.class);
    }
}
```
{% endtab %}
{% endtabs %}



> 만약 등록된 빈에 동일한 타입이 두 개 이상 있다면?
>
> 빈에 등록된 객체가 고유하지 않다면 의존 관계를 주입하는데 문제가 생길 수 있다. 그렇기 때문에 빈을 조회 했을 때 두 개 이상이 나온다면 `NoUniqueBeanDefinitionException` 에러가 발생한다.

{% tabs %}
{% tab title="동일한 타입이 두 개 이상 있을 때 빈 조회하기" %}
```java
package hello.core.beanfind;

import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;

import hello.core.member.MemberRepository;
import hello.core.member.MemoryMemberRepository;
import java.util.Map;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoUniqueBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

class ApplicationContextSameBeanFindTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);

    @Test
    @DisplayName("타입으로 조회시 같은 타입이 둘 이상 있으면 중복 오류가 발생한다")
    void findBeanByTypeDuplicate() {
        assertThatThrownBy(() -> {
            ac.getBean(MemberRepository.class);
        }).isInstanceOf(NoUniqueBeanDefinitionException.class);
    }

    @Test
    @DisplayName("타입으로 조회시 같은 타입이 둘 이상 있으면, 빈 이름을 지정하면 된다")
    void findBeanByName() {
        MemberRepository memberRepository1 = ac.getBean("memberRepository1", MemberRepository.class);
        assertThat(memberRepository1).isInstanceOf(MemberRepository.class);
    }

    @Test
    @DisplayName("특정 타입을 모두 조회하기")
    void findAllBeanByTpe() {
        Map<String, MemberRepository> beansOfType = ac.getBeansOfType(MemberRepository.class);

        assertThat(beansOfType).hasSize(2);
    }

    @Configuration
    static class SameBeanConfig {

        @Bean
        public MemberRepository memberRepository1() {
            return new MemoryMemberRepository();
        }

        @Bean
        public MemberRepository memberRepository2() {
            return new MemoryMemberRepository();
        }
    }
}
```
{% endtab %}
{% endtabs %}



