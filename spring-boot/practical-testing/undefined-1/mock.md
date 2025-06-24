# Mock을 마주하는 자세

### Test Double

***

#### <i class="fa-notes">:notes:</i> 용어 정리

{% hint style="info" %}
* **Dummy** 아무런 행동과 상태도 없는 빈 객체
* **Fake** 단순한 형태로 동일한 기능은 하지만, 실제 프로덕션의 환경과는 달라 모든 것을 검증할 수 없지만 간단한 기능과 빠른 개발 단계에서 사용하기 쉬움 -> ex. FakeRepository(단순 Map 자료 형을 이용하는 방법)
* **Stub** 테스트에서 요청한 것에 대해 미리 준비한 결과를 제공하는 객체
* **Spy** Stub 이면서 호출된 내용을 기록하여 보여줄 수 있는 객체로 일부는 실제 객체처럼 동작 하되, 일부는 Stubbing 할 수 있음
* **Mock** 행위에 대한 기대를 명세하고, 그에 따라 동작하도록 만들어진 객체
{% endhint %}



#### <i class="fa-newspaper">:newspaper:</i> 마틴 파울러의 MocksArentStubs 아티클과 함께 알아보는 Mock과 Stub 의 차이

{% embed url="https://martinfowler.com/articles/mocksArentStubs.html" %}

***

{% hint style="info" %}
#### Mock 과 Stub 의 차이

<mark style="color:blue;">**Mock**</mark> 실제 사용하는 코드의 행동이 아닌 그 행동을 구사하는 척 하는 것 -> 행동 검증 중점

<mark style="color:blue;">**Stub**</mark> 객체가 특정 행위를 수행한 이후 속성 값으로 갖고 있는 상태의 대한 변화를 검증 하는 것 -> 상태 검증 중점
{% endhint %}



> **예시로 살펴보기**
>
> _**"In both cases I'm using a test double instead of the real mail service. There is a difference in that the****&#x20;**<mark style="color:red;">**stub uses state verification**</mark>**&#x20;****while the****&#x20;**<mark style="color:red;">**mock uses behavior verification**</mark>**."**_

{% tabs %}
{% tab title="Mock" %}
"Using mocks this test would look quite different."

```java

class OrderInteractionTester...

  public void testOrderSendsMailIfUnfilled() {
    Order order = new Order(TALISKER, 51);
    Mock warehouse = mock(Warehouse.class);
    Mock mailer = mock(MailService.class);
    order.setMailer((MailService) mailer.proxy());

    mailer.expects(once()).method("send");
    warehouse.expects(once()).method("hasInventory")
      .withAnyArguments()
      .will(returnValue(false));

    order.fill((Warehouse) warehouse.proxy());
  }
}
```
{% endtab %}

{% tab title="Stub" %}
"We can then use state verification on the stub like this."

```java
class OrderStateTester...
  public void testOrderSendsMailIfUnfilled() {
    Order order = new Order(TALISKER, 51);
    MailServiceStub mailer = new MailServiceStub();
    order.setMailer(mailer);
    order.fill(warehouse);
    assertEquals(1, mailer.numberSent());
  }
```
{% endtab %}
{% endtabs %}

Mock 은 행동에 대한 결과가 어떠한 값을 반환해야 하는지를 검증하게 되며, 위 Mock 예시의 테스트 코드에서 실제 모킹된 MailService가 send() 라는 메서드를 호출이 되었는지 **메서드 호출 여부**에 대해 검증을 한다.

⇒ 호출 여부에 대한 검증을 한다는 것은 객체가 특정 행동을 정말 수행 했는가? 를 중점적으로 둔다는 것이다.

_**"사용자가 회원가입을 하면 회원가입 축하 메일을 보내는 상황을 테스트 한다면?"**_

AuthService 의 signUp() 라는 메서드를 호출하면 EventListner를 통해 메일을 발송하고 있다고 가정한다.

그렇다면 실제 MailService 에 createAccountUser() 라는 메서드가 이벤트에 의해 정말 호출이 되었는지 테스트를 해볼 수 있는데, 이 때 Mock 객체를 이용한 테스트를 활용하는 것이다.



Stub 은 Stub 객체의 메서드를 수행한 후, 객체가 갖고 있는 상태값이 메서드에 의해 제대로 변경 되었는지 검증하는 것이다.

_**"사용자가 게시글을 읽으면 조회수가 1 증가 해야 한다고 하는 상황을 테스트 한다면?"**_

ArticleService 의 read() 라는 메서드를 호출한 후 viewCount 가 1이 되었다 라는 코드를 작성하면, 이 테스트는 Stub 객체를 이용한 테스트인 것이다.















