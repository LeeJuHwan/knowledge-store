---
description: Web(Controller) 계층 테스트하기
---

# Presentation Layer

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
**Prensentation Layer**

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

**웹 계층에서 테스트 관점**

* 외부 사용자가 입력한 값을 검증하여 정확히 서비스 계층으로 내려주고 있는지
* 서비스 계층에서 정상적인 답변을 받았다면 외부 사용자에게 응답은 하고 있는지

#### Stubbing

***

Mock 객체는 반환 하는 값에 초점을 두지 않고, "행위에 대해 수행하는 척" 했지만 Stub 객체는 행위 이후의 변하는 상태 값 즉, 상태에 대한 검증을 이루는 객체이다.

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

Mock 과 Stub 의 자세한 정의와 둘의 차이는 [#test-double](mock.md#test-double "mention") 에서 상세하게 설명하니, 해당 테스트에선 예시 코드만 다루거 넘어간다.

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

<i class="fa-message-exclamation">:message-exclamation:</i> **계층 의존관계**

* \[단방향] Controller -> Service -> Domain -> Repository
* \[양방향] Controller <-> Service -> Domain -> Repository

<i class="fa-triangle-exclamation">:triangle-exclamation:</i> **양방향 구조의 단점**

컨트롤러에서 서비스 계층으로 처리해야 할 데이터를 넘길 때 서비스는 이미 컨트롤러의 모든 속성을 알고 있다.

컨트롤러 계층에서 외부 사용자에게 입력 받을 데이터가 늘어난다고 했을 때, 서비스 계층은 불필요한 데이터를 같이 갖고 있는 샘이 된다.

이는 결코, 특정 계층이 원하는 값을 받기 위해선 의존하고 있는 계층의 입력 값을 추가로 받아서 전달해야 하는 이 무모한 강결합을 어떻게 해서든 깨트려야한다.

{% embed url="https://techblog.woowahan.com/2711/" %}

해당 글의 "_**Controller와 Service 레이어의 강한 결합**_**"** 챕터에서 나와있듯이 규모가 커지면 커질수록 계층간 요구하는 형식은 다른 방향으로 흘러갈 때가 많다.

중복 코드가 껄끄럽게 느껴지는 것 또한 이해하는 입장으로 더 이상 확장되지 않고 폐쇄적인 구조를 유지하는 프로젝트에서는 계층간 DTO 공유도 하나의 방법이 될 뿐더러, 훨씬 간편하게 개발할 수 있다.

하지만, 코드는 최대한 단순하게 작성 하고 남이 이해하기 쉽도록 작성하는 것이 좋다.

마치, 침팬지에게 단순노동을 알려주듯 서비스 계층에서 받는 요청 DTO 값이 들어오면 어떠한 행동을 할지만 알려주자.

"_더이상 컨트롤러 계층에서 추가적인 데이터가 들어왔을 때 이 데이터는 필요 없으니 얘만 사용해서 처리해_." 라는 복잡한 내용을 침팬지에게 가르치지 말자.

침팬지가 전지전능 해지는 순간 굉장히 많은 지식을 습득하여 미래의 소프트웨어를 공격하는 날이 올 것이다.

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
