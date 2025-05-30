# Business Layer 테스트

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
_**Business Layer**_

* 비즈니스 로직을 구현하는 역할
* Persistence Layer 와의 상호작용(Data를 읽고 쓰는 행위)를 통해 비즈니스 로직을 전개시킨다.
* <mark style="color:red;">**트랜잭션**</mark>을 보장해야한다.
{% endhint %}



### 비즈니스 요구사항

***

* [ ] &#x20;상품 번호 리스트를 받아 주문 생성하기
* [ ] 주문은 주문 상태, 주문 등록 시간을 가져야 함
* [ ] 주문의 총 금액을 계산 할 수 있어야 함



### 엔티티 설계

***

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

주문과 상품과의 다대다 관계를 띄고 있다. 하지만, JPA 의 연관관계에서 다대다 관계를 이용하기엔 성능 상 이슈나 데이터 설계 관점에서의 정규화가 어긋날 수 있어 지양하는 관계구조이다.

그렇기에 대부분 다대다 관계를 풀어 내기 위한 해결책으로 일대다 - 다대일 방식으로 풀어내며 중간 테이블을 만든다.&#x20;

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

실패하는 테스트 코드를 우선 "실패" 시키기 위해서라면 컴파일 단계에서 발생할 수 있는 에러는 모두 잡아줘야한다.&#x20;

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
