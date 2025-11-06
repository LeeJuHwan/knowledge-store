# 테스트 환경의 독립성을 보장하자

> 짚고 넘어갈 내용

* 테스트 코드에서 테스트 환경에 대한 독립성을 보장한다는 의미는 무엇일까?
* 위 테스트 코드에서 독립성을 깨트리고 있는 부분은 어느 부분일까?



테스트 코드 내에서 테스트 환경은 한가지 주제에 대한 테스트를 수행하기 위한 환경을 의미한다.&#x20;

예를 들어 상품 A, B, C 를 등록 해두고 전체 상품을 조회 하는 아래와 같은 테스트 코드가 있다고 가정해보자.

```java
    @Test
    void find() {
        // given
        Product productA = createProduct("A");
        Product productB = createProduct("B");
        Product productC = createProduct("C");
        productRepository.saveAll(List.of(productA, productB, productC));


        // when // then
        List<Product> products = productRepository.findAll();
        assertThat(products).hasSize(3)
    }

```

이 테스트 코드가 갖고 있는 주제는 "상품을 등록하면 데이터베이스에 저장된다."  라는 내용을 담고있다.

여기서 이 주제를 뒷받침 하는 환경은 상품 A, B, C 를 생성하는 과정이다. 하지만, 우리는 지금 저 함수가 어떤 역할을 하는지 알 수 없다.

**여기서 문제가 발생한다.**

모두 통과한 테스트 코드가 있으니 이제 다른 요구사항을 해결하러 갔다. 옆 자리에 앉은 동료 PM 이 내게 상품을 주문할 때 앞으로는 언제 등록 했는지 시간도 기록 해달라고 요구했다.

그래서 난 호기롭게 시간을 추가했고, 또 다른 요구사항이 생겨 앞으로는 상품의 갯수를 같이 등록할 수 있게 해달라고 한다.

지금 이렇게 테스트 코드에서 수정해야 할 사항이 두 가지가 생겨났다. 이는 메서드 시그니처의 변경이든 내부 메서드의 구현내용이든 이러한 메서드를 사용하고 있을 땐 외부의 요구사항에 의해 테스트 코드가 실패하는 것을 지양해야한다.

그렇다면, 저 코드에서 요구사항을 변경하지 않고 테스트 코드를 실행하면 당연히 컴파일 에러가 발생하고 저 코드를 고쳐야 하는 상황이 발생한다.



{% tabs %}
{% tab title="테스트 코드" %}
```java
    @DisplayName("재고가 부족한 상품으로 주문을 생성하려는 경우 예외가 발생한다.")
    @Test
    void createOrderWithoutStock() {
        // given
        LocalDateTime registeredDateTime = LocalDateTime.now();

        Product product1 = createProduct(BOTTLE, "001", 1000);
        Product product2 = createProduct(BAKERY, "002", 3000);
        Product product3 = createProduct(HANDMADE, "003", 5000);
        productRepository.saveAll(List.of(product1, product2, product3));

        Stock stock1 = Stock.create("001", 2);
        Stock stock2 = Stock.create("002", 2);
        stock1.deductQuantity(1); // todo
        stockRepository.saveAll(List.of(stock1, stock2));

        OrderCreateServiceRequest request = OrderCreateServiceRequest.builder()
                .productNumbers(List.of("001", "001", "002", "003"))
                .build();

        // when // then
        assertThatThrownBy(() -> orderService.createOrder(request, registeredDateTime))
                .isInstanceOf(IllegalArgumentException.class)
                .hasMessage("재고가 부족한 상품이 있습니다.");
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


```
{% endtab %}
{% endtabs %}

위 테스트 코드를 보면, 예시로 들었던 createProduct도 있지만 여기서는 테스트 코드를 위한 헬퍼 함수로 쓰이고 있다.

그러니 문제 되지 않는데, 걸리는 부분은 Stock.creaete() 와 stock1.deductQuantity() 메서드이다.



> 테스트 코드에서는 팩토리 메서드를 사용하는 것을 왜 지양할까?

팩토리 메서드 패턴을 사용하는 이유는 빈 생성자가 주는 가독성에서 더 자연스러움을 택하면서 팩토리 메서드 내부에서 유효성 검증을 같이 하고자 하는 경우에 많이 쓰인다.

여기서 메서드 내부에 "유효성 검증"이 포함이 되는 순간, 맨 처음 예시 처럼 테스트 코드에서 임계 값을 수정하기 위해 팩터리 메서드로 생성하는 생성자의 파라미터 값만 변경 했는데 테스트가 깨졌다.

이러한 이유로 테스트 코드에서는 최대한 빈 생성자로 값을 생성 하여 독립성을 확보하는 것이 바람직하다.

> 이미 사용중인 메서드는 왜 테스트코드에서 재사용하면 안될까?

그렇다면 stock1.deductQuantity() 메서드도 팩터리 메서드를 지양하자는 이유와 마찬가지일 것이다.

이 메서드도 내부에서 수행하는 행위에 따라 메서드 시그니처를 변경하는 순간 테스트가 깨질 수 있다는 것이 문제다.



결론적으로, 편하게 쓰기 위해 Given 환경에서 메서드를 통해 여러가지 구성하다보면 언젠가 변경이 되는 날 Given 에 의해 테스트가 깨지고 만다.

우리 테스트에서 검증이 일어나야 할 부분은 언제나 When 이나 Then 에 위치 해 있어야한다.



### 테스트 간 독립성을 보장하자

두 가지 이상의 테스트 케이스가 한 개의 자원을 서로 공유하여 사용하는 경우를 방지하는 것이 목표이다.



{% tabs %}
{% tab title="예시코드" %}
```java
package sample.cafekiosk.spring.domain.stock;

import static org.assertj.core.api.Assertions.assertThat;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

public class StockExampleTest {

    private static final Stock stock = Stock.create("001", 1);

    @DisplayName("재고의 수량이 제공된 수량보다 작은지 확인한다.")
    @Test
    void isQuantityLessThanEx() {
      // given
        int quantity = 2;

      // when
        boolean result = stock.isQuantityLessThan(quantity);

        // then
        assertThat(result).isTrue();
    }

    @DisplayName("재고를 주어진 개수만큼 차감할 수 있다.")
    @Test
    void deductQuantityEx() {
      // given
        int quantity = 1;

      // when
        stock.deductQuantity(quantity);

        // then
        assertThat(stock.getQuantity()).isZero();
    }

}
```
{% endtab %}
{% endtabs %}

자원을 공유하게 되면 상태 값 또는 속성에 해당하는 값을 여러 테스트 케이스에서 변경하고 검증한다면 테스트가 수행 되는 순서에 따라 성공과 실패를 결정하게 된다.

테스트는 서로간의 순서와 상관 없이 실행 되어야 하며, 같은 환경 내 공유하는 자원에 의존하지 말아야한다.

결코 테스트 환경에서 작성한 순서대로 테스트가 수행 되지 않으니, 위에서 아래로 진행될거란 생각은 금물이다. 테스트는 언제나 병렬적으로 일어나기 때문에 순서는 보장할 수 없다.

