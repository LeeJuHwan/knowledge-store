# 더 나은 테스트를 작성하기 위한 가이드

그레이들의 테스트를 통해 지금까지 작성했던 모든 코드를 돌려보면 스프링 부트가 빌드 되는 시간이 굉장히 오래걸린다는 것을 알 수 있다.

이런 빌드 시간이 긴 어플리케이션을 우리의 통합 테스트 단계에서 여러번 새롭게 빌드 한다면 통합 테스트를 "자주" 실행하기 꺼려지고 매번 단위 테스트만 실행하게 될 것이다.

이로인해 코드 변경점에 대한 사이드 이펙트를 인지하는 시점이 굉장히 늦어지는데, 이 땐 많은 량의 코드를 작성한 이후가 되어 디버깅하기 까다로워진다.

<figure><img src="../../../.gitbook/assets/스크린샷 2025-07-07 오후 12.30.12.png" alt=""><figcaption></figcaption></figure>

현재 테스트코드에서만 보더라도 스프링 부트가 새롭게 빌드 되는 횟수가 총 "6" 번이다. 만약, 한 번에 100ms 가 걸린다고 가정해도 벌써 600ms를 기다려야 한다.

> _**"왜 테스트 코드에서 여러 번의 스프링 부트를 빌드할까?"**_

스프링 부트 테스트를 하다보면 환경이 서로 다 다른 경우가 굉장히 많다. 예를 들어 Service 계층에서는 SpringBootTest 를 진행하고, Domain 계층에서 DataJpaTest 를 진행했을 때 벌써 2개의 환경이 생성된다.

추가적으로 Controller에 WebMvcTest 와 별도로 Mock Bean이 존재한다면 또 테스트 환경이 달라지게된다.

그래서 이 테스트 코드의 환경을 통합하게 되면 여러번 빌드 되었던 스프링 부트가 비교적 적게 호출되는 것을 확인할 수 있다.

> _**"스프링부트 환경이 다른 경우 어떻게 통합할까?"**_

1. MockBean 을 사용하면 환경을 통합해도 새롭게 스프링부트는 다른 환경으로 인식한다.
   * MockBean 을 쓰는 통합환경 상위 클래스 하나 구축하고, MockBean 을 쓰지 않는 통합환경 상위 클래스 하나 구축하여 해결
2. SpringBootTest를 사용하는 환경에서 DataJpaTest 를 쓰는 경우
   * DataJpaTest가 정말 필요한 이점이 없는 경우 SpringBootTest 로 통합한 환경을 사용하고 Transactional 을 사용하거나 DataJpaTest 전용 환경을 하나 따로 구축하여 해결
3. WebMvcTest 를 사용하는 경우
   * 컨트롤러를 위한 서포트 클래스를 생성하지만 컨트롤러 정의를 모두 다 정의함

**테스트 환경 통합하기**

Repository에 대한 테스트도 DataJpaTest가 아닌 SpringBootTest 를 사용하는 것을 추천하는 이유도 이 중 하나이다.

Transactional 어노테이션만 추가하면 DataJpaTest 가 갖고 있는 기능을 누릴 수 있으면서 테스트 환경을 통합하여 전체 테스트를 실행 시키는 횟수를 증가시키는 것이다.

{% tabs %}
{% tab title="Controller" %}
```java
@WebMvcTest(controllers = {
        OrderController.class,
        ProductController.class,
})
public abstract class ControllerTestSupport {

    @Autowired
    protected MockMvc mockMvc;

    @Autowired
    protected ObjectMapper objectMapper;

    @MockitoBean
    protected OrderService orderService;

    @MockitoBean
    protected ProductService productService;


}
```

```java
class OrderControllerTest extends ControllerTestSupport {

    @DisplayName("신규 주문을 등록한다.")
    @Test
    void createOrder() throws Exception {
        // given
    ...
    }
```
{% endtab %}

{% tab title="Service + Domain with Mock" %}
```java
@ActiveProfiles("test")
@SpringBootTest
public abstract class IntegrationTestSupport {

    @MockitoBean
    protected MailSendClient mailSendClient;

}
```
{% endtab %}
{% endtabs %}

<figure><img src="../../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

테스트 환경을 통합하게 되면 이 처럼 6번 스프링 부트를 재실행 했던 통합 테스트와 달리 2번으로 줄어들었다.

개발하면서 사이드 이펙트에 대한 불안감을 안고 코드를 작성하는 것은 눈을 감고 운전하는 것과 같다.

그러니 자주 통합 테스트를 수행 하여 현재 도로위 내 자동차가 안전하게 가고 있는지 혹은 올바른 방향으로 가고 있는지 파악해야한다.

#### Private method도 테스트해야 하는가?

***

"일단, 할 필요 없다" 가 내가 생각하는 바이다. 대신, 프라이빗 메서드에 대한 테스트 필요성을 느꼈다면 객체를 분리해야 할 시점이 왔다는 것은 알 수 있다.

> "왜 프라이빗 메서드는 테스트 할 필요 없을까?"

1. 클라이언트가 호출하는 입장에서 외부에 공개 할 메서드만 공개 하고, 프라이빗 메서드는 공개하지 않는 관점이기 때문에 공개적으로 테스트할 필요가 없음
2. 프라이빗 메서드를 테스트하지 않아도 퍼블릭 메서드를 테스트 하면서 같이 검증이 된다
3. 만약, 프라이빗 메서드가 단위 테스트가 절실하다면 객체를 분리할 시점인지 자문할 상황이 왔다는 것

**예시**

```java
public class ProductService {

    private final ProductRepository productRepository;

    public ProductResponse createProduct(ProductCreateServiceRequest request) {
        // productNumber 부여
        // DB에서 마지막으로 저장된 상품의 상품번호를 읽어와서 1 증가
        String nextProductNumber = createNextProductNumber();

        Product product = request.toEntity(nextProductNumber);
        Product savedProduct = productRepository.save(product);

        return ProductResponse.of(savedProduct);
    }

    private String createNextProductNumber() {
        String latestProductNumber = productRepository.findLatestProductNumber();

        if (latestProductNumber == null) {
            return "001";
        }

        int lastestProductNumberInt = Integer.parseInt(latestProductNumber);
        int nextProductNumberInt = lastestProductNumberInt + 1;

        return String.format("%03d", nextProductNumberInt);
    }

```

**개선**

{% tabs fullWidth="false" %}
{% tab title="ProductNumberFactory" %}
```java
@RequiredArgsConstructor
@Component
public class ProductNumberFactory {

    private final ProductRepository productRepository;

    public String createNextProductNumber() {
        String latestProductNumber = productRepository.findLatestProductNumber();

        if (latestProductNumber == null) {
            return "001";
        }

        int lastestProductNumberInt = Integer.parseInt(latestProductNumber);
        int nextProductNumberInt = lastestProductNumberInt + 1;

        return String.format("%03d", nextProductNumberInt);
    }


}
```
{% endtab %}

{% tab title="ProductService" %}
```java
public class ProductService {

    private final ProductRepository productRepository;
    private final ProductNumberFactory productNumberFactory;

    // 동시성 이슈 -> 무언가 증가하는 로직이 있는 경우 여러명이 동시에 수행하면 어떻게 해결할 것인가?
    // 동시 접속자가 적은 경우 DB field 에 unique Index 걸고, 재시도 하는 로직을 선택할 수도 있음(3회 이상)
    // 동시 접속자가 너무 많다면 UUID 같은 값을 활용
    @Transactional
    public ProductResponse createProduct(ProductCreateServiceRequest request) {
        // productNumber 부여
        // DB에서 마지막으로 저장된 상품의 상품번호를 읽어와서 1 증가
        String nextProductNumber = productNumberFactory.createNextProductNumber();

        Product product = request.toEntity(nextProductNumber);
        Product savedProduct = productRepository.save(product);

        return ProductResponse.of(savedProduct);
    }

```
{% endtab %}
{% endtabs %}

### 테스트에서만 필요한 메서드가 생겼는데 프로덕션에는 필요가 없다면?

***

```java
@Getter
@NoArgsConstructor
public class ProductCreateRequest {

    @NotNull(message = "상품 타입은 필수입니다.")
    private ProductType type;

    @NotNull(message = "상품 판매 상태는 필수입니다.")
    private ProductSellingStatus sellingStatus;

    private String name;

    @Positive(message = "상품 가격은 양수여야 합니다.")
    private int price;

    @Builder
    private ProductCreateRequest(ProductType type, ProductSellingStatus sellingStatus, String name, int price) {
        this.type = type;
        this.sellingStatus = sellingStatus;
        this.name = name;
        this.price = price;
    }

    public ProductCreateServiceRequest toServiceRequest() {
        return ProductCreateServiceRequest.builder()
                .type(type)
                .sellingStatus(sellingStatus)
                .name(name)
                .price(price)
                .build();
    }
}
```

이 예시 코드에서는 builder 메서드가 실제 프로덕션에서는 사용하지 않는다. 그 이유는 Controller 계층에서 외부 사용자가 입력하는 값을 전달하는 DTO 이기 때문에 생성자를 필요로 하지 않는데, 테스트 코드에서 가독성을 위해 빌더를 제공했다.

만들지 않는 것이 가장 좋겠지만 만약 만들어야 한다면 테스트에서만 사용되는 메서드를 만드는 것은 허용한다.

다만, 제약 없이 만드는 것은 지양하고 어떤 객체가 마땅히 가져도 되는 행위 이면서 미래에도 사용이 될 가능성이 있다면 만들어도된다.
