# 한 문단에는 한 개의 주제만 담자

{% tabs %}
{% tab title="나쁜 예시" %}
```java
    @DisplayName("상품 타입이 재고 관련 타입인지를 체크한다.")
    @Test
    void badCase() {
      // given
        ProductType[] values = ProductType.values();

        // when
        for (ProductType value : values) {
            if (value == ProductType.HANDMADE) {
                // when
                boolean b = ProductType.containsStockType(value);
                
                // then
                assertThat(b).isFalse();
            }
            
            if (value == ProductType.BAKERY || value == ProductType.BOTTLE) {
                // when
                boolean b = ProductType.containsStockType(value);
                
                // then
                assertThat(b).isTrue();
            }
        }
    }

```
{% endtab %}

{% tab title="Second Tab" %}

{% endtab %}
{% endtabs %}

위 코드를 읽어보면 한 개의 테스트 내에 두 가지 상황을 테스트하고자 한다.

이 코드가 일단 읽기 좋지 못한 테스트 코드라는 것은 알 수 있겠다만, 왜 이 코드가 나쁜 코드라고 표현할까?



> 테스트 코드의 목적

테스트 코드는 내가 만든 시나리오 내에서 어떠한 환경을 구성하고자 하는지에 대한 궁극적인 의미가 담겨있다.

하지만 그런 코드의 흐름을 저해하는 것은 한 개의 테스트 문단 내에 여러가지 경우를 모두 고려하고 있는 상황인데, 이는 읽는이가 사고 회로를 한 단계 더 들어가야 한다는 것을 의미한다.

테스트 코드 = 문서, 이 내용이 성립되는 구조에서 문서를 읽는데 내용을 파악하러 들어갔더니 다른 링크로 보내고, 또 들어갔더니 다른 링크로 보내는 상황이 되는 것이다.



> 앞으로 우리가 마주해야 할 테스트 코드의 목적

우리가 지양해야 할 테스트 코드는 앞으로 한 개의 테스트 케이스에서 하나의 주제만 다루는 것을 지향하자.

만약 중복 코드가 너무 많아서 읽는 양이 길어진다면 @Parameterize 라는 어노테이션을 통해 같은 상황이지만 여러 환경을 테스트해볼 수 있다.

