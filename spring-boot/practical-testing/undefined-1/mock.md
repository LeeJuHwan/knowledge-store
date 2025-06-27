# Mock을 마주하는 자세

### Stunt Double 이 아닌 Test Double

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

{% hint style="info" %}
**"stubbing이 필요한 이유"**

\
모킹 객체인의 특정 메서드가 기본 값 반환 정책이 false 라면, 테스트 하고자 하는 메서드에 영향을 미치기 때문에 stubbing 하여 반환하는 값을 바꾼다.

여기서 기본 값 반환 정책이 true였다면 굳이 stubbing 해주지 않아도 된다

* &#x20;테스트에서 검증하고자 하는 대상이 아니기 때문
{% endhint %}

결론적으로 모킹해오는 객체들은 stubbing을 해주지 않는다면 기본 값들을 반환하는 정책을 따르며, 테스트하려는 대상에만 집중하고 내가 제어할 수 없는 외부 세계의 영역을 stubbing(mocking) 한다.

이토록 어렵게 느껴지는 Mock 과 Stub 의 차이는 정의의 내용이 우리를 혼란스럽게 할 뿐, 실질적으로 용어를 사용할 땐 대부분 Mock 이라는 단어로 포괄하여 사용한다.&#x20;

굳이 굳이, mock 과 stub 을 엄밀히 구분해가며 소통하지 않으니 개념적으로만 이해할 요소라고 생각이 든다.



### Mockito 어노테이션 여행기

***

{% tabs %}
{% tab title="@Mock" %}
```java
@ExtendWith(MockitoExtension.class) // NOTE: 해당 어노테이션이 있으므로 테스트가 시작될 때 Mockito 를 활용하여 Mock 객체를 생성한다..
class MailServiceTest {

    @Mock
    private MailSendClient mailSendClient;

    @Mock
    private MailSendHistoryRepository mailSendHistoryRepository;

    @DisplayName("메일 전송 테스트")
    @Test
    void sendMail() {
        // given
        MailService mailService = new MailService(mailSendClient, mailSendHistoryRepository)
        // @Mock Stubbing
        when(mailSendClient.sendMail(anyString(), anyString(), anyString(), anyString()))
                .thenReturn(true);

        // when
        boolean result = mailService.sendMail("fromEmail", "toEmail", "subject", "content");

        // then
        assertThat(result).isTrue();

        // save 행위가 1번 호출 됐는지 검증 하는 메서드
        verify(mailSendHistoryRepository, times(1)).save(any(MailSendHistory.class));

    }
}
```
{% endtab %}

{% tab title="@InjectMocks" %}
```java
@ExtendWith(MockitoExtension.class) // NOTE: 해당 어노테이션이 있으므로 테스트가 시작될 때 Mockito 를 활용하여 Mock 객체를 생성한다..
class MailServiceTest {

    @Mock
    private MailSendClient mailSendClient;

    @Mock
    private MailSendHistoryRepository mailSendHistoryRepository;

    @InjectMocks  // NOTE: MailService의 생성자를 확인하여 Mock 객체로 생성된 객체를 주입한다. -> DI와 동일
    private MailService mailService;

    @DisplayName("메일 전송 테스트")
    @Test
    void sendMail() {
        // given
//        MailSendClient mailSendClient1 = mock(MailSendClient.class);
        // @Mock Stubbing
        when(mailSendClient.sendMail(anyString(), anyString(), anyString(), anyString()))
                .thenReturn(true);

        // when
        boolean result = mailService.sendMail("fromEmail", "toEmail", "subject", "content");

        // then
        assertThat(result).isTrue();

        // save 행위가 1번 호출 됐는지 검증 하는 메서드
        verify(mailSendHistoryRepository, times(1)).save(any(MailSendHistory.class));

    }
}
```
{% endtab %}

{% tab title="@Spy" %}
```java
@ExtendWith(MockitoExtension.class) // NOTE: 해당 어노테이션이 있으므로 테스트가 시작될 때 Mockito 를 활용하여 Mock 객체를 생성한다..
class MailServiceTest {

    @Spy
    private MailSendClient mailSendClient;

    @Mock
    private MailSendHistoryRepository mailSendHistoryRepository;

    @InjectMocks  // NOTE: MailService의 생성자를 확인하여 Mock 객체로 생성된 객체를 주입한다. -> DI와 동일
    private MailService mailService;

    @DisplayName("메일 전송 테스트")
    @Test
    void sendMail() {
        // given
        // @Spy Stubbing
        doReturn(true)
                .when(mailSendClient)
                .sendMail(anyString(), anyString(), anyString(), anyString());

        // when
        boolean result = mailService.sendMail("fromEmail", "toEmail", "subject", "content");

        // then
        assertThat(result).isTrue();

        // save 행위가 1번 호출 됐는지 검증 하는 메서드
        verify(mailSendHistoryRepository, times(1)).save(any(MailSendHistory.class));

    }
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
#### @Mock

MockitoBean 과 달리, 순수 Mock 객체를 이용하여 스프링 컨테이너 환경에서 동작하는 통합 테스트가 아닌, 단위 테스트 위주의 테스트 케이스를 작성할 수 있다.

Mock 어노테이션을 클래스 변수로 작성한 뒤 ExtendWith(MockitoExtension.class) 를 클래스 레벨에 사용하면 Mock 객체가 기본으로 생성된다.



@Mock 어노테이션은 아래 코드와 같은 기본 값 생성 코드를 생략할 수 있는 편리함을 제공한다.

```java
MailSendClient mailSendClient1 = mock(MailSendClient.class);
```
{% endhint %}

> "Mock 객체의 특정 메서드를 Stubbing 하였지만, 메서드 내부에서 다른 메서드를 호출 하는 경우 어떻게 될까?"

```java
public class MailService {

    private final MailSendClient mailSendClient;
    private final MailSendHistoryRepository mailSendHistoryRepository;

    public boolean sendMail(String fromEmail, String toEmail, String subject, String content) {

        boolean result = mailSendClient.sendMail(fromEmail, toEmail, subject, content);

        if (result) {
            mailSendHistoryRepository.save(
                    MailSendHistory.builder()
                            .fromEmail(fromEmail)
                            .toEmail(toEmail)
                            .subject(subject)
                            .content(content)
                            .build()
            );
            mailSendClient.a();
            mailSendClient.b();
            mailSendClient.c();

            return true;
        }

        return false;
    }

...

MailServiceTest

        // @Mock Stubbing
        when(mailSendClient.sendMail(anyString(), anyString(), anyString(), anyString()))
                .thenReturn(true);

```

위 코드를 보면, MailServiceTest 에서 MailSendClient 의 sendMail() 을 Stubbing 하여 실제 사용자에게 보내는 메일을 보내지 않고 테스트 하기 위한 코드를 작성했다.

하지만, 내부적으로 호출 되고 있는 mailSendHistoryRepository 의 save() 메서드는 어떻게 될까?&#x20;

메일 발송 기록을 저장하는 코드인데, 이 내용은 테스트 할 때 마다 저장되면 안되기 때문에 똑같이 Stubbing 을 해줘야하는걸까?



_**Stubbing 메서드 내부에서 실행되는 다른 메서드 디버깅하기**_

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

디버깅을 해보면 save() 메서드의 결과값이 null 인 것을 알 수 있고, 실제 저장이 되지 않았다.

Mock 객체 내부 코드를 살펴보면, 생성할 때 기본값을 사용할 수 있도록 유도하며 기본 값을 반환하게 된다.

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
#### @InjectMocks

어노테이션 명칭에서도 알 수 있듯이 Mock 객체를 주입하는 어노테이션으로, MailService 같이 Mock 객체에 대한 의존관계를 갖고 있을 때 테스트 케이스에서 Mock 객체가 생성 되어 있다면 의존 관계를 주입해주는 방식이다.

스프링의 DI 방식과 동일하다.
{% endhint %}

{% hint style="info" %}
#### @Spy

Mock과 유사하지만 Spy는 실제 객체의 일부 메서드만 Stubbing 하여 사용할 수 있다.

Mock 은 특정 행동에 대해 정의하지 않으면 기본 값을 반환 하지만, Spy는 정의 되지 않은 행동은 실제 메서드의 반환 값을 그대로 사용한다.

그 중 정의된 행동만 의도한 값이 사용된다.



```java
public boolean sendMail(String fromEmail, String toEmail, String subject, String content) {

    boolean result = mailSendClient.sendMail(fromEmail, toEmail, subject, content);

    if (result) {
        mailSendHistoryRepository.save(
                MailSendHistory.builder()
                        .fromEmail(fromEmail)
                        .toEmail(toEmail)
                        .subject(subject)
                        .content(content)
                        .build()
        );
        mailSendClient.a();
        mailSendClient.b();
        mailSendClient.c();
```



sendMail() 에 a, b, c 메서드를 호출하고 각 메서드는 각자의 메서드명을 로그로 출력하는 행동을 한다.

그 다음, 테스트 코드에서 sendMail 만 Stubbing 하면 아래와 같은 출력 결과를 얻는다.



```
12:53:45.919 [main] INFO sample.cafekiosk.spring.client.mail.MailSendClient -- a
12:53:45.921 [main] INFO sample.cafekiosk.spring.client.mail.MailSendClient -- b
12:53:45.921 [main] INFO sample.cafekiosk.spring.client.mail.MailSendClient -- c
```
{% endhint %}



### BDD Given 절에서 Mockito.when 을 쓰며 어색함을 느끼지 못하였다

***

{% tabs %}
{% tab title="MailServiceTest.java" %}
```java
package sample.cafekiosk.spring.api.service.mail;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.anyString;
import static org.mockito.Mockito.doReturn;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.BDDMockito;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.Spy;
import org.mockito.junit.jupiter.MockitoExtension;
import sample.cafekiosk.spring.client.mail.MailSendClient;
import sample.cafekiosk.spring.domain.history.mail.MailSendHistory;
import sample.cafekiosk.spring.domain.history.mail.MailSendHistoryRepository;

@ExtendWith(MockitoExtension.class) // NOTE: 해당 어노테이션이 있으므로 테스트가 시작될 때 Mockito 를 활용하여 Mock 객체를 생성한다..
class MailServiceTest {

    @Mock
    private MailSendClient mailSendClient;

    @Mock
    private MailSendHistoryRepository mailSendHistoryRepository;

    @InjectMocks  // NOTE: MailService의 생성자를 확인하여 Mock 객체로 생성된 객체를 주입한다. -> DI와 동일
    private MailService mailService;

    @DisplayName("메일 전송 테스트")
    @Test
    void sendMail() {
        // given
        // @Mock Stubbing
//        when(mailSendClient.sendMail(anyString(), anyString(), anyString(), anyString()))
//                .thenReturn(true);
        BDDMockito.given(mailSendClient.sendMail(anyString(), anyString(), anyString(), anyString()))
                .willReturn(true);

        // when
        boolean result = mailService.sendMail("fromEmail", "toEmail", "subject", "content");

        // then
        assertThat(result).isTrue();

        // save 행위가 1번 호출 됐는지 검증 하는 메서드
        verify(mailSendHistoryRepository, times(1)).save(any(MailSendHistory.class));

    }
}
```
{% endtab %}
{% endtabs %}

테스트 케이스를 작성할 때 현재까지 BDD 방식을 따라왔다. 그렇기에 Mock 객체 생성 또는 Stubbing 과정을 테스트를 위한 준비 과정으로써 Given 에 속하는 것이 맞다.

하지만, Mockito 의 Stubbing 메서드 명칭은 Mockito.when 이다. 테스트 코드도 하나의 문서 처럼 활용할 수 있기 때문에 읽는이로 하여금 오해를 살 수 있는 부분이된다.

역시 소프트웨어 세계는 대부분 누군가 했던 고민의 흔적이 있듯, 이런 Mockito 라이브러리를 그대로 래핑하여 BDDMockito 라는 라이브러리를 제공하고, 위 코드에서 볼 수 있듯이 given 이라는 명칭을 쓴다.

모든 동작은 Mockito 와 같지만 더 자연스러운 읽는 흐름을 갖을 수 있기 때문에, 앞으로 Given 절에서 Stubbing 할 땐 BDDMockito 로 꾸미지 않았지만 꾸민 느낌을 주어 편안함을 제공해보자.



### 당신은 Classicist 인가요? Mockist 인가요?

{% embed url="https://jamesblog95.tistory.com/entry/Mockist-vs-Classicist" %}

{% hint style="info" %}
#### Classicist

실제 객체의 메서드 반환 값을 중점으로 테스트 하며 동작에 대한 신뢰성을 바탕으로 하는 소프트웨어 테스팅 접근 방식

이 입장은 모든 객체에 대한 Mocking 을 진행하게 되면 실제 반영되는 코드가 변경 되었을 시 모킹 객체의 행동 값을 재정의 하지 않으면 테스트는 통과하지만, 사용자 행동은 오류가 나는 상황을 일련의 방지하고자 하는 목적이다.

다만, 난 클래시스트이기 때문에 모킹은 절대 하지않아 의 입장은 아니기 때문에 최소한으로 사용하자는 주의이다.

> "엣헴, 누가 모키토 소리를 내었는가? 😤"
{% endhint %}

{% hint style="info" %}
#### Mockist

의존 관계를 Mocking 하여 현재 테스트 하고자 하는 부분만 잘게 쪼개어 메서드간 독립성을 바탕으로 하는 소프트웨어 테스팅 접근 방식

앞서 레이어드 아키텍처의 계층간 테스트 코드를 작성하며 필요한 의존 관계를 Mocking 하거나 실제 객체의 코드를 이용하기도 했었다.

Presentation Layer 를 예로 들었을 때 해당 계층에서는 의존 관계인 서비스 계층을 아예 Mocking 해버린 뒤 클라이언트가 입력한 값이 정상적으로 들어오는지만 판단했다.

그럼 굉장히 단순한 부분만 테스트할 수 있고, 시간도 꽤나 단축 된다. 이렇게 테스트하는 것을 서비스 계층, 퍼시스턴스 계층 또는 객체간 단위 테스트 속에서도 의존관계를 띄고 있다면 Mocking 해서 현재 내가 구현해야 할 기능을 빠르게 테스트 해볼 수 있다.
{% endhint %}



이 두 입장 간의 차이는 개발자간의 성향에서 나타나겠지만 나는 개인적으로 클래시스트이다. 테스트 코드를 작성하는 이유는 현재 기능에 대한 빠른 테스트가 아닌 전체적인 서비스에 대한 신뢰성이라고 생각하는데, 이 또한 반대 입장의 모키스트의 얘기를 들어보면 모호한 부분이 있을 것이다.

그 이유는 아무래도 테스트 코드 작성만으로 서비스의 신뢰성을 모두 보장할 수 있는가? 와 밀접한 주제일 것 같고, 그 외에는 매번 테스트할 때 마다 드는 시간적 비용이 우리 서비스 코드를 작성하는 데 뺏기고 있다 라는 것이 될 것 같다.

그럼에도 불구하고 클래시스트인 이유는 Mocking 한 계층의 서비스 코드 변경 사항이 테스트 코드를 업데이트 하지 않은 채 프로덕션에 반영하려고 CI 를 돌려보면 통과하지만, 사용자의 행동은 우리가 의도했던 바와 다르게 동작할 수 있다는 이유만으로 개인적으로 클래시스트 입장에서 테스트 코드를 작성한다.



> "그럼 어떨 때 Mocking 을 해야할까?"

Mocking 이 필요한 부분은 우리 시스템 코드가 아닌, 외부에 영향을 받고 있는 시스템들을 사용중인 메서드가 있다면 단언컨대 이 때 필요하다 라고 말할 수 있다.

외부 메일 서버를 이용하여 사용자에게 메일을 발송하거나, 외부 인증/인가 서버를 통해 사용자 정보를 받아오거나 등 외부 시스템에서 발생한 장애는 우리 시스템에서 수정할 수 없고, 제어권이 없기 때문이다.
