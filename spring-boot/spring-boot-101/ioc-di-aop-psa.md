# 스프링의 3가지 핵심 개념

IoC/DI, AOP, PSA 는 스프링 프레임워크의 3대 모델이며, 스프링의 핵심이다.

스프링은 객체 지향 설계를 잘 활용하는 프레임워크로, 프레임워크의 중심은 항상 객체에 있다. 그렇기에 객체를 "잘" 관리하는 방법에 대한 가이드를 제공한다.

그 중 3대 모델을 알기 전 알아야 할 것은 [**스프링 컨테이너**](../spring-core-basic/spring-core/스프링_컨테이너와_스프링_빈.md)이다. 스프링 프레임워크가 객체를 "잘" 관리하는 방법 중 하나는 컨테이너에서 객체의 라이프 사이클을 관리하고, 의존 관계 주입을 동적으로 활용하기 때문이다.

### IoC/DI

객체의 생명주기와 의존 관계에 대한 프로그래밍 모델이다.

#### 정의

***

{% hint style="info" %}
#### Depndency Injection

객체 내부에서 다른 객체를 참조하는 경우 의존성이 존재한다 라고 표현하며, 객체 스스로 의존성을 관리하지 않고 외부에서 주입

하는 것을 의존 관계 주입이라고 한다.
{% endhint %}

{% hint style="info" %}
#### Inversion Of Control

실행 흐름의 제어권을 외부에 위임하여, 핵심 로직이 외부에서 주입받은 의존성으로 동작하게 하는 원칙이다.

자세한 설명은 SOLID 원칙에서 DIP 를 설명할 때 상세하게 다루었다.
{% endhint %}

{% content-ref url="https://app.gitbook.com/s/VJhVCjGzSpinr6v1FHUo/undefined/5-.-5-solid#undefined-6" %}
[5장. 객체 지향 설계 5원칙 - SOLID #의존성 역전 원칙은, 왜 의존성을 역전 시킬까?](https://app.gitbook.com/s/VJhVCjGzSpinr6v1FHUo/undefined/5-.-5-solid#undefined-6)
{% endcontent-ref %}

#### 예시

***

최근에 갑작스런 첫 눈으로 인해 빙판위에 차 들이 많이 미끄러지는 사고가 발생했었는데, 이 때를 떠올리며 자동차와 타이어의 관계를 통해 의존 관계 주입과 제어의 역전을 알아보도록 하자.

> 의존 관계 주입
>
> 1. 생성자 주입
> 2. setter 메서드 주입
> 3. field 주입

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

{% tabs %}
{% tab title="생성자 주입" %}
```java
@Component
public class Car {

    private final Tire tire;

    @Autowired
    public Car(Tire tire) {
        this.tire = tire;
    }
}
```

* 의존 관계를 주입하는 객체의 속성 값에 불변을 적용할 수 있다. 이는 객체의 생성 시점에 최초 주입된 객체가 변하지 않도록 강제한다.
* 테스트할 때 의존 관계를 자유롭게 변경할 수 있다.
{% endtab %}

{% tab title="setter 메서드 주입" %}
```java
@Component
public class Car {

    private Tire tire;

    public String getTireInfo() {
        return "현재 장착된 타이어: " + tire.getInfo();
    }

    public Tire getTire() {
        return tire;
    }

    @Autowired
    public void setTire(Tire tire) {
        this.tire = tire;
    }

    @Override
    public String toString() {
        return "Car{" +
                "tire=" + tire +
                '}';
    }
}
```

* `setter` 메서드에 `@Autowired` 어노테이션을 정의하면 의존성을 주입받을 수 있다.
* 생성자가 아닌 메서드로 객체의 의존 관계를 주입해야 하는 경우, 즉 동작중인 코드에서 강제로 내부 객체를 변경할 때 유용하다.
* `Bean Lite Mode` 로 생성된 구성 정보를 사용하더라도 의존하는 객체의 생성은 싱글턴이 사용된다.
{% endtab %}

{% tab title="field 주입" %}
```java
@Component
public class Car {

    @Autowired
    private Tire tire;

    public String getTireInfo() {
        return "현재 장착된 타이어: " + tire.getInfo();
    }

    public Tire getTire() {
        return tire;
    }

    public void setTire(Tire tire) {
        this.tire = tire;
    }

    @Override
    public String toString() {
        return "Car{" +
                "tire=" + tire +
                '}';
    }
}
```

* 결합도가 굉장히 높아져, 테스트 하는 데 의존 객체를 마음대로 관리할 수 없다.
* 프레임워크 구성 정보에 의존하여 런타임에 의존 관계를 주입받게 된다.
{% endtab %}

{% tab title="학습테스트" %}
```java
class CarTest {

    @DisplayName("생성자를 통해 자동차의 타이어를 갈아끼울 수 있다.")
    @Test
    void constructor() {
        // Arrange
        Tire allWeatherTire = new AllWeatherTire();
        Car allWeatherTireCar = new Car(allWeatherTire);

        Tire winterTire = new WinterTire();
        Car winterTireCar = new Car(winterTire);

        // Act
        String allWeatherTireInfo = allWeatherTireCar.getTireInfo();
        String winterTireInfo = winterTireCar.getTireInfo();

        // Assert
        assertThat(allWeatherTireInfo).isEqualTo("현재 장착된 타이어: 사계절 타이어");
        assertThat(winterTireInfo).isEqualTo("현재 장착된 타이어: 겨울 타이어");
    }

    @DisplayName("속성 접근자 메서드를 통해 자동차의 타이어를 갈아끼울 수 있다.")
    @Test
    void set() {
        // Arrange
        Tire allWeatherTire = new AllWeatherTire();
        Car car = new Car(allWeatherTire);

        // Act
        String allWeatherTireInfo = car.getTireInfo();

        // Assert
        assertThat(allWeatherTireInfo).isEqualTo("현재 장착된 타이어: 사계절 타이어");

        // Arrange
        Tire winterTire = new WinterTire();
        car.setTire(winterTire);

        // Act
        String winterTireInfo = car.getTireInfo();

        assertThat(winterTireInfo).isEqualTo("현재 장착된 타이어: 겨울 타이어");
    }

    @DisplayName("스프링의 도움을 받아 구성 정보 클래스와 스프링 컨테이너를 이용하여 자동차의 타이어를 설정할 수 있다.")
    @Test
    void usingConfiguration() {
        // Arrange
        ApplicationContext context = new AnnotationConfigApplicationContext(CarConfig.class);
        Car car = context.getBean(Car.class);

        // Act
        String tireInfo = car.getTireInfo();

        // Assert
        assertThat(tireInfo).isEqualTo("현재 장착된 타이어: 겨울 타이어");
    }

    @DisplayName("스프링의 Autowired 어노테이션을 통해 자동차의 타이어를 결정할 수 있다.")
    @Test
    void usingAutowired() {
        // Arrange
        ApplicationContext context = new AnnotationConfigApplicationContext(CarAutoConfig.class);
        Car car = context.getBean(Car.class);

        // Act
        String tireInfo = car.getTireInfo();

        // Assert
        assertThat(tireInfo).isEqualTo("현재 장착된 타이어: 겨울 타이어");
    }

    @DisplayName("빈으로 등록된 객체는 싱글턴을 보장한다")
    @Test
    void beanSingleton() {
        // Arrange
        ApplicationContext context = new AnnotationConfigApplicationContext(CarConfig.class);

        // Act
        Car car1 = context.getBean(Car.class);
        Car car2 = context.getBean(Car.class);
        // Assert
        assertThat(car1).isSameAs(car2);
    }

    @DisplayName("스프링의 Configuration 어노테이션이 사용된 구성 정보를 이용하면 의존 관계의 객체를 싱글턴으로 반환 받는다")
    @Test
    void dependencySingleton() {
        // Arrange
        ApplicationContext context = new AnnotationConfigApplicationContext(CarAutoConfig.class);
        Car car = context.getBean(Car.class);
        Tire beanRegistrationTire = context.getBean(WinterTire.class);

        // Act
        Tire tire = car.getTire();

        // Assert
        assertThat(beanRegistrationTire).isSameAs(tire);
    }

    @DisplayName("스프링의 Configuration 어노테이션을 사용하지 않은 구성 정보 클래스를 이용하면 의존 관계의 객체를 새로운 참조 주소를 갖는 객체를 반환한다")
    @Test
    void dependencyNotSingleton() {
        // Arrange
        ApplicationContext context = new AnnotationConfigApplicationContext(CarConfig.class);
        Car car = context.getBean(Car.class);
        Tire beanRegistrationTire = context.getBean(WinterTire.class);

        // Act
        Tire tire = car.getTire();

        // Assert
        assertThat(beanRegistrationTire).isNotSameAs(tire);
    }

}
```
{% endtab %}
{% endtabs %}

#### 생성자 주입을 해야 하는 이유

1. setter 메서드를 통한 주입은 논리적 흐름 중간에 언제든 변경이 될 수 있으며, 객체간 순환 참조를 방지할 수 없다.
2. field 주입은 프레임워크 구성 정보에 의존하여 테스트 환경에서 객체를 자유롭게 변경할 수 없다.
3. 생성자 주입을 활용하면 객체의 불변성, 테스트 코드 시 객체 변경의 자유로움, 순환 참조를 방지하는 이점을 이용할 수 있게된다.

{% hint style="warning" %}
#### 순환 참조가 발생하면 왜 안될까?

서로 계속적으로 호출하게 되면, 메모리에 함수의 `CallStack`이 계속 쌓여 `StackOverflow` 에러가 발생하게 된다.

이를 컴파일 시점에 탐지하지 못한 채 운영 환경에서 서버를 구동하게 되면 서버가 갑작스레 중단되기 때문이다.
{% endhint %}

{% hint style="warning" %}
#### setter 나 필드 주입에서는 어떻게 순환 참조가 발생하게 되는걸까?

`setter` 메서드나 `field` 를 통한 주입은 항상 `@Autowired` 어노테이션을 같이 사용한다.

그럼, 객체를 빈에 등록할 때는 문제 없이 등록이 되지만 실제 의존 관계를 연결해야 하는 시점에서 순환 참조가 발생하게 되는 구조이다.

반면에, 생성자를 통한 주입은 빈에 객체를 등록하는 시점에 의존 관계를 같이 연결하기 때문에 빈 등록 과정에서 순환 참조가 발생했다는 것을 미리 알 수 있어 컴파일 단계에서 에러를 발생 시키게된다.

* 이는 스프링 부트 2.6 버전에서 부터 순환 참조를 기본적으로 허용하지 않기 때문에 로딩 시점에 에러가 발생한다.
{% endhint %}



> 의존 관계 역전

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

{% tabs %}
{% tab title="스프링 컨테이너 없이" %}
<pre class="language-java"><code class="lang-java">class CarTest {

    @DisplayName("생성자를 통해 자동차의 타이어를 갈아끼울 수 있다.")
    @Test
    void constructor() {
        // Arrange
        Tire allWeatherTire = new AllWeatherTire();
<strong>        Car allWeatherTireCar = new Car(allWeatherTire);
</strong>
        Tire winterTire = new WinterTire();
<strong>        Car winterTireCar = new Car(winterTire);
</strong>
        // Act
        String allWeatherTireInfo = allWeatherTireCar.getTireInfo();
        String winterTireInfo = winterTireCar.getTireInfo();

        // Assert
        assertThat(allWeatherTireInfo).isEqualTo("현재 장착된 타이어: 사계절 타이어");
        assertThat(winterTireInfo).isEqualTo("현재 장착된 타이어: 겨울 타이어");
    }
</code></pre>
{% endtab %}

{% tab title="스프링 컨테이너 사용" %}
```java
    @DisplayName("스프링의 도움을 받아 구성 정보 클래스와 스프링 컨테이너를 이용하여 자동차의 타이어를 설정할 수 있다.")
    @Test
    void usingConfiguration() {
        // Arrange
        ApplicationContext context = new AnnotationConfigApplicationContext(CarConfig.class);
        Car car = context.getBean(Car.class);

        // Act
        String tireInfo = car.getTireInfo();

        // Assert
        assertThat(tireInfo).isEqualTo("현재 장착된 타이어: 겨울 타이어");
    }
```
{% endtab %}

{% tab title="스프링 구성 정보" %}
```java
public class CarConfig {

    @Bean
    public Car car() {
        return new Car(winterTire());
    }

    @Bean
    public Tire winterTire() {
        return new  WinterTire();
    }

    @Bean
    public Tire allWeatherTire() {
        return new AllWeatherTire();
    }
}
```
{% endtab %}
{% endtabs %}

위 "스프링 컨테이너 없이" 코드를 보면 의존 관계를 주입하고 있지만, 새로운 객체를 주입하기 위해 핵심 로직이 변경되는 것을 볼 수 있다.

이러한 코드는 개발자가 객체의 생성과 주입 모두 직접 제어하기 때문에 OCP 원칙을 지키고자 다형성을 활용 했다고 하더라도 코드의 변경이 일어날 수 밖에 없는 구조이다.

여기서, 변경 되는 부분은 하이라이팅 되어 있듯 타이어를 넘겨주는 객체인데 이 제어권을 외부에 넘기게 되면 핵심 로직의 변경 없이 타이어를 교체할 수 있게 된다.

이로써 눈이 오는 날 부랴부랴 핵심 로직에서 타이어를 갈아끼우지 않고, 우린 구성 정보만 수정하면 자동으로 윈터 타이어를 끼는 마법을 부릴 수 있게 된다.



### AOP

애플리케이션에 공통적으로 나타나는 부가적인 기능들을 독립적으로 모듈화하는 프로그래밍 모델이다.

***

#### 정의

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
#### AOP

Aspect Oriented Programming 의 약어이다. 이는 관점 지향 프로그래밍이라는 뜻으로, 애플리케이션의 여러 로직에 공통적으로 들어가는 부가 기능을 비즈니스 로직으로 부터 분리하여 횡단 관심사라는 단위로 모듈화하여 재사용하는 프로그래밍 패러다임이다.
{% endhint %}

횡단 관심사는 주로 비즈니스 로직과 거리가 멀지만 공통적으로 필요한 부분에 속하며, 로깅, 트랜잭션, 보안 등 다양한 관심사를 처리할 수 있다.

트랜잭션 로직과 같이 삽입되는 부가 기능에 대한 로직을 `Advice`, 그리고 로직이 삽입될 함수를 `JoinPoint`라고 정의한다.

#### AOP 가 동작하는 원리

***

스프링 프레임워크에서 제공하는 AOP 는 [프록시](proxy.md#undefined-2) 방식을 사용한다. 프록시 방식을 사용하면 얻을 수 있는 이점은 사용자가 정의한 로직을 수행하기 전 흐름을 제어할 수 있다.

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

{% stepper %}
{% step %}
`DefaultAopProxyFactory` 를 통해 프록시 객체를 결정하고,  `CglibApoProxy` 가 요청을 처리한다.
{% endstep %}

{% step %}
`AdviseSupport` 를 통해 `Target` 에 대한 정보를 가져온다.
{% endstep %}

{% step %}
어떤 객체, 어떤 메서드를 실행해야 할 정보를 얻은 뒤 `ReflectiveMethodInvocation` 을 통해 메서드를 실행할 준비를 한다.
{% endstep %}

{% step %}
최종적으로 메서드를 실행 시키기 위해 슈퍼 클래스를 상속 받은 뒤 서브 클래스를 생성한다.
{% endstep %}
{% endstepper %}

이 동작 과정에서 알 수 있는 정보는 JDK 동적 프록시와 CGLIB 을 상황에 맞게 적용하여 최적화 했다는 점이다.

그리고 최종적으로 생성 되는 프록시 객체는 사용자가 정의 한 클래스를 슈퍼 클래스로 상속하여 생성 된다.

하지만, 빈에 객체를 등록할 때 항상 CGLIB 프록시 객체가 등록되는 것은 아니었다.

> 프록시 객체가 빈에 등록되는 기준

1. `@Aspect` 어노테이션을 사용한 클래스가 스프링 컨테이너에 등록 했을 때
2. 클래스 레벨 또는 메서드 레벨에 `@Transactional` 어노테이션을 사용했을 때



### PSA

***

스프링은 추상화 레벨을 높여 사용자에게 저수준 API 를 직접 제어하지 않고 편리하게 사용할 수 있도록 기능을 제공한다.

가령 위에서 살펴 보았듯 `@Transactional` 같은 어노테이션을 사용하면 저수준의 커밋과 롤백에 대한 흐름을 신경쓰지 않고 기능을 개발할 수 있다.

추가로, JPA 같은 데이터베이스 접근 기술을 살펴 보면 MySQL, Oracle, PostreSQL 등 각 데이터베이스 마다 갖고 있는 성질과 미세하게 다른 쿼리를 추상화 하여 쉽게 사용할 수 있도록 제공하는 모델이다.

이로인해 추상화 레벨을 높이는 목적으로 내부 코드를 살펴보면 프록시 패턴과 같은 디자인 패턴이 다양하게 사용되는 것을 알 수 있다.



<details>

<summary>참고 자료</summary>

* [https://jojoldu.tistory.com/71?category=635883](https://jojoldu.tistory.com/71?category=635883)
* [https://tech.kakaopay.com/post/overcome-spring-aop-with-kotlin/](https://tech.kakaopay.com/post/overcome-spring-aop-with-kotlin/)

</details>

