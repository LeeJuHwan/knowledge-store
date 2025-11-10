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

<figure><img src="../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

_**소켓**_ 은 `file descriptor`, `local ip`, `local port`, `remote ip`, `remote port` 등 5가지 정보를 활용하여 생성 된다.

이중 우리가 할당하는 포트인 `local port`의 경우, 프로세스가 구동될 때 할당하기도 하고, /etc/services 를 보면 이미 known port로 지정되어 있기도 한다.

특정 프로세스 (가령, 8080 포트번호의 톰캣 서버)에 보내는 클라이언트의 요청은 `remote port`를 제외하고 모두 동일하다.

**`remote port`는 client의 브라우저에서 요청 보낼 시 랜덤으로 지정하는 `unknown port` 이며 이로인해 요청을 구분할 수 있게 된다**.

> _**"하나의 프로세스(서버)에서 몇 개의 클라이언트와 연결이 가능할까?"**_

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

위 이미지 처럼 해당 프로세스에 별다른 제약이 없다면 기본적으로 생성할 수 있는 <mark style="color:red;">**파일의 갯수가 제한**</mark>되어 있다.

> _**"소켓을 하나 더 두어 요청을 다른 포트 번호로 전달할 수는 없을까?"**_

<figure><img src="../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

192.168.0.1:80 포트로 요청을 전달 받아 192.168.0.1:8080 포트로 전달하여 프로세스가 처리할 수 있는데, 이를 "<mark style="color:red;">**포트 포워딩**</mark>" 이라고 한다.

주로 컨테이너 환경에서 많이 사용된다.