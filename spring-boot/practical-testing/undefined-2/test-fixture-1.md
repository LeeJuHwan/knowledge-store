---
description: 테스트 환경에서 구성한 Fixture 들을 어떻게 하면 효율적으로 삭제할 수 있을까?
---

# Test Fixture 클렌징





### deleteAll vs deleteAllInBatch

```java
class Test {

    @AfterEach
    void tearDown() {
        repository.deleteAllInBatch();
        repository.deleteAll();
    }
    ...
```

테스트 코드를 작성하다보면 위 처럼 테스트 환경의 독립성을 위해 구성되었던 환경을 모두 재설정하는 작업을 하게된다.

이 과정에서 deleteAll() 을 쓸지, deleteAllInBatch() 를 쓸지 결정해야 하는데 그 차이점에 대한 이해하는 내용이다.



#### 예시 Entity

{% tabs %}
{% tab title="OrderProduct" %}
```java
public class OrderProduct extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    private Order order;

    @ManyToOne(fetch = FetchType.LAZY)
    private Product product;

    public OrderProduct(Order order, Product product) {
        this.order = order;
        this.product = product;
    }
}
```
{% endtab %}

{% tab title="Order" %}
```java
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
```
{% endtab %}

{% tab title="Product" %}
```java
public class Product extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String productNumber;

    @Enumerated(EnumType.STRING)
    private ProductType type;

    @Enumerated(EnumType.STRING)
    private ProductSellingStatus sellingStatus;

    private String name;

    private int price;
```
{% endtab %}
{% endtabs %}

앞으로 설명하게 될 메서드에서 해당 코드 구조를 기반으로 테스트 코드의 AfterEach 내 클렌징 코드를 예시로 든다.



#### deleteAllInBatch

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

deleteAllInBatch 가 실행하는 쿼리를 보면 실제 테이블에 대한 아무런 조건 없이 모든 데이터를 삭제하게 된다.

그렇기 때문에 테스트 환경에서 생성 되었던 모든 데이터를 보다 빠르게(조건 없이 한 번에 삭제하기 때문) 삭제가 가능한데, 단점이 있다면 JPA 특성상 다른 객체와 연관 관계를 맺고 있어 서로 외래키로 강하게 묶여있는 경우는 삭제가 불가능하다.

이 때는 외래 키를 참조하고 있는 객체 먼저 삭제 해야하는 "순서" 를 고려해서 삭제 순서를 결정해야한다.

```java
class OrderServiceTest {

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private OrderProductRepository orderProductRepository;

    @Autowired
    private StockRepository stockRepository;

    @Autowired
    private OrderService orderService;

    @AfterEach
    void tearDown() {
        orderProductRepository.deleteAllInBatch();
        orderRepository.deleteAllInBatch();
        productRepository.deleteAllInBatch();
        stockRepository.deleteAllInBatch();
    }

```

<[#entity](test-fixture-1.md#entity "mention") 참고 >&#x20;

OrderProduct 는 Order 와 Product 의 다대다 관계를 풀어주는 N:1 매핑 테이블이다.

만약 OrderServiceTest에서 tearDown 순서를 Order 먼저 제거하거나 Product 먼저 제거하는 경우 JPA Foreign Key 제약 사항으로 인한 에러를 발생시킨다.

이렇게 연관 관계를 조금 더 수월하게 삭제할 수 있는 메서드가 deleteAll() 이다.



#### deleteAll()

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

deleteAll() 의 쿼리를 살펴보면 Product 테이블에 존재하는 모든 데이터를 지우는 것은 동일 하지만, deleteAllInBatch() 의 delete from product 가 아닌 개별로 하나씩 접근해서 지우는 것을 확인할 수 있다.

또한 삭제하기 전 모든 데이터를 조건 없이 조회하는 것을 볼 수 있다.

이 때 deleteAllInBatch 가 갖고 있던 문제점 중 "연관 관계를 맺고있는 객체를 삭제할 수 없기 때문에 순서를 고려하여 삭제해야 한다." 라는 내용을 나름 해결할 수 있다.

<[#entity](test-fixture-1.md#entity "mention") 참고 >&#x20;

삭제하고자 하는 테이블에 모든 데이터를 조회할 때 관계를 맺고 있던 테이블도 같이 조회하여  삭제한다.

다만, 양방향인 경우 Order 를 먼저 삭제하든 Product 를 먼저 삭제하든 상관 없지만 Entity 관계를 보면 단방향으로 Product는 Order를 모르기 때문에 이 때 Product를 먼저 삭제하면 deleteAllInBatch 와 마찬가지로 외래 키 제약 에러가 발생한다.

즉, 객체가 다른 객체를 참조하고 있는 경우 또한 같이 조회 하여 삭제하기 때문에 관계 매핑을 순서에 비교적 자유롭게(아예 상관 없지는 않으니) 모두 지우고 싶다면 deleteAll 이 해결책이 될 수있다.



> 두 메서드의 차이는 어떻게 구현되어있을까?

{% tabs %}
{% tab title="SimpleJpaRepository" %}
```java
	@Override
	@Transactional
	public void deleteAll() {

		for (T element : findAll()) {
			delete(element);
		}
	}

	@Override
	@Transactional
	public void deleteAllInBatch() {

		Query query = entityManager.createQuery(getDeleteAllQueryString());

		applyQueryHints(query);

		query.executeUpdate();
	}
```
{% endtab %}
{% endtabs %}

실제 SimpleJpaRepository 의 deleteAll 과 deleteAllInBatch 내부 구현부분이다.

이 부분만 보더라도 둘의 차이를 명확히 알 수 있다.

[#undefined-7](../undefined-1/business-layer.md#undefined-7 "mention")섹션에서 다루는 SpringBootTest 와 Transactional 내용 중 tearDown 메서드를 권장하고, 트랜잭셔널은 가급적 자제하자 라고 말을 했지만 실제 트랜잭셔널 어노테이션이 테스트 환경에 대한 독립성을 보장하는데 굉장히 큰 편리함을 준다.

섹션에서는 트랜잭셔널을 사용했을 때 발생할 수 있는 사이드 이펙트를 정확히 인지하지 않은 채 사용하는 것을 지양하고 있으니 충분히 인지된 상태에서는 편리하게 롤백을 지원 받는 트랜잭셔널 어노테이션을 충분히 잘 활용하며 혼합적으로 사용하는 것을 추천한다.

그 외에도 스프링배치 처럼 트랜잭션에 대한 경계가 무수히 많아 트랜잭셔널을 적용하기 어려운 경우 위 처럼 순수하게 deleteAll 또는 deleteAllInBatch 을 고려해서 사용하면 된다.

절대적으로 모든것을 파훼할 방법이란 것은 없으니 상황에 맞게 트랜잭셔널을 사용하든, 티어다운 메서드를 작성하든 독립성을 위한 클렌징 작업을 한다는 것은 다름 없다.

