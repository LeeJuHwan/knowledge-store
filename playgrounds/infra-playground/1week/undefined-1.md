---
description: 지하철 노선도를 위해 구성한 망에 서버를 얹어보아요
---

# 서버 구성하기

[미션 진행 코드](https://github.com/LeeJuHwan/infra-workshop/blob/main/subwaymap/README.md)

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

프로젝트를 위해 생성한 VPC에 지하철 노선도 어플리케이션을 실행 할 서버를 구성합니다.

{% hint style="success" %}
**서버 구성 카테고리**

* Bastion
* WEB + WAS
* Database
{% endhint %}



#### Bastion

***

{% hint style="info" %}
_**TODO**_

* 관리자 개인 로컬 IP 32 bit만 SSH 접근할 수 있도록 허용
* 로컬 키 파일 생성 및 _**0400**_ 권한 부여
{% endhint %}

***

<table data-full-width="true"><thead><tr><th width="166">PURPOSE</th><th>NAME</th><th>SUBNET</th><th>SG</th><th>SPEC</th><th>OS</th><th>KEYPAIR</th></tr></thead><tbody><tr><td><mark style="color:green;"><strong>Bastion</strong></mark></td><td>infraworkshop-apne2-bastion</td><td>infraworkshop-apne2-management-subnet-c</td><td>infraworkshop-apne2-admin-permit-security-group</td><td>t3.micro</td><td>Aamazon  Linux 2023</td><td>infraworkshop-apne2-keypair</td></tr></tbody></table>



#### WEB + WAS

***

{% hint style="info" %}
_**TODO**_

* 모든 대역폭의 Https 443 포트 접근 허용
* 관리망 SSH 22 포트 접근 허용
* Bastion Host와 로컬 키 파일 공용 사용\

{% endhint %}

***

<table data-full-width="true"><thead><tr><th width="166">PURPOSE</th><th>NAME</th><th>SUBNET</th><th>SG</th><th>SPEC</th><th>OS</th><th>KEYPAIR</th></tr></thead><tbody><tr><td><mark style="color:green;"><strong>WEB + WAS</strong></mark></td><td>infraworkshop-apne2-was</td><td>infraworkshop-apne2-public-subnet-a</td><td>infraworkshop-apne2-external-permit-security-group</td><td>t3.small</td><td>Aamazon Linux 2023</td><td>infraworkshop-apne2-keypair</td></tr></tbody></table>



#### Database

***

{% hint style="info" %}
_**TODO**_

* 외부망 MySQL 3306 포트 접근 허용
* 관리망 SSH 22 포트 접근 허용
* Bastion Host와 로컬 키 파일 공용 사용
{% endhint %}

<table data-full-width="true"><thead><tr><th width="166">PURPOSE</th><th>NAME</th><th>SUBNET</th><th>SG</th><th>SPEC</th><th>OS</th><th>KEYPAIR</th></tr></thead><tbody><tr><td><mark style="color:green;"><strong>Database</strong></mark></td><td>infraworkshop-apne2-database</td><td>infraworkshop-apne2-private-subnet-a</td><td>nfraworkshop-apne2-internal-permit-security-group</td><td>t3.micro</td><td>Aamazon Linux 2023</td><td>infraworkshop-apne2-keypair</td></tr></tbody></table>
