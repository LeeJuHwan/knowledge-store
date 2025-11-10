---
description: 소켓의 본질에 대한 이해
---

# Socket

{% hint style="info" %}
#### 소켓이란?

소켓은 네트워크 통신을 위해 OS 커널에 구현되어 있는 프로토콜(TCP/IP 등)을 추상화하여 애플리케이션에 제공하는 인터페이스이다.

* 즉, 소켓은네트워크를 파일처럼 읽고 쓸 수 있게 해주는 OS 객체인 셈이다.

애플리케이션은 소켓에 대해 파일 디스크립터를 부여받고, 이 파일 디스크립터를 통해 send/recv 같은 시스템 콜로 데이터를 주고받는다.



소켓은 디스크에 저장되는 파일 포맷이 아니라, OS가 네트워크 장치를 추상화하여 하나의 장치 파일처럼 취급하는 것이다.
{% endhint %}

_**소켓**_ 은 `file descriptor`, `local ip`, `local port`, `remote ip`, `remote port` 등 5가지 정보를 활용하여 생성 된다.









<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
