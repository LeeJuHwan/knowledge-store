---
description: 소켓의 본질에 대한 이해
---

# Socket

{% hint style="info" %}
**소켓이란?**

소켓은 네트워크 통신을 위해 OS 커널에 구현되어 있는 프로토콜(TCP/IP 등)을 추상화하여 애플리케이션에 제공하는 인터페이스이다.

* 즉, 소켓은 네트워크를 파일처럼 읽고 쓸 수 있게 해주는 OS 객체인 셈이다.

애플리케이션은 소켓에 대해 파일 디스크립터를 부여받고, 이 파일 디스크립터를 통해 send/recv 같은 시스템 콜로 데이터를 주고받는다.

소켓은 디스크에 저장되는 파일 포맷이 아니라, OS가 네트워크 장치를 추상화하여 하나의 장치 파일처럼 취급하는 것이다.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

_**소켓**_ 은 `file descriptor`, `local ip`, `local port`, `remote ip`, `remote port` 등 5가지 정보를 활용하여 생성 된다.

이중 우리가 할당하는 포트인 `local port`의 경우, 프로세스가 구동될 때 할당하기도 하고, /etc/services 를 보면 이미 known port로 지정되어 있기도 한다.

특정 프로세스 (가령, 8080 포트번호의 톰캣 서버)에 보내는 클라이언트의 요청은 `remote port`를 제외하고 모두 동일하다.

> "IP가 같은 동일한 인터넷 환경에서 여러 사용자가 접근하면, 사용자를 어떻게 구분할까?"

**`remote port`는 client의 브라우저에서 요청 보낼 시 랜덤으로 지정하는 `unknown port` 이며 이로인해 요청을 구분할 수 있게 된다**.



소켓 프로그래밍을 직접 해보면 아래와 같은 흐름으로 클라이언트의 요청을 수신 받을 수 있다.

{% stepper %}
{% step %}
소켓 객체를 생성 하고, 명시된 IP 주소를 바인딩 한다.
{% endstep %}

{% step %}
소켓 서버에서 클라이언트가 접속 할 수 있도록 요청을 받을 준비를 한다.
{% endstep %}

{% step %}
무한 루프를 통해 클라이언트의 접속 요청이 있는지 지속적으로 확인한다.
{% endstep %}

{% step %}
클라이언트의 메세지를 수신 한다.
{% endstep %}

{% step %}
클라이언트는 접속을 종료 시킨다.
{% endstep %}
{% endstepper %}

{% tabs %}
{% tab title="파이썬 소켓 프로그래밍" %}
```python
def run_server(self):
    with socket.socket(family=socket.AF_INET, type=socket.SOCK_STREAM) as sock:
    self.server_listen(sock, "0.0.0.0", 8080)

        while 1:
            client_socket, (client_host, client_port) = sock.accept()
            message = client_socket.recv(1024)
            client_socket.close()
```
{% endtab %}
{% endtabs %}

TCP 소켓에서 데이터를 받을 때 스트림 형태로 받게 된다. 이는 마치 파일을 `read` 모드로 열고, 그 내용을 처음부터 끝까지 순차적으로 읽는 것과 같다.

그리고 스트림 데이터를 여러개의 `Segment` 로 만들기 위해 `Segmentation` 을 거쳐, 하위 레이어에 데이터를 넘기기 위해 패킷으로 포장하여 전달하는 과정을 거쳐 데이터를 송신하게 된다.

* 애플리케이션은 5000바이트 스트림을 TCP에 전달
* TCP는 이 5000바이트를 여러 개의 `Segment`로 분할
  * Segment 1: 1460 바이트
  * Segment 2: 1460 바이트
  * Segment 3: 1460 바이트
  * Segment 4: 620 바이트



> _**"하나의 프로세스(서버)에서 몇 개의 클라이언트와 연결이 가능할까?"**_

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

위 이미지 처럼 해당 프로세스에 별다른 제약이 없다면 기본적으로 생성할 수 있는 <mark style="color:red;">**파일의 갯수가 제한**</mark>되어 있다.



> _**"소켓을 하나 더 두어 요청을 다른 포트 번호로 전달할 수는 없을까?"**_

<figure><img src="../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

192.168.0.1:80 포트로 요청을 전달 받아 192.168.0.1:8080 포트로 전달하여 프로세스가 처리할 수 있는데, 이를 "<mark style="color:red;">**포트 포워딩**</mark>" 이라고 한다.

주로 컨테이너 환경에서 많이 사용된다.



### 소켓 통신 흐름

***

#### 연결 - 3way handshake

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

연결 시 세그먼트의 주요 정보로는 Sequence Number, Acknowldgement Number 가 있다.

* Sequence Number: 이건 내가 생성한 번호야.
* Acknowldegement Number: 이건 우리가 잘 받았다는 의미로 보내는 신호야. 내가 보낸 번호에서 1 더해서 반환해줘.

{% hint style="info" %}
#### Sequnce Number

Sequence Number는 자기 자신이 보내야 할 임의의 수이며 Acknowledegement Number는 상대에게 다음 차례에 보내야 할 숫자를 가르키는 것과 같다.
{% endhint %}

{% stepper %}
{% step %}
**클라이언트가 서버에게 요청을 보냄**

이 때 세그먼트 정보는 Source Port(클라이언트 자신), Destination Port(대상 서버), Sequence Number(클라이언트의 임의의 수), Acknowldegement Number(서버의 임의의 수), Flags(요청 목적)

* 클라이언트의 Sequence Number 생성 `13`
* Flags 할당 `SYN`: 클라이언트의 첫 요청이기 때문
{% endstep %}

{% step %}
**서버가 요청을 받은 뒤 새로운 세그먼트를 생성하여 응답**

클라이언트의 Sequence Number에서 + 1: `14` -> 요청을 잘 받았다는 의미

* 서버의 Sequence Number 생성 `4431`
* 세그먼트:
  * &#x20;Sequence Number: `4431`
  * Acknowldegement Number: `14`
  * Flags: `SYN, ACK`
* 생성된 세그먼트를 다시 클라이언트에게 응답
{% endstep %}

{% step %}
**클라이언트가 응답을 확인 후 서버와 통신 종료 요청**

서버가 보내준 Acknowledege Number는 기존 "13" 이었지만 서버와 통신 후 응답을 받을 땐 "14"로 1이 증가 되어있음

* 서버의 Sequence Number를 잘 받았다는 의미로 +1 -> "4432"
* Flags를 ACK로 할당 하여 세그먼트를 생성 후 전달
{% endstep %}
{% endstepper %}

#### 종료 - 4way handhsake

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

{% stepper %}
{% step %}
클라이언트는 서버에게 ‘연결 종료 요청(FIN)’ 메시지를 보낸다.

해당 메시지는 서버에게 클라이언트가 ‘더 이상 데이터를 보내지 않겠다’라는 의미이다.
{% endstep %}

{% step %}
서버는 클라이언트로부터 메시지를 받고 ‘연결 종료 요청 수락(ACK)’ 메시지를 응답한다.

해당 시점에서 서버는 아직 클라이언트에게 보낼 데이터가 있으므로, 서버에서 클라이언트로의 데이터 전송은 계속 될 수 있다.
{% endstep %}

{% step %}
서버는 클라이언트에게 ‘연결 종료 요청(FIN)’ 메시지를 보낸다.

해당 메시지는 클라이언트에게 ‘더 이상 데이터를 보내지 않겠다’라는 의미로 메시지를 전송한다.
{% endstep %}

{% step %}
클라이언트는 서버로부터 메시지를 받고 ‘연결 종료 수락(ACK)’ 메시지를 응답한다.

해당 메시지는 클라이언트가 서버의 메시지를 받았음을 확인하는 메시지를 서버에게 전송합니다.
{% endstep %}
{% endstepper %}



### 소켓 커넥션 풀

***

#### 사용자가 웹 어플리케이션으로 요청하는 HTTP 요청 커넥션 풀

TCP 3way handshake 에 대한 연결을 수립한 소켓 통신을 시작하여, 사용자가 7계층에서 헤더 정보를 포함하여 요청을 보내게된다.

가령 GET http://localhost:8080/index.html 이라는 요청을 보냈다고 하면, 헤더의 첫 줄에 해당 정보를 확인할 수 있는데, 이 요청을 가공하여 정적 소스를 반환하게 되면 그와 연관 되어 있는 CSS, JS, Ico 같은 정적 리소스도 요청 해서 반환 받아야 한다.

이 때, 소켓을 계속 연결하면 복잡한 과정이 자주 발생하기 때문에 Keep-Alive 옵션 덕분에 새로운 소켓을 맺지 않고 정적 리소스를 요청할 수 있게 된다.

아래 디버깅 코드를 보면 자바로 구현한 소켓 서버이며, 정적 파일을 반환하기 위해 브라우저에 `http://localhost:8080/index.html` 한 줄을 입력했을 때 발생하는 요청이며, 이 외에 더 존재하지만 가독성을 위해 일부 삭제했다.

이런식으로 핸드쉐이크 과정을 지속적으로 수립하지 않고 연결된 TCP 소켓을 재사용하여 추가적인 요청을 할 수 있도록 지원하는 것이 사용자가 서버로 요청을 보낼 때의 커넥션 풀이다.

<pre><code>18:52:43.352 [DEBUG] [Thread-0] [webserver.RequestHandler] - New Client Connect! Connected IP : /0:0:0:0:0:0:0:1, Port : 62851
<strong>18:52:43.366 [DEBUG] [Thread-0] [util.HttpHeaderUtils] - header: GET /index.html HTTP/1.1
</strong>18:52:43.367 [DEBUG] [Thread-0] [util.HttpHeaderUtils] - header: Host: localhost:8080
18:52:43.367 [DEBUG] [Thread-0] [util.HttpHeaderUtils] - header: Connection: keep-alive
<strong>18:52:43.369 [DEBUG] [Thread-0] [util.HttpHeaderUtils] - Request Resource: /index.html
</strong><strong>18:52:43.375 [DEBUG] [Thread-0] [webserver.RequestHandler] - Resource Path: ./webapp/index.html
</strong>18:52:43.438 [DEBUG] [Thread-1] [webserver.RequestHandler] - New Client Connect! Connected IP : /0:0:0:0:0:0:0:1, Port : 62852
18:52:43.445 [DEBUG] [Thread-3] [util.HttpHeaderUtils] - header: User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
18:52:43.445 [DEBUG] [Thread-2] [util.HttpHeaderUtils] - header: Host: localhost:8080
18:52:43.445 [DEBUG] [Thread-2] [util.HttpHeaderUtils] - header: Connection: keep-alive
<strong>18:52:43.446 [DEBUG] [Thread-2] [util.HttpHeaderUtils] - Request Resource: /css/styles.css
</strong><strong>18:52:43.446 [DEBUG] [Thread-2] [webserver.RequestHandler] - Resource Path: ./webapp/css/styles.css
</strong>18:52:43.446 [DEBUG] [Thread-3] [util.HttpHeaderUtils] - Request Resource: /js/jquery-2.2.0.min.js
18:52:43.448 [DEBUG] [Thread-3] [webserver.RequestHandler] - Resource Path: ./webapp/js/jquery-2.2.0.min.js
18:52:43.448 [DEBUG] [Thread-4] [webserver.RequestHandler] - New Client Connect! Connected IP : /0:0:0:0:0:0:0:1, Port : 62855
<strong>18:52:43.449 [DEBUG] [Thread-4] [util.HttpHeaderUtils] - header: GET /js/bootstrap.min.js HTTP/1.1
</strong>18:52:43.449 [DEBUG] [Thread-4] [util.HttpHeaderUtils] - header: Host: localhost:8080
18:52:43.449 [DEBUG] [Thread-4] [util.HttpHeaderUtils] - header: Connection: keep-alive
<strong>18:52:43.450 [DEBUG] [Thread-4] [util.HttpHeaderUtils] - Request Resource: /js/bootstrap.min.js
</strong>18:52:43.450 [DEBUG] [Thread-5] [webserver.RequestHandler] - New Client Connect! Connected IP : /0:0:0:0:0:0:0:1, Port : 62856
<strong>18:52:43.450 [DEBUG] [Thread-4] [webserver.RequestHandler] - Resource Path: ./webapp/js/bootstrap.min.js
</strong><strong>18:52:43.450 [DEBUG] [Thread-5] [util.HttpHeaderUtils] - header: GET /js/scripts.js HTTP/1.1
</strong>18:52:43.450 [DEBUG] [Thread-5] [util.HttpHeaderUtils] - header: Host: localhost:8080
18:52:43.450 [DEBUG] [Thread-5] [util.HttpHeaderUtils] - header: Connection: keep-alive
<strong>18:52:43.451 [DEBUG] [Thread-5] [util.HttpHeaderUtils] - Request Resource: /js/scripts.js
</strong><strong>18:52:43.451 [DEBUG] [Thread-5] [webserver.RequestHandler] - Resource Path: ./webapp/js/scripts.js
</strong>
</code></pre>

#### 웹 어플리케이션 서버가 외부로 전송하는 커넥션 풀

가장 쉽게 이해할 수 있는 내용은 서버와 DB가 통신하는 통로일 것이다. 서버와 DB가 같은 호스트에 존재 한다고 하더라도 포트가 다르기 때문에 네트워크 요청이 발생하게 된다.

이 때는 위 상황과 달리 소켓의 5 tuple 정보를 모두 알고 있다. 미리 연결을 맺어둔 뒤 `ESTABLISHED` 상태의 커넥션을 풀에 담아두고 대기한다.&#x20;

* 연결은 되어 있지만 서로 데이터를 주고 받지 않는 유휴 상태의 커넥션을 관리함

이 커넥션 풀을 통해 DB 와 통신하고, 요청의 처리가 모두 완료 되었다면 커넥션을 종료하지 않고 커넥션 풀에 반환한다. 그럼 다음 요청은 재사용할 수 있기 때문에 네트워크 연결 비용과 종료 비용을 상대적으로 최적화할 수 있다.
