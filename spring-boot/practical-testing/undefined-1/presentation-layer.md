---
description: Web(Controller) 계층 테스트하기
---

# Presentation Layer



<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
#### Prensentation Layer

* 외부 사용자의 요청을 가장 먼저 받는 계층
* 외부 사용자에게 필요한 정보에 대해 최소한의 검증을 수행
{% endhint %}

외부 사용자가 의도한 대로 데이터를 보냈을 시 어플리케이션은 내부적으로 어떻게 처리할 것인가를 해당 레이어에서 명시하게 되며, 이 때 주로 데이터가 처리 되는 부분을 "Mocking" 해서 테스트를 진행한다.



#### Mock

***

잘 동작한다고 가정하고, 실제 사용하는 코드의 행동이 아닌 그 행동을 구사하는 척 하는 가짜 객체를 의미함

> MockMvc
>
> Mock 객체를 사용해 스프링 MVC 동작을 재현할 수 있는 테스트 프레임워크



{% tabs %}
{% tab title="MockBean" %}
```java
@WebMvcTest(controllers = OrderController.class)
class OrderControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @MockitoBean
    private OrderService orderService;

    @DisplayName("신규 주문을 등록한다.")
    @Test
    void createOrder() throws Exception {
        // given
        OrderCreateRequest request = OrderCreateRequest.builder()
                .productNumbers(List.of("001"))
                .build();

        // when // then
        mockMvc.perform(
                        post("/api/v1/orders/new")
                                .content(objectMapper.writeValueAsString(request))
                                .contentType(MediaType.APPLICATION_JSON)
                )
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(jsonPath("$.code").value("200"))
                .andExpect(jsonPath("$.status").value("OK"))
                .andExpect(jsonPath("$.message").value("OK"));

    }
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
위 테스트 코드에서 `MockitoBean` 으로 선언된 `orderService`가 어떠한 행동을 통해 얻은 결과를 검증 할 필요가 없다.&#x20;

단편적인 이유는 `orderService` 에 대한 테스트는 서비스 계층에서 통합 테스트를 모두 끝냈을 것이다.&#x20;

그렇기 때문에 웹 계층에서는 서비스 로직의 반환 값을 알 필요가 없다.

⇒ 이러한 이유로 인해 **Mock** 객체를 활용하게 되며, 서비스 로직의 반환 값을 중점으로 둔 **Stub** 객체가 여기서 사용되면 어색한 이유이기도 하다.
{% endhint %}

**웹 계층에서 테스트 관점**

* 외부 사용자가 입력한 값을 검증하여 정확히 서비스 계층으로 내려주고 있는지
* 서비스 계층에서 정상적인 답변을 받았다면 외부 사용자에게 응답은 하고 있는지



#### Stubbing

***

Mock 객체는 반환 하는 값에 초점을 두지 않고, "행위에 대해 수행하는 척" 했지만 Stub 객체는 행위 이후의 반환 하는 값 즉, 상태에 대한 검증을 이루는 객체이다.



{% tabs %}
{% tab title="Stubbing" %}
```java
    @DisplayName("결제완료 주문들을 조회하여 매출 통계 메일을 전송한다.")
    @Test
    void sendOrderStatisticsMail() {
        // given
        LocalDateTime now = LocalDateTime.of(2023, 3, 5, 0, 0);

        Product product1 = createProduct(HANDMADE, "001", 1000);
        Product product2 = createProduct(HANDMADE, "002", 2000);
        Product product3 = createProduct(HANDMADE, "003", 3000);
        List<Product> products = List.of(product1, product2, product3);
        productRepository.saveAll(products);

        Order order1 = createPaymentCompletedOrder(products, LocalDateTime.of(2023, 3, 4, 23, 59, 59));
        Order order2 = createPaymentCompletedOrder(products, now);
        Order order3 = createPaymentCompletedOrder(products, LocalDateTime.of(2023, 3, 5, 23, 59, 59));
        Order order4 = createPaymentCompletedOrder(products, LocalDateTime.of(2023, 3, 6, 0, 0, 0));

        // stubbing - Mock 객체에 원하는 행위를 정의하는 것
        when(mailSendClient.sendMail(any(String.class), any(String.class), any(String.class), any(String.class)))
                .thenReturn(true);

        // when
        boolean result = orderStatisticsService.sendOrderStatisticsMail(LocalDate.of(2023, 3, 5), "test@test.com");

        // then
        assertThat(result).isTrue();

        List<MailSendHistory> histories = mailSendHistoryRepository.findAll();

        assertThat(histories).hasSize(1)
                .extracting("content")
                .contains("총 매출 합계는 12000 원 입니다.");
    }

```
{% endtab %}
{% endtabs %}



{% hint style="success" %}
#### Mock 과 Stub 의 차이

Mock 은 실제 사용하는 코드의 행동이 아닌 그 행동을 구사하는 척 하는 것 -> 행동 검증 중점

Stub 은 실제 사용하는 코드의 행동에 따른 결과 값을 원하는 값으로 정의 하는 것 -> 상태 검증 중점
{% endhint %}



#### 테스트 어노테이션

***

* SpringBootTest
* DataJpaTest
* WebMvcTest



{% hint style="info" %}
#### 언제 어떤 어노테이션을 사용하여 테스트 해야할까?

<mark style="color:purple;">**SpringBootTest**</mark> - 전체 통합 테스트가 필요할 때, 스프링부트에 사용되는 모든 빈을 컨테이너에 등록 하여 사용하며 빌드 시간이 오래 소요 됨

<mark style="color:purple;">**DataJpaTest**</mark> - Repository 관련 빈만 컨테이너에 등록하며 데이터 접근을 위한 테스트하기 유리함, 단 QueryDSL 을 사용하여 구현체를 사용하는 경우 SpringBootTest 로 통합 테스트를 진행 해야 함

<mark style="color:purple;">WebMvcTest</mark> - RestController 관련 빈만 컨테이너에 등록하며 외부 사용자가 입력하는 값 등을 검증할 때 테스트하기 유리함
{% endhint %}

간략하게 살펴보면 각 어노테이션마다 사용해야 하는 경우가 꽤 다른 편인데, 테스트코드를 작성하다보면 의외로 통합테스트를 많이 요구한다.

하지만, 도메인을 잘 분리하여 도메인의 메서드를 빠르게 테스트할 수 있는 단위 테스트에 집중하는 것이 좋다.



위 어노테이션 별 설명에서 보충 설명이 필요한 부분이 있다.

바로 DataJpaTest 와 SpringBootTest 간의 내용인데, 이 둘의 관계는 트랜잭션에 따라 나뉘게된다.

{% tabs %}
{% tab title="SpringBootTest" %}
<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
{% endtab %}

{% tab title="DataJpaTest" %}
<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>
{% endtab %}
{% endtabs %}

DataJpaTest 는 기본적으로 Transactional 을 지원하여 테스트 컨텍스트 간 데이터 롤백이 자동 지원된다.

반면, SpringBootTest 는 Transactional 이 없기 때문에 사용자가 직접 Teardown Method 를 통해 테스트 케이스가 끝날 때 마다 데이터를 정리하는 작업을 진행해야 한다.

한 눈에 보기엔 당연히 DataJpaTest 를 사용하는게 편하리라 생각할 수 있지만, 사실 자동으로 지원된다는 것은 실제 프로덕션에 배포될 코드에 누락될 수 있는 여지를 주는 것이다.

자동으로 수행하기 때문에 당연시하게 생각하고 넘기는 경우가 생길 수 있으니, 되도록이면 안전하게 테스트 환경에서는 대부분을 하드 코딩 하듯 통합테스트를 사용하고 데이터 클렌징 작업을 손수 하는 것을 권장한다.



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

> DTO 분리

프로젝트 패키지를 구분하다보면 컨트롤러, 서비스 이렇게 각 계층 별로 패키지를 구성하는 경우가 많다.

프로젝트의 아키텍처를 구성하다보면 많은 양의 코드를 수정하지 않고도 요구사항을 반영하길 원한다. 그렇게 레이어드 아키텍처, 클린 아키텍처, 헥사고날 아키텍처 등 다양한 아키텍처가 고안 되었는데 이런 아키텍처를 생각하기 전에 계층간 의존 관계를 단방향으로 만드는 것을 먼저 신경 써 볼 필요가 있다.



계층 의존관계

* \[단방향] Controller -> Service -> Domain -> Repository
* \[양방향] Controller <-> Service -> Domain -> Repository



양방향 구조가 갖고 있는 단점을 먼저 살펴보면, 컨트롤러에서 서비스 계층으로 처리해야 할 데이터를 넘길 때 서비스는 이미 컨트롤러의 모든 속성을 알고 있으며&#x20;







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







