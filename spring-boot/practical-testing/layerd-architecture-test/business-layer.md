---
description: Application(Service) 계층 테스트 하기
---

# Business Layer

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
_**Business Layer**_

* 비즈니스 로직을 구현하는 역할
* Persistence Layer 와의 상호작용(Data를 읽고 쓰는 행위)를 통해 비즈니스 로직을 전개시킨다.
* <mark style="color:red;">**트랜잭션**</mark>을 보장해야한다.
{% endhint %}

### 비즈니스 요구사항

***

* [ ] 상품 번호 리스트를 받아 주문 생성하기
* [ ] 주문은 주문 상태, 주문 등록 시간을 가져야 함
* [ ] 주문의 총 금액을 계산 할 수 있어야 함

### 엔티티 설계

***

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

주문과 상품과의 다대다 관계를 띄고 있다. 하지만, JPA 의 연관관계에서 다대다 관계를 이용하기엔 성능 상 이슈나 데이터 설계 관점에서의 정규화가 어긋날 수 있어 지양하는 관계구조이다.

그렇기에 대부분 다대다 관계를 풀어 내기 위한 해결책으로 일대다 - 다대일 방식으로 풀어내며 중간 테이블을 만든다.

이 것이 그 예제이다.

{% tabs %}
{% tab title="Order.java" %}
```java
package sample.cafekiosk.spring.domain.order;


import jakarta.persistence.CascadeType;
import jakarta.persistence.Entity;
import jakarta.persistence.EnumType;
import jakarta.persistence.Enumerated;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.OneToMany;
import jakarta.persistence.Table;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;
import sample.cafekiosk.spring.domain.BaseEntity;
import sample.cafekiosk.spring.domain.orderproduct.OrderProduct;

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

}
```
{% endtab %}

{% tab title="OrderStatus.java" %}
```java
package sample.cafekiosk.spring.domain.order;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public enum OrderStatus {

    INIT("주문생성"),
    CANCELED("주문취소"),
    PAYMENT_COMPLETED("결제완료"),
    PAYMENT_FAILED("결제실패"),
    RECEIVED("주문접수"),
    COMPLETED("처리완료");

    private final String description;
}
```
{% endtab %}

{% tab title="OrderProduct.java" %}
```java
package sample.cafekiosk.spring.domain.orderproduct;


import jakarta.persistence.Entity;
import jakarta.persistence.FetchType;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.ManyToOne;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;
import sample.cafekiosk.spring.domain.BaseEntity;
import sample.cafekiosk.spring.domain.order.Order;
import sample.cafekiosk.spring.domain.product.Product;

@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
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
{% endtabs %}

### TDD와 함께 비즈니스 로직 테스트 하기

***

#### 실패하는 테스트 코드 작성하기

우선 실패하는 테스트 코드를 먼저 작성하는데, 이 때 객체의 생성 관점이 아닌 객체의 행동 관점으로 기능을 테스트 한다.

객체의 생성 관점은 지나친 시그니처(속성 값)을 만들어내지만, 행동은 필요한 것만 만들기 때문에 올바른 객체지향 설계를 하고싶다면 관점을 이해해야 한다.

{% tabs %}
{% tab title="OrderServiceTest.java" %}
```java
package sample.cafekiosk.spring.api.service.order;

import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.tuple;
import static sample.cafekiosk.spring.domain.product.ProductSellingStatus.SELLING;
import static sample.cafekiosk.spring.domain.product.ProductType.HANDMADE;

import java.time.LocalDateTime;
import java.util.List;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import sample.cafekiosk.spring.api.controller.order.request.OrderCreateRequest;
import sample.cafekiosk.spring.api.service.order.response.OrderResponse;
import sample.cafekiosk.spring.domain.product.Product;
import sample.cafekiosk.spring.domain.product.ProductRepository;
import sample.cafekiosk.spring.domain.product.ProductType;


@SpringBootTest
class OrderServiceTest {

    @Autowired
    ProductRepository productRepository;

    @Autowired
    private OrderService orderService;

    @DisplayName("주문번호 리스트를 받아 주문을 생성한다.")
    @Test
    void createOrder() {
        // given
        Product product1 = createProduct(HANDMADE, "001", 1000);
        Product product2 = createProduct(HANDMADE, "002", 3000);
        Product product3 = createProduct(HANDMADE, "003", 5000);
        productRepository.saveAll(List.of(product1, product2, product3));

        OrderCreateRequest request = OrderCreateRequest.builder()
                .productNumbers(List.of("001", "002"))
                .build();

        // when
        OrderResponse orderResponse = orderService.createOrder(request);

        // then
        assertThat(orderResponse.getId()).isNotNull();
        assertThat(orderResponse)
                .extracting("registeredDateTime", "totalPrice")
                .contains(LocalDateTime.now(), 4000);
        assertThat(orderResponse.getProducts()).hasSize(2)
                .extracting("productNumber", "price")
                .containsExactlyInAnyOrder(
                        tuple("001", 1000),
                        tuple("002", 3000)
                );

    }


    private Product createProduct(ProductType type, String productNumber, int price) {
        return Product.builder()
                .type(type)
                .productNumber(productNumber)
                .price(price)
                .sellingStatus(SELLING)
                .name("메뉴 이름")
                .build();
    }

}
```
{% endtab %}
{% endtabs %}

#### 컴파일 에러를 없애 줄 객체 생성하기

실패하는 테스트 코드를 우선 "실패" 시키기 위해서라면 컴파일 단계에서 발생할 수 있는 에러는 모두 잡아줘야한다.

이 코드로 인해서 아주 빠르게 테스트 코드를 실패시킬 수 있다.

{% tabs %}
{% tab title="OrderService.java" %}
```java
package sample.cafekiosk.spring.api.service.order;

import org.springframework.stereotype.Service;
import sample.cafekiosk.spring.api.controller.order.request.OrderCreateRequest;
import sample.cafekiosk.spring.api.service.order.response.OrderResponse;

@Service
public class OrderService {

    public OrderResponse createOrder(OrderCreateRequest request) {
        return null;
    }
}
```
{% endtab %}

{% tab title="OrderResponse.java" %}
```java
package sample.cafekiosk.spring.api.service.order.response;

import java.time.LocalDateTime;
import java.util.List;
import lombok.Getter;
import sample.cafekiosk.spring.api.service.product.response.ProductResponse;

@Getter
public class OrderResponse {

    private Long id;
    private int totalPrice;
    private LocalDateTime registeredDateTime;
    private List<ProductResponse> products;

}
```
{% endtab %}
{% endtabs %}

#### 서비스 계층에서 테스트 하는 영역의 독립성 보장하기

도메인 계층에 있던 `Repository` 의 메서드를 테스트 할 때는 전혀 문제 없던 부분이 갑자기 서비스 계층을 테스트 하면서 생겨났다.

문제는 테스트하는 영역이 서로 공유 되어 사용하며 의도했던 데이터가 다른 테스트에 의해 오염이 발생한 것이다.

아래 예시 코드의 AS-IS 를 보면, `createOrder()` 테스트와 `createOrderWithDuplicateProductNumbers`() 테스트는 서로 같은 `Repository` 를 사용하게 되면서 검증 단계에서 의도치 않게 실패하게 되는 경우이다.

이러한 이유로 서비스 계층을 테스트할 땐 `TearDownMethod` 로 데이터 클린징 작업을 해야 서로간의 영역을 침범하지 않고 독립성을 보장시킬 수 있다.

{% tabs %}
{% tab title="AS-IS" %}
```java
@DisplayName("주문번호 리스트를 받아 주문을 생성한다.")
@Test
void createOrder() {
    // given
    Product product1 = createProduct(HANDMADE, "001", 1000);
    Product product2 = createProduct(HANDMADE, "002", 3000);
    Product product3 = createProduct(HANDMADE, "003", 5000);
    productRepository.saveAll(List.of(product1, product2, product3));

    OrderCreateRequest request = OrderCreateRequest.builder()
            .productNumbers(List.of("001", "002"))
            .build();

    // when
    LocalDateTime registeredDateTime = LocalDateTime.now();
    OrderResponse orderResponse = orderService.createOrder(request, registeredDateTime);

    // then
    assertThat(orderResponse.getId()).isNotNull();
    assertThat(orderResponse)
            .extracting("registeredDateTime", "totalPrice")
            .contains(registeredDateTime, 4000);
    assertThat(orderResponse.getProducts()).hasSize(2)
            .extracting("productNumber", "price")
            .containsExactlyInAnyOrder(
                    tuple("001", 1000),
                    tuple("002", 3000)
            );

}

@DisplayName("중복되는 상품번호 리스트로 주문을 생성할 수 있다.")
@Test
void createOrderWithDuplicateProductNumbers() {
    // given
    Product product1 = createProduct(HANDMADE, "001", 1000);
    Product product2 = createProduct(HANDMADE, "002", 3000);
    Product product3 = createProduct(HANDMADE, "003", 5000);
    productRepository.saveAll(List.of(product1, product2, product3));

    OrderCreateRequest request = OrderCreateRequest.builder()
            .productNumbers(List.of("001", "001"))
            .build();

    // when
    LocalDateTime registeredDateTime = LocalDateTime.now();
    OrderResponse orderResponse = orderService.createOrder(request, registeredDateTime);

    // then
    assertThat(orderResponse.getId()).isNotNull();
    assertThat(orderResponse)
            .extracting("registeredDateTime", "totalPrice")
            .contains(registeredDateTime, 2000);
    assertThat(orderResponse.getProducts()).hasSize(2)
            .extracting("productNumber", "price")
            .containsExactlyInAnyOrder(
                    tuple("001", 1000),
                    tuple("001", 1000)
            );

}
```
{% endtab %}

{% tab title="TO-BE" %}
```java
@ActiveProfiles("test")
@SpringBootTest
//@DataJpaTest
class OrderServiceTest {

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private OrderProductRepository orderProductRepository;

    @Autowired
    private OrderService orderService;

    @AfterEach
    void tearDown() {
//        productRepository.deleteAll();
        orderProductRepository.deleteAllInBatch();
        productRepository.deleteAllInBatch();
        orderRepository.deleteAllInBatch();
    }

    @DisplayName("주문번호 리스트를 받아 주문을 생성한다.")
    @Test
    void createOrder() {
        // given
        Product product1 = createProduct(HANDMADE, "001", 1000);
        Product product2 = createProduct(HANDMADE, "002", 3000);
        Product product3 = createProduct(HANDMADE, "003", 5000);
        productRepository.saveAll(List.of(product1, product2, product3));

        OrderCreateRequest request = OrderCreateRequest.builder()
                .productNumbers(List.of("001", "002"))
                .build();

        // when
        LocalDateTime registeredDateTime = LocalDateTime.now();
        OrderResponse orderResponse = orderService.createOrder(request, registeredDateTime);

        // then
        System.out.println("orderResponse = " + orderResponse);
        assertThat(orderResponse.getId()).isNotNull();
        assertThat(orderResponse)
                .extracting("registeredDateTime", "totalPrice")
                .contains(registeredDateTime, 4000);
        assertThat(orderResponse.getProducts()).hasSize(2)
                .extracting("productNumber", "price")
                .containsExactlyInAnyOrder(
                        tuple("001", 1000),
                        tuple("002", 3000)
                );

    }

    @DisplayName("중복되는 상품번호 리스트로 주문을 생성할 수 있다.")
    @Test
    void createOrderWithDuplicateProductNumbers() {
        // given
        Product product1 = createProduct(HANDMADE, "001", 1000);
        Product product2 = createProduct(HANDMADE, "002", 3000);
        Product product3 = createProduct(HANDMADE, "003", 5000);
        productRepository.saveAll(List.of(product1, product2, product3));

        OrderCreateRequest request = OrderCreateRequest.builder()
                .productNumbers(List.of("001", "001"))
                .build();

        // when
        LocalDateTime registeredDateTime = LocalDateTime.now();
        OrderResponse orderResponse = orderService.createOrder(request, registeredDateTime);

        // then
        assertThat(orderResponse.getId()).isNotNull();
        assertThat(orderResponse)
                .extracting("registeredDateTime", "totalPrice")
                .contains(registeredDateTime, 2000);
        assertThat(orderResponse.getProducts()).hasSize(2)
                .extracting("productNumber", "price")
                .containsExactlyInAnyOrder(
                        tuple("001", 1000),
                        tuple("001", 1000)
                );

    }
```
{% endtab %}
{% endtabs %}

실제 이 예시에 사용된 서비스 계층은 `SpringBootTest` 어노테이션을 사용하고 있으며, 도메인 계층에서 사용한 테스트 코드는 `DataJpaTest` 어노테이션을 사용하여 테스트 하게 된다.

* <mark style="color:purple;">`SpringBootTest`</mark>: 스프링을 실행하기 위한 모든 `Bean` 들을 스프링 컨테이너에 등록하여 테스트 함
* <mark style="color:purple;">`DataJpaTest`</mark>: `JPA` 와 관련한 `Bean` 들만 스프링 컨테이너에 등록하여 테스트 함

어노테이션에 따른 차이는 위와 같이 단편적으로만 이해하고 있었으나, 실제 해당 어노테이션의 내부를 살펴보면 `DataJpaTest` 는 `Transactional` 을 사용하고 있기 때문에 테스트 영역간의 롤백을 지원 받아 독립성을 보장한 것이다.

반면, `SpringBootTest` 는 `Transactional`이 없기 때문에 테스트 영역을 공유하여 `TearDownMethod` 를 사용했지만, `DataJpaTest` 처럼 `Transcational` 을 붙여도 테스트는 통과한다.

{% tabs %}
{% tab title="SpringBootTest" %}
<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
{% endtab %}

{% tab title="DataJpaTest" %}
<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
{% endtab %}
{% endtabs %}

아래 콘솔은 `SpringBootFramework` 의 `ORM` 로그 레벨을 `DEBUG` 상태로 변경 하여 실제 `DataJpaTest` 에서 테스트 영역간 롤백을 진행 하는지 확인 해볼 수 있다.

테스트를 시작하기 전 트랜잭션으로 테스트 영역을 묶어둔 뒤 `DML` 이 일어나고, 모든 테스트가 끝나면 롤백 시킨 후 트랜잭션이 종료된다.

```
2025-06-04T13:40:31.656+09:00 DEBUG 81634 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Found thread-bound EntityManager [SessionImpl(1200408049<open>)] for JPA transaction
2025-06-04T13:40:31.656+09:00 DEBUG 81634 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Participating in existing transaction
Hibernate: 
    insert 
    into
        product
        (created_date_time, last_modified_date_time, name, price, product_number, selling_status, type, id) 
    values
        (?, ?, ?, ?, ?, ?, ?, default)
Hibernate: 
    insert 
    into
        product
        (created_date_time, last_modified_date_time, name, price, product_number, selling_status, type, id) 
    values
        (?, ?, ?, ?, ?, ?, ?, default)
Hibernate: 
    insert 
    into
        product
        (created_date_time, last_modified_date_time, name, price, product_number, selling_status, type, id) 
    values
        (?, ?, ?, ?, ?, ?, ?, default)
Hibernate: 
    select
        p1_0.id,
        p1_0.created_date_time,
        p1_0.last_modified_date_time,
        p1_0.name,
        p1_0.price,
        p1_0.product_number,
        p1_0.selling_status,
        p1_0.type 
    from
        product p1_0 
    where
        p1_0.selling_status in (?, ?)
2025-06-04T13:40:31.793+09:00 DEBUG 81634 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Initiating transaction rollback
2025-06-04T13:40:31.793+09:00 DEBUG 81634 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Rolling back JPA transaction on EntityManager [SessionImpl(1200408049<open>)]
2025-06-04T13:40:31.794+09:00 DEBUG 81634 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Closing JPA EntityManager [SessionImpl(1200408049<open>)] after transaction
2025-06-04T13:40:31.801+09:00 DEBUG 81634 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Creating new transaction with name [sample.cafekiosk.spring.domain.product.ProductRepositoryTest.findAllByProductNumberIn]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
2025-06-04T13:40:31.801+09:00 DEBUG 81634 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Opened new EntityManager [SessionImpl(1747991213<open>)] for JPA transaction
2025-06-04T13:40:31.801+09:00 DEBUG 81634 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Exposing JPA transaction as JDBC [org.springframework.orm.jpa.vendor.HibernateJpaDialect$HibernateConnectionHandle@1d0acb8f]
2025-06-04T13:40:31.802+09:00 DEBUG 81634 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Found thread-bound EntityManager [SessionImpl(1747991213<open>)] for JPA transaction
2025-06-04T13:40:31.802+09:00 DEBUG 81634 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Participating in existing transaction

```

> _**"****`SpringBootTest`****&#x20;****와****&#x20;****`Transactional`****&#x20;****\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*을 같이 쓴다면 이런 점은 주의 해야한다."**_

```java
@ActiveProfiles("test")
@SpringBootTest
@Transactional
class OrderServiceTest {
...
}
```

{% hint style="info" %}
스프링 데이터 JPA는 트랜잭션이 시작될 때 스냅샷을 찍어 상태를 보존하고, EntityManaer의 커밋이 완료 되는 시점에서 최초 스냅샷과 현재 상태를 비교하여 변경이 감지 되면(`Dirty Check`) 업데이트 쿼리를 호출한다.
{% endhint %}

위와 같이 테스트 클래스에서 트랜잭션이 적용되면 실제 서비스 계층에서 트랜잭션이 적용되었다는 인지의 오류를 범할 수 있다.

그렇기 때문에 테스트는 항상 테스트 독립적인 환경을 구성하기 위해 노력해야 하고, 이 사실을 정확히 인지하였다고 하더라도 명시적으로 데이터 클린징 작업을 직접 하는 것이 소프트웨어의 오류를 조금이나마 줄일 수 있는 방법이다.

:bulb:서비스 계층을 테스트 하면서 데이터 클린징 작업이 필요하다면 `TearDown` 을 적극 활용하자

> _**"도메인 객체에서 비즈니스 로직에 대한 유효성 검증과 서비스 계층에서 연산을 위한 유효성 검증을 동일하게 할 필요가 있을까?"**_

{% tabs %}
{% tab title="Stock.java" %}
```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class Stock extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String productNumber;

    private int quantity;

    @Builder
    private Stock(String productNumber, int quantity) {
        this.productNumber = productNumber;
        this.quantity = quantity;
    }

    public static Stock create(String productNumber, int quantity) {
        return Stock.builder()
                .productNumber(productNumber)
                .quantity(quantity)
                .build();
    }

    public boolean isQuantityLessThan(int quantity) {
        return this.quantity < quantity;
    }

    public void deductQuantity(int quantity) {
        if (isQuantityLessThan(quantity)) {
            throw new IllegalArgumentException("차감할 재고 수량이 없습니다.");
        }

        this.quantity -= quantity;
    }
}
```
{% endtab %}

{% tab title="OrderService.java" %}
```java
        // 재고 차감 시도
        for (String stockProductNumber : new HashSet<>(stockProductNumbers)) {
            Stock stock = stockMap.get(stockProductNumber);
            int quantity = productCountingMap.get(stockProductNumber).intValue();

            if (stock.isQuantityLessThan(quantity)) {
                throw new IllegalArgumentException("재고가 부족한 상품이 있습니다.");
            }

            stock.deductQuantity(quantity);

        }
```
{% endtab %}
{% endtabs %}

위 도메인(`Stock`) 과 서비스(`OrderService`) 코드를 보면, 두 계층에서 모두 동일한 유효성 검증(`isQuantityLessThan`)을 하고 있다.

재고를 차감하는 메서드(`deductQuantity`) 내부에서만 진행 하면 되지 않을까? 라는 궁금증이 생길 수 있지만, 이는 엄연히 객체를 바라보는 관점에서 차이를 두어야한다.

`OrderService` 가 아닌 `StockService` 라는 곳에서 `deductQuantity` 메서드를 호출하는데, 만약 이 때 서비스 계층에서 유효성 검증을 진행 했었기 때문에 `Stock` 도메인 객체 내부에서는 유효성 검증을 진행 하지 않는다면 오류가 발생할 여지가 생긴다.

또한 도메인이 제공하는 비즈니스 로직은 언제나 유효한 로직이 진행 되어야한다.

그리고, 서비스 계층은 도메인의 비즈니스 로직을 호출하는 입장에서 다른 오류 메시지나 다른 관점에서 유효성 검증을 진행할 수도 있기 때문에, 두 영역이 중복되는 유효성 검증을 진행한다고 해서 검증을 제거할 필요는 없다는 것이고, 이런 검증이 필요하다는 것이다.
