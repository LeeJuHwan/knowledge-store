---
description: 도커 컨테이너에 대한 짧은 이야기
---

# 컨테이너 사전 지식

### 도커 컨테이너는 프로세스를 관리한다

{% hint style="info" %}
_**도커 컨테이너는 프로세스의 라이프 사이클을 관리한다**_

프로세스가 생성되고 운영되고 제거되기 까지 생애주기를 관리하는데 이 때 필요한 것은 "<mark style="color:green;">**격리**</mark>" 이다.
{% endhint %}

{% hint style="info" %}
_**컨테이너 안의 프로세스는 제한된 자원내에서 제한된 사용자만 접근이 가능하다.**_
{% endhint %}

{% hint style="info" %}
_**컨테이너는 Host OS의 시스템 커널을 사용한다.**_

<img src="../../.gitbook/assets/image.png" alt="" data-size="original">
{% endhint %}



#### 가상머신과 컨테이너의 차이

***

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

> **가상머신**

하이퍼 바이저가 존재하고 각각의 Guest OS 들이 리소스들을 활용하기 위해서 하이퍼 바이저를 통해 시스템 콜을 발생 시킨다.

> **도커 컨테이너**

가상 머신과 달리 도커 컨테이너는 도커 컨테이너 엔진이 별도의 프로세스로 할당 되어 실행이 되고 각각의 프로세스들을 격리해서 사용한다.

> **"하지만 파일 시스템은 다르다"**

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

프로세스를 별개로 실행 시켜 파일 시스템이 다르다는 것을 어떻게 할 수 있는 것일까?

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
_**chroot**_

루트 파일 시스템을 강제로 인식시켜 프로세스를 실행하는 것
{% endhint %}

프로세스는 이와 같이 자기 자신이 위치해 있는 곳을 루트로 인식 하기 때문에 컨테이너가 곧 자신의 세상이라고 인식하는 것이다.

> **그래서 컨테이너 내에선 자기가 첫번째 프로세스로 할당된다.**

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>



> _**"Host OS는 컨테이너를 어떻게 바라볼까?"**_

위 프로세스는 Nginx 컨테이너인데, Nginx를 직접 실행하나, 컨테이너 안에서 실행되나 Host OS에겐 그저 동일한 Nginx 프로세스일 뿐이다.



### 도커 컨테이너 네트워크

#### 격리중인 컨테이너가 다른 프로세스와 통신하기

***

> _**"도커 컨테이너는 격리 되어 있는데 어떻게 다른 컨테이너와 통신이 가능할까?"**_

도커 컨테이너는 컨테이너들의 위치를 아는 요소가 존재한다. 만약, Nginx 프로세스를 관리하는 도커 컨테이너 1개를 생성 했을 때 망 구성이 어떻게 될지 살펴보자.

{% stepper %}
{% step %}
### 네트워크 인터페이스 카드 생성

위에서 <mark style="color:purple;">**chroot**</mark>를 통해 독자적인 파일 시스템이 생성 된다고 했었는데, 이 때 <mark style="color:red;">**네트워크 인터페이스 카드**</mark>도 하나 생성된다.


{% endstep %}

{% step %}
### MAC 주소와 IP 주소 할당

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

컨테이너 내부 네트워크 상태를 확인 해보면 컨테이너에 "<mark style="color:red;">**MAC 주소와 IP가 할당 되어있음**</mark>"
{% endstep %}

{% step %}
### 라우팅 테이블의 게이트웨이 설정

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

컨테이너 내부 라우팅 테이블 구성을 보면 **모르는 대역의 경우 `172.17.0.1`로 보내도록 게이트웨이가 설정** 되어있음
{% endstep %}

{% step %}
### L2 스위치 장비 소프트웨어 버전인 Bridge driver

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
**Bridge**

* L2 Switch 장비의 소프트웨어 버전
{% endhint %}

**`172.17.0.1`이 할당된 게이트웨이가 존재하고 이 네트워크는 bridge 모드로 동작게된다.**
{% endstep %}

{% step %}
### docker0

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

**`172.17.0.1`은&#x20;**<mark style="color:red;">**Host OS의 docker0 라는 녀석에게 할당**</mark>되어 있다.
{% endstep %}
{% endstepper %}

> "요약 해보자면"

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

> **망 외부 통신**

IP가 `172.17.0.3`인 녀석이 `172.17.0.0/16` <mark style="color:green;">**망 외부**</mark>와 통신하려고 할 때 `172.17.0.1`에게 통신을 보낸다.

물론, 이 때 서버 내에서 `172.17.0,3` IP로 직접 통신도 가능하다.



> **망 내부 통신**

IP가 `172.17.0.3`인 녀석이 `172.17.0.0/16` <mark style="color:green;">**망 내부**</mark>와 통신하려고 할 때 docker0는 L2 Switch 역할을 하는 bridge 모드를 통해서 직접 통신을 한다.

{% hint style="info" %}
**주소를 알고 있다면 다른 컨테이너에게 직접 접근 하고 모른다면 전체 컨테이너에게 broadcast를 보낸다.**
{% endhint %}

{% hint style="info" %}
Summary

1. 컨테이너를 생성하면, 가상 네트워크 인터페이스가 생성이 되고 컨테이너 내의 <mark style="color:green;">**eth0**</mark>과 연결된다.
2. 컨테이너들의 가상 네트워크 인터페이스들은 <mark style="color:green;">**docker0**</mark>을 통해 컨테이너들간에 통신이 가능하다.
3. 컨테이너는 gateway인 <mark style="color:green;">**docker0**</mark>을 거쳐 외부와 통신이 가능하다.
{% endhint %}



> **"컨테이너 IP를 직접 다 확인하기 어렵고 외부로부터 오는 요청은 Host의 NIC로 오는데 어떻게 다른 컨테이너간 통신을 유연하게 이룰 수 있지?"**

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
_**포트 포워딩 활용**_

<mark style="color:green;">**Host OS의 80 포트번호를 사용하는 소켓 파일**</mark>과 <mark style="color:blue;">**컨테이너 내의 80 포트번호를 사용하는 소켓 파일**</mark>을 서로 연결할 수 있다.

포트 포워딩을 활용하면 Host OS가 직접 요청을 처리할 수 있게 된다.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
방화벽 정책(iptables)

도커 컨테이너 명령어로 포트포워딩을 활용 하면 방화벽 정책에 목적지가 추가된 것을 확인할 수 있다.
{% endhint %}



#### 도커 컨테이너의 영속성

***

> "컨테이너 이미지를 제거하면 어떻게 될까?"

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

컨테이너 이미지는 <mark style="color:green;">**읽기 전용**</mark>이다. 즉, 컨테이너를 제거하면 메모리에서 사라지기 때문에 영속적인 데이터를 다룰 수 없다.

그렇기 때문에 영속성 데이터는 위 이미지 처럼 컨테이너가 아닌 외부에 데이터를 저장하고, 컨테이너는 그 데이터로 동작할 수 있도록 stateless 하게 구성해야한다.

