# 완벽하게 제어하기

{% tabs %}
{% tab title="나쁜 예시" %}
```java
class TestExample {

    @Test
    void createOrder() {
        CafeKiosk cafeKiosk = new CafeKiosk();
        Americano americano = new Americano();

        cafeKiosk.add(americano);

        Order order = cafeKiosk.createOrder();
        assertThat(order.getBeverages()).hasSize(1);
        assertThat(order.getBeverages().getFirst().getName()).isEqualTo("아메리카노");
    }
}


public ServiceClass {
    public Order createOrder() {
        LocalDateTime currentDateTime = LocalDateTime.now();
        LocalTime currentTime = currentDateTime.toLocalTime();

        if (currentTime.isBefore(SHOP_OPEN_TIME) || currentTime.isAfter(CLOSE_OPEN_TIME)) {
            throw new IllegalArgumentException("주문 시간이 아닙니다. 관리자에게 문의하세요.");
        }

        return new Order(currentDateTime, beverages);
    }
}
```
{% endtab %}

{% tab title="좋은 예시" %}
```java
class TestExample {

    @Test
    void createOrderWithCurrentTime() {
        CafeKiosk cafeKiosk = new CafeKiosk();
        Americano americano = new Americano();

        cafeKiosk.add(americano);

        Order order = cafeKiosk.createOrder(LocalDateTime.of(2025, 2, 10, 10, 0));
        assertThat(order.getBeverages()).hasSize(1);
        assertThat(order.getBeverages().getFirst().getName()).isEqualTo("아메리카노");

    }

}


public ServiceClass {
    public Order createOrder(LocalDateTime currentDateTime) {
        LocalTime currentTime = currentDateTime.toLocalTime();

        if (currentTime.isBefore(SHOP_OPEN_TIME) || currentTime.isAfter(CLOSE_OPEN_TIME)) {
            throw new IllegalArgumentException("주문 시간이 아닙니다. 관리자에게 문의하세요.");
        }

        return new Order(currentDateTime, beverages);
    }

}
```
{% endtab %}
{% endtabs %}

두 가지 상황에 대한 예시 코드의 핵심은 내가 제어할 수 없는 값을 어떻게 테스트할 것인가? 에 대한 내용이다.

좋은 테스트코드를 작성하기 위해서는 내가 제어할 수 없는 값을 상위 계층으로 옮기고, 메서드 내부에서는 주입되는 값을 기준으로 행동해야한다.

"나쁜 예시" 에 나왔듯 시간을 기준으로 행동하는 메서드가 있을 때 시간에 대한 제약사항이 생긴다면 실행 하는 시점에 따라 테스트가 성공할수도, 실패할 수도 있다.&#x20;

잊지 말아야 할 점은 테스트 코드는 언제나 실패하거나 언제나 성공해야 하는 트랜잭션의 원자성과 같은 존재다.



좋은 예시에서는 시간을 외부 사용하는 시점에서 주입하게 되고, 메서드 내부에서는 주입 받는 값을 사용하는데 적절하게 쓰이고 있으니 테스트하기 편리하다.

* 이는 테스트코드에서 Given 절에서 필요한 시간 데이터를 생성해서 주입하면 되기 때문



이렇게 내부 시스템에서 내가 온전히 제어할 수 없다면 항상 상위 계층으로 빼거나, Mocking 을 통해 의도하는 값을 사용해서 언제나 테스트에 대한 신뢰를 보장해야한다.



앞으로는 코드를 작성할 때 이 질문을 끊임없이 던져보자.

> 내부 시스템에서 메서드를 완벽하게 제어할 수 있는가?

```java
    @DisplayName("주문 생성 시 주문 등록 시간을 기록한다.")
    @Test
    void registeredDateTime() {
        // given
        LocalDateTime registeredDateTime = LocalDateTime.now();
        List<Product> products = List.of(
                createProduct("001", 1000),
                createProduct("002", 2000)
        );

        // when
        Order order = Order.create(products, registeredDateTime);

        // then
        assertThat(order.getRegisteredDateTime()).isEqualTo(registeredDateTime);
    }

```

추가적으로 위 테스트 코드를 보면, 시간을 외부에서 주입하기 때문에 좋은 테스트코드 처럼 보인다. 또 그렇게 설명을 했었다.

하지만 여기서 간과한 점은 테스트 코드 내에서도 변하는 값에 대한 신뢰를 보장하기 위해서 우리는 "상수" 를 이용하여 테스트 하는 것이 가장 바람직하다.

그 이유는 만약, LocalDateTime.now 라는 메서드를 한 번 호출 했다면 이 메서드를 검증 하거나 사용해야 하는 부분에 이 변수는 계속 따라다니게 된다.

즉 한 개의 변수가 테스트 코드 내에서 한 번 사용 되었다는 이유로 테스트 코드의 내용이 추가 될 때 마다 같이 쫓아다니며 동일한 값을 유도해야한다.

그래서 가능하다면 최대한 검증하는 값은 상수를 사용하고, 항상 내부 시스템에서 온전히 제어할 수 있도록 코드를 작성하는 의식적 연습이 필요하다.



