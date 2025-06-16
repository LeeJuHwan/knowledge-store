---
description: Web(Controller) 계층 테스트하기
---

# Presentation Layer



<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
#### Prensentation Layer

* 외부 사용자의 요청을 가장 먼저 받는 계층
* 외부 사용자에게 필요한 정보에 대해 최소한의 검증을 수행
{% endhint %}

외부 사용자가 의도한 대로 데이터를 보냈을 시 어플리케이션은 내부적으로 어떻게 처리할 것인가를 해당 레이어에서 명시하게 되며, 이 때 주로 데이터가 처리 되는 부분을 "Mocking" 해서 테스트를 진행한다.



### Mock

***

잘 동작한다고 가정하고, 실제 사용하는 코드의 행동이 아닌 그 행동을 구사하는 척 하는 가짜 객체를 의미함

> #### MockMvc
>
> #### Mock 객체를 사용해 스프링 MVC 동작을 재현할 수 있는 테스트 프레임워크



#### 테스트 어노테이션

***

* SpringBootTest
* DataJpaTest
* WebMvcTest



#### 트랜잭션

***

* Transactional
* Transactional(read\_only)



#### 동시성 제어

***

* 낙관전 락
* 비관적 락



#### 예외

***

* rest controller advice
* excpetion handler



#### 컨트롤러 유효성 검증

***

* not null
* not empty
* not blank





#### 레이어간 의존성 최소화

***

* DTO 분리





#### 레이어드 아키텍처의 단점

***

{% tabs %}
{% tab title="Order.java" %}
```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Table(name = "orders")
@Entity
public class Order extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Enumerated(EnumType.STRING)
    private OrderStatus orderStatus;

    private int totalPrice;

    private LocalDateTime registeredDateTime;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderProduct> orderProducts = new ArrayList<>();
    
    ...
}
```
{% endtab %}

{% tab title="OrderRepository.java" %}
```javascript
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {

}
```
{% endtab %}
{% endtabs %}

도메인 객체가 데이터베이스(JPA)와 너무 강결합 되어 있는 관계로 사용 되고 있다.

또한, Order에 대한 실제 DB 접근을 위한 레포지토리를 작성 하더라도 어쩔 수 없이 JpaRepository 를 구현하며 JPA와 강결합 구조를 띄고 있다.

이러한 특성은 실제 JPA 가 아닌 다른 인프라 레이러를 쓰려고 할 때 이미 강결합 된 구조에서 프로젝트 규모가 커져서, 레이어 하나를 바꾸는 데 투입되는 비용이 너무 많아진다.

이런 단점이 대두 되다 보니 업계에서는 더 나은 구조로 "헥사고날 아키텍처"를 많이 언급한다.



> 헥사고날 아키텍처

<figure><img src="../../../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

도메인 모델에 접근하기 위해서 어댑터와 포트를 활용하고, 도메인 모델은 외부 요소들을 전혀 알 필요가 없다.

이 아키텍처는 DI 의 개념을 확장시킨 구조로, 위 레이어드 아키텍처에서 발생했던 문제 중 하나인 `DomainRepository` 인터페이스를 하나 두고, 그 구현체인 `JpaRepository` 를 사용하게 되면 추후 `MongoRepository` 가 등장 되더라도 `DomainRepository`만 구현하면 된다.







