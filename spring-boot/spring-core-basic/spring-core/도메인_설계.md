# 예제 만들기

### 도메인 설계

***

#### 회원

{% tabs %}
{% tab title="회원 도메인 협력 관계" %}
<figure><img src="../../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>
{% endtab %}

{% tab title="회원 클래스 다이어그램" %}
<figure><img src="../../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>
{% endtab %}

{% tab title="회원 객체 다이어그램" %}
<figure><img src="../../../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>
{% endtab %}
{% endtabs %}



#### 주문

{% tabs %}
{% tab title="주문 도메인 전체" %}
<figure><img src="../../../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>
{% endtab %}

{% tab title="주문 도메인 클래스 다이어그램" %}
<figure><img src="../../../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>
{% endtab %}

{% tab title="주문 도메인 객체 다이어그램" %}
<figure><img src="../../../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>
{% endtab %}
{% endtabs %}



회원과 주문 관계에서 해당 도메인 영역에 위치해 있는 비즈니스 로직이 변경 되더라도 다른 도메인은 코드의 변경이 일어나지 않는 구조 설계이다.

가장 기초가 되는 객체지향 원칙 중 다형성을 활용한 인터페이스가 중점이 되었다.

하지만 아직 해당 인터페이스를 구현하는 구현체를 사용하는 입장에서 어떤 구현체를 쓸지 정의하고 있기 때문에 순수 자바에서 스프링이 당연시하게 해주었던 의존성 주입과 객체들을 관리하는 방식에 대한 불편함을 배우는 섹션이다.
