---
description: Web(Controller) 계층 테스트하기
---

# Presentation Layer



<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
#### Prensentation Layer

* 외부 사용자의 요청을 가장 먼저 받는 계층
* 외부 사용자에게 필요한 정보에 대해 최소한의 검증을 수행
{% endhint %}

외부 사용자가 의도한 대로 데이터를 보냈을 시 어플리케이션은 내부적으로 어떻게 처리할 것인가를 해당 레이어에서 명시하게 되며, 이 때 주로 데이터가 처리 되는 부분을 "Mocking" 해서 테스트를 진행한다.



### Mock

***

잘 동작한다고 가정하고, 실제 사용하는 코드의 행동이 아닌 그 행동을 구사하는 척 하는 가짜 객체를 의미함

> #### MockMvc
>
> #### Mock 객체를 사용해 스프링 MVC 동작을 재현할 수 있는 테스트 프레임워크









