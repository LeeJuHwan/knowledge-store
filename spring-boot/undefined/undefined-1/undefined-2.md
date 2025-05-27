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
**스프링 컨테이너 생성**

<figure><img src="../../../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

스프링 컨테이너를 사용하기 위해 인스턴스를 생성할 때 인자로 구성 정보를 지정해주어야 한다. 위 에서는 `AppConfig` 가 이에 해당하며, 이렇게 넘긴 인자는 별도의 "스프링 빈 저장소" 가 생성되게 되고, 설정 구성 정보를 관리하게 된다.
{% endstep %}

{% step %}
**스프링 빈 등록**

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

스프링 빈을 등록 하는 과정은 `@Bean` 이라는 어노테이션을 사용한 메서드는 모두 스프링 빈으로 저장된다. 이 때, 스프링 빈 저장소에 빈 이름으로 메서드 이름이 키 값으로 저장 되며, 인스턴스 객체를 밸류 값으로 저장한다.

만약, 빈 메서드 이름을 의도적으로 사용하고 싶다면 `@Bean(name="customBeanName")` 이런식으로 사용할 수 있다.

{% hint style="danger" %}
**스프링 빈 이름은 항상 다른 이름을 부여 해야한다**. 같은 이름을 부여하면, 설정이 덮어씌워지거나 오류가 발생할 수 있다.

이 원리는 `HashMap` 자료 구조 원리를 생각하면 빈의 이름이 같을 때 왜 무시하는 상황이 발생하는지 이해할 수 있다.
{% endhint %}


{% endstep %}

{% step %}
**스프링 빈 의존관계 설정**&#x20;

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

스프링 빈은 라이프 사이클에 의해 빈을 먼저 생성하고, 의존 관계를 설정하는 두 단계로 나누어서 진행한다.&#x20;

지금 순수 자바 코드(AppConfig)에서는 메서드를 호출하면서 자연스레 의존성이 주입 되었지만 스프링 부트 프레임워크에서 의존관계 자동 주입 이라는 개념을 차용하기 때문에 이 내용이 단계별로 언급이 되었다.
{% endstep %}
{% endstepper %}



