---
description: AWS에서 신규 망을 생성하여 DMZ와 INSIDE 분리
---

# 망 구성하기

[미션 진행 코드](https://github.com/LeeJuHwan/infra-workshop/blob/main/subwaymap/README.md)

<figure><img src="../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

VPC를 생성하고 외부망과 내부망을 나누며 필요한 아이피 대역을 서브넷팅 한 기본 논리적 구조를 생성합니다.

{% hint style="success" %}
**망 구성 카테고리**

* VPC
* Subnets
* Security Groups
* Internet Gateway
* Nat Gateway
* Route Tables
{% endhint %}



#### VPC

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

{% hint style="info" %}
_**TODO**_

* C Class CIDR VPC 생성
{% endhint %}

***

| NAME                | CIDR              |
| ------------------- | ----------------- |
| infraworkshop-apne2 | `192.168.10.0/24` |





#### Subnets

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

{% hint style="info" %}
_**TODO**_

* 외부망으로 사용할 Subnet : 64개씩 2개 (AZ를 다르게 구성)
  * 외부망은 인터넷 구간과 통신 가능
* 내부망으로 사용할 Subnet : 32개씩 1개
  * 내부망에서만 인터넷 접근 가능
* 관리용으로 사용할 Subnet : 32개씩 1개 (system manager를 사용한다면 별도의 관리망 없이 내부망 2개로 구성, 보안그룹도 상황에 맞게 구성)\

{% endhint %}

***

<table><thead><tr><th width="166">PURPOSE</th><th>NAME</th><th>CIDR</th><th>AZ</th></tr></thead><tbody><tr><td><mark style="color:green;"><strong>external</strong></mark></td><td>infraworkshop-apne2-public-subnet-a</td><td><code>192.168.10.0/26</code></td><td>ap-northeast-2a</td></tr><tr><td><mark style="color:green;"><strong>external</strong></mark></td><td>infraworkshop-apne2-public-subnet-c</td><td><code>192.168.10.64/26</code></td><td>ap-northeast-2c</td></tr><tr><td><mark style="color:green;"><strong>internal</strong></mark></td><td>infraworkshop-apne2-private-subnet-a</td><td><code>192.168.10.128/27</code></td><td>ap-northeast-2a</td></tr><tr><td><mark style="color:green;"><strong>management</strong></mark></td><td>infraworkshop-apne2-private-subnet-c</td><td><code>192.168.10.160/27</code></td><td>ap-northeast-2c</td></tr></tbody></table>



#### Security Groups

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

{% hint style="info" %}
_**TODO**_

* **외부망**
  * 전체 대역 : 443 포트 오픈
  * 관리망 : 22번 포트 오픈
* **내부망**
  * 외부망 : 3306 포트 오픈
  * 관리망 : 22번 포트 오픈
* **관리망**
  * 자신의 로컬 PC 공인 IP : 22번 포트 오픈
{% endhint %}



<table><thead><tr><th width="138">PURPOSE</th><th>NAME</th><th>INBOUND</th><th>OUTBOUND</th></tr></thead><tbody><tr><td><mark style="color:green;"><strong>external</strong></mark></td><td>infraworkshop-apne2-external-permit-security-group</td><td><code>"0.0.0.0:ICMP"</code>, <code>"0.0.0.0/0:443"</code>, <code>infraworkshop-apne2-admin-permit-security-group:22</code></td><td>0.0.0.0/0:0</td></tr><tr><td><mark style="color:green;"><strong>internal</strong></mark></td><td>infraworkshop-apne2-public-subnet-c</td><td><code>"0.0.0.0:ICMP"</code>, <code>infraworkshop-apne2-external-permit-security-group:3306</code>, <code>infraworkshop-apne2-admin-permit-security-group:22</code></td><td>0.0.0.0/0:0</td></tr><tr><td><mark style="color:green;"><strong>management</strong></mark></td><td>infraworkshop-apne2-admin-permit-security-group</td><td>my local IP/32:22</td><td>0.0.0.0/0:0</td></tr></tbody></table>



#### Internet Gateway

<figure><img src="../../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

{% hint style="info" %}
_**TODO**_

* 인터넷 게이트웨이는 퍼블릭 IP 주소를 지닌 인스턴스를 인터넷과 연결하면 인터넷에서 들어오는 요청을 수신할 수 있도록 해보아요.
{% endhint %}

<table><thead><tr><th width="138">PURPOSE</th><th>NAME</th><th>DESTINATION</th><th>VPC</th></tr></thead><tbody><tr><td><mark style="color:green;"><strong>external</strong></mark></td><td>infraworkshop-apne2-igw</td><td><code>0.0.0.0</code></td><td>infraworkshop-apne2</td></tr></tbody></table>



#### Nat Gateway

<figure><img src="../../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

{% hint style="info" %}
_**TODO**_

* INSIDE 존에 위치 해 있으며 외부와 통신을 못하는 환경에서 외부 인터넷 망에 있는 데이터를 요청할 수 있도록 구성해보아요.
{% endhint %}

<table><thead><tr><th width="138">PURPOSE</th><th>NAME</th><th>DESTINATION</th><th>VPC</th><th>CIDR</th></tr></thead><tbody><tr><td><mark style="color:green;"><strong>internal</strong></mark></td><td>infraworkshop-apne2-nat-instance</td><td><code>0.0.0.0</code></td><td>infraworkshop-apne2</td><td>192.168.10.0/24</td></tr></tbody></table>





#### Route Tables

***

<table><thead><tr><th width="138">PURPOSE</th><th>NAME</th><th>SUBNETS</th><th>GATEWAY</th></tr></thead><tbody><tr><td><mark style="color:green;"><strong>external</strong></mark></td><td>infraworkshop-apne2-public-route-table</td><td>infraworkshop-apne2-public-subnet-a, infraworkshop-apne2-public-subnet-c</td><td>infraworkshop-apne2-igw</td></tr><tr><td><mark style="color:green;"><strong>internal</strong></mark></td><td>infraworkshop-apne2-private-route-table</td><td>infraworkshop-apne2-private-subnet-a, infraworkshop-apne2-private-subnet-c</td><td>infraworkshop-apne2-nat-instance</td></tr></tbody></table>

