# 클라우드 기초 개념: EC2 기초

<details>

<summary>Properties</summary>

:pencil:2024.08.30

:page_facing_up: [AWS 강의실](https://www.inflearn.com/course/%EC%89%BD%EA%B2%8C-%EC%84%A4%EB%AA%85%ED%95%98%EB%8A%94-aws-%EA%B8%B0%EC%B4%88/dashboard)

</details>

## ENI, EIP

{% hint style="info" %}

"탄력적 네트워크 인터페이스는 VPC에서 가상 네트워크 카드를 나타내는 논리적 네트워킹 구성 요소입니다." - AWS

{% endhint %}


### ENI(Elastic Network Interface)

- EC2의 가상 랜카드이며 IP주소와 MAC주소를 보유 하고 있음

- 하나의 인스턴스에 "여러개의 ENI"를 연동 할 수 있음
    - 즉, 하나의 인스턴스가 한 개 이상의 아이피를 보유 할 수 있음

- 인스턴스 유형 및 사이즈에 따라 최대 보유 가능한 IP주소가 변동 -> 사용하는 리소스 자원이 비쌀수록 하나의 인스턴스에서 여러 개의 아이피를 보유하여 트래픽을 분산하는 방식 등 다양하게 이용이 가능함

- 내부적으로는 Security Group은 ENI에 부착 되어있음

- 기본적으로 Private IP와 Private 도메인을 보유 하고 있으며 선택적으로 퍼블릭 아이피와 퍼블릭 도메인을 보유 할 수 있음


### EIP(Elastic IP): 탄력적 아이피

- EC2의 퍼블릭 아이피를 고정해주는 서비스
    - 인스턴스를 중지 후 재시작 시 퍼블릭 아이피가 변경 되었지만 탄력적 아이피를 사용하면 고정적인 아이피를 확보 할 수 있음

- EC2이외에도 다른 서비스에서 사용 가능, 예: ELB 등

- 내가 보유한 아이피 주소를 AWS에서 직접적으로 사용이 가능함

- 리전 단위이기 때문에 서울리전에서 생성한 탄력적 아이피의 주소가 버지니아 북부 리전에서 동일하게 존재 할 수 없음

- 연결하지 않아도 보유하기만 해도 비용 발생(IPv4 비용 발생)

![image](../../../.gitbook/assets/eip.png)
