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


{% endstep %}
{% endstepper %}



