# 클라우드 기초 개념: EC2 기초

<details>

<summary>Properties</summary>

:pencil:2024.08.30

:page_facing_up: [AWS 강의실](https://www.inflearn.com/course/%EC%89%BD%EA%B2%8C-%EC%84%A4%EB%AA%85%ED%95%98%EB%8A%94-aws-%EA%B8%B0%EC%B4%88/dashboard)

</details>

## Amazon EC2

### EC2 인스턴스

![image](../../../.gitbook/assets/ec2_structure.png)

EC2에서 컴퓨팅을 담당하며 다양한 유형과 크기로 구성 되어있고 저장을 담당하는 EBS와 네트워크로 연결 되어있다.

저장 방법에 따라 두 가지로 분류 할 수 있는데 "EBS 연동", "인스턴스 스토어"로 분류가 가능하다.

인스턴스는 갯 수와 상관 없이 **하나의 가용영역(AZ)에 존재** 한다. 

인스턴스를 사용 할 때 여러가지 유형을 미리 정의 해놓고 해당 유형에 최적화 된 컴퓨팅 자원을 빌릴 수 있도록 AWS에서 지원 하는데 만약, 10$로 EC2를 사용해본다고 가정 해보자.

- CPU가 중요한 알고리즘을 처리 하는 서비스
    - CPU: 7$
    - RAM: 2$
    - 그래픽카드: 1$

- 메모리가 중요한 서비스
    - CPU: 2$
    - RAM: 7$
    - 그래픽카드 :1$


이런식으로 서비스에 따라 필요한 리소스가 최적화 된 유형을 선택 할 수 있다.

> EC2 인스턴스의 유형(패밀리)

- 인스턴스의 역할에 따라 CPU, 메모리, 스토리지, 네트워크 등을 조합하여 구성 되어있다.

- 각 인스턴스 유형 별로 사용 목적에 따라 최적화 된 서비스를 지원한다.
    - 예: 메모리 위주, CPU 위주 등

- 유형 별로 이름이 존재한다.
    - 예: t유형, m유형, inf유형 등
    - 같은 유형의 인스턴스들을 "패밀리" 라고 부른다.

- 타입 별 세대별로 숫자가 부여된다.
    - 예: m5 -> m 인스턴스 유형의 5번째 세대를 가르킴

- 아키텍처 및 프로세서/추가 기술에 따라 접미사가 붙는다.
    - c7gn -> c 인스턴스 유형 중 AWS Graviton 프로세서를 사용(g)을 하고 네트워크가 최적화(n) 되어있는 인스턴스


**인스턴스 유형을 읽는 방법**

- `cg7n.xlarge`
    - c: 인스턴스 타입
    - 7: 세대
    - g: 프로세서
    - n: 추가 기능 -> 여기선 네트워크 최적화
    - xlarge: 사이즈


> 인스턴스의 크기

- 같은 인스턴스 패밀리에서도 다양한 크기가 존재함

- 인스턴스의 CPU 갯 수, 메모리 크기, 성능 등으로 크기가 결정됨

- 크기가 클 수록 더 많은 메모리, CPU, 네트워크 대역폭 EBS와의 통신 가능한 대역폭을 지원함


### EC2 요금 모델

{% hint style="info" %}

"미리 내지 않고, 사용한 만큼 내고, 많이 쓸수록 적게 내개고, 예약할수록 더 적게 낸다." - AWS 비용 백서

{% endhint %}

> 요금 구성

- 인스턴스 요금

- 데이터 전송

- Public IPv4 IP

- 그 외 통합적으로 사용 될 수 있는 연동 서비스들
    - ELB, CloudWatch, EBS 등

> 요금 모델

- On-Demand: 사용한 시간 만큼
    - 수요 예측이 힘들거나 유연하게 EC2를 사용하고 싶을 때

- Spot Instances: 남는 인스턴스를 저렴하게
    - AWS에서 보유중인 남는 인스턴스를 저렴한 가격으로 제공
        - 가용영역 별, 인스턴스 유형 별 다른 풀로 관리
        - AWS도 컴퓨팅 서비스를 빌려주는 임대업자와 마찬가지이다. 만약 호텔에서 남는 방이 많다면 이 남는 방에 대한 자원이 매우 낭비가 될 것이다. 그렇기 때문에 남는 방에 한해서 일시적 할인을 적용 시켜 손님을 유치한 뒤 낭비 없이 자원이 모두 사용되도록 유도 하는게 탁월한 전략인데 이와 같은 방식을 적용한게 AWS의 Spot Instance이다.

    - **단!! 인스턴스가 언제 종료 될지 예측이 불가능함**

- Reserved Instances: 인스턴스의 사용 기간을 약정
    - 온디맨드 EC2 사용 요금을 할이 받는 방식
    - 할인 받고 싶은 EC2 인스턴스와 같은 리전, 유형 구매가 필요함
    - 약정 기간이 더 길수록 더 큰 할인율이 적용 됨 -> 1년 혹은 3년 선택 가능
    - 최대 72% 저렴

- Dedicated: 물리적인 전용 인스턴스 임대
    - 주로 라이선스 이슈/퍼포먼스 이슈를 해결 하기 위해 사용하며, 물리적으로 인스턴스/호스트 단위로 격리된 서버에서 EC2를 실행한다는 특징을 갖고 있음

    - 전용 인스턴스는 AWS에서 EC2와 같은 가상 서버를 빌릴 때 진짜 CPU 공간 1개에 가상 서버 여러개가 구성 되어있고 그 중 한 개의 가상 서버를 빌리는 것이다. 가상 서버와 가상 서버 사이에서 간섭이 이루어지지 않는 것이 이론적이나 실제로는 간섭이 발생 할 수도 있기 때문에 전용 인스턴스를 통째로 빌릴 수 있다.

- Savigs Plan: AWS의 컴퓨터 사용량을 약정
    - Compute Savings Plans: 다른 서비스(Lambda, Fargate 등 다른 서비스와 같이 사용)와 같이 약정
    - EC2 Instance Savings Plans: EC2 인스턴스 패밀리를 지정해서 약정


### EC2 생명 주기

EC2는 사용자에 의해 실행, 중지, 종료 등 다양한 액션을 제공하며 이에 따른 상태를 가진다.

![image](../../../.gitbook/assets/ec2_status.png)


- pending: EC2 프로비저닝 상태

- running: EC2가 정상적으로 프로비저닝 되어 실행중인 상태 

- rebooting: OS만 재시작 하는 상태, 이 때는 Public IP의 변동 없이 OS만 재실행 됨

- stopping: 중지를 위해 정리하는 상태

- stopped: 중지 된 상태이며 이 때 퍼블릭 IP를 회수하여 아무런 IP를 갖고 있지 않음

- Shutting down: 종료시 서버를 종료하기 위한 작업을 하는 상태

- terminated: 서버 리소스를 정리 완료 하여 종료 된 상태


**생명 주기 별 요금 표**

| 인스턴스 상태  | 설명  | 인스턴스 사용 요금 |
| :-: | :-: | :-: |
| pending  | 인스턴스가 running 상태로 될 준비 | 미청구 |
| running  | 인스턴스 사용 중 | 청구 |
| stopping  | 인스턴스가 중지 또는 최대 절전모드로 전환 중 | 미청구, 최대 절전모드 시 청구 |
| stopped  | 인스턴스 중지 상태: 재시작 가능 | 미청구 |
| shutting-down  | 인스턴스 종료 중 | 미청구 |
| terminated  | 인스턴스 영구 삭제 | 미청구 |


**생명 주기에서 알아 볼 키워드**

> "Stopped"

"Stopped" 상태에서는 요금이 청구 되지 않는다.
- 단, EBS 요금, EIP(Elastic IP) 요금 및 그 외 서비스들에 대한 요금은 계속 부과됨

- 중지 후 재시작 시 퍼블릭 아이피는 변경 됨

- EBS를 사용하는 인스턴스만 중지가 가능하며 인스턴스 스토어를 사용하면 불가능
    - EBS는 네트워크로 연결 되어 있어 중지 시 AWS에서 빌려주는 컴퓨팅 서버를 회수 해간다고 생각 하면 됨. 회수 되었을 때 사용자가 갖고 있던 데이터를 보존 할 수 있으면 중지가 가능하고, 인스턴스 스토어 처럼 데이터가 날아간다면 중지가 불가능함


> "rebooting"

재부팅은 운영체제 레벨에서만 재시작 하는 단계이다.
- 재부팅 시에는 퍼블릭 아이피의 변동이 없음


### EC2 Userdata, Metadata

> EC2 User Data

![image](../../../.gitbook/assets/ec2_userdata.png)

- EC2 인스턴스의 최초 실행 시 지정한 스크립트를 실행 하도록 설정 가능
    - 별도 설정을 통해서 재부팅 마다 실행 하도록 설정 가능

- 두가지 모드
    - Shell Script
    - cloud-init: 리눅스 이미지의 부트스트래핑을 위한 오픈소스 어플리케이션

- 주요 사용 사례
    - EC2인스턴스 설정(보안 설정, 인스턴스 설정 등)

    - 외부 패키지 다운로드

    - 설치되어 있는 어플리케이션 실행

    - 기타 EC2 실행시 필요한 동작들


> EC2 Instance Meatadata

{% hint style="info" %}

"인스턴스 메타데이터는 실행 중인 인스턴스를 구성 또는 관리하는 데 사용될 수 있는 인스턴스 관련 데이터입니다. 인스턴스 메타데이터는 호스트 이름, 이벤트 및 보안 그룹과 같은 범주로 분류됩니다." - AWS

{% endhint %}

- EC2 인스턴스의 속성 및 정보 데이터
    - AMI ID. IPv4/IPv6 주소, EBS 맵핑, 보안 그룹 연동 상황, IAM 역할 연동 등 다양한 정보가 존재

- 실행중인 EC2 인스턴스의 메타데이터 IMDS(Instance Meatadata Service)로 조회 가능
    - HTTP Endpoint로 메타데이터를 조회 할 수 있음

    - HTTP Endpoint IP 주소, 해당 주소는 고정 아이피로 외워두면 좋음
        - 169.254.169.254(IPv4)
        - fd00:ec2::254(IPv6)
    
    - 두가지 버전 지원
        - IMDS v1: Request/Response 기반의 HTTP 통신
        - IMDS v2: 세션 기반 (기본 값으로 설정 되어있음)
    
    - 주요 사용 사례
        - 인스턴스 별 설정, IAM 임시 자격증명 조회 등(AWS CLI, SDK 등 내부적으로 활용)

- EC2 실행시 "고급 설정" 탭에서 메타데이터 액세스 가능 여부 설정 가능
    - 기본 값으로 "활성화"로 설정 됨
    - IMDS의 버전을 명시 할 수 있음
    - 비용 발생 X


**IMDS v1**

버전 1은 별도의 인증이 필요 없는 HTTP Response/Request 방식으로 메타데이터를 주고 받으며, Link Local IP(169.254.169.254)는 고정적이지만 호출 하는 인스턴스의 주소에 따라 값이 변동 됨

하지만, 별도의 인증이 없기 때문에 보안적으로 취약할 수 있기 때문에 이 점을 보완 하기 위해 버전 2가 릴리즈 되었음

IMDS v1 사용 예시: 인스턴스 이름 가져오기


{% code title="bash" overflow="wrap" lineNumbers="true" %}

```bash
curl http://169.254.169.254/latest/meta-data/tags/instance/Name
```

{% endcode %}


**IMDS v2**

보안 토큰을 발급 받아 토큰 정보를 기반으로 인증하는 세션 방식이며 토큰의 유효기간은 1초에서 최대 6시간 까지 설정이 가능하다.

IMDS v1보다 더 높은 보안 수준을 제공 하고 IAM 정책 등을 활용 하여 EC2 인스턴스가 오직 IMDS v2만 사용 하도록 강제 설정이 가능함


IMDS v2 사용 예시: 인스턴스 이름 가져오기

{% code title="bash" overflow="wrap" lineNumbers="true" %}

```bash
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/tags/instance/Name
```

{% endcode %}


### EC2 권한 설정

> **IAM 자격 증명을 AWS Credential로 등록 하는 경우**

- IAM 사용자를 생성하고 IAM 자격증명을 발급 받아 EC2에 등록

- AWS Configure(AWS CLI)를 통해 등록 -> ~/aws/.credentials

- AWS Configure를 통해 손 쉽게 자격 증명을 등록 할 수 있지만 여러 계정을 관리 하려면 매 번 자격 증명을 등록하기 위한 인스턴스 여러 대에 직접 접근하여 수정하는 불상사가 있어 관리가 어려움


> **IAM 역할을 프로파일링 하여 EC2에 제공하는 경우**

- 권한이 부여된 IAM 역할을 만들고 EC2에 부여함

- 관리가 쉽고 역할만 변경 하면 되기 때문에 교체가 쉬움

- 내부적으로 지속적인 자격 증명을 변경 할 수 있으며 AWS Credentail 방식은 EC2에 접근이 가능하면 누구나 탈취 가능한 정보이지만 IAM 역할은 탈취 당하더라도 권한을 막으면 되기 때문에 보안성이 뛰어남


![image](../../../.gitbook/assets/ec2_iam_role.png)


### EC2를 잘 활용하기

{% hint style="info" %}

"Everything fails, all the time" - Werner Vogels, AWS CTO

{% endhint %}

클라우드 환경에서 컴퓨팅 자원을 활용 하는 Best Practice는 아래와 같다.

1. 클라우드 환경에서 각 인스턴스는 소모품이라고 생각하고 설계한다.
    - 예고 없이 장애가 발생하거나 통제된 방법으로 종료되는 것이 극히 정상정인 환경임
    - 즉, <mark style="color:orange;background-color:purple;">언제나 인스턴스는 예고없이 종료된다</mark>
    - 결론은 필요하면 언제나 더 가져다 쓸 수 있으면서 필요 없을 땐 버릴 수 있어야함


    > **잠깐! "EC2의 수평 확장, 수직 확장 알아보기"**

    - 수평 확장("Horizontal Scale" or "Scale Out"): 인스턴스1(CPUx1, Memory 1gb) -> 인스턴스1 * n
        - 즉, 저사양 인스턴스를 여러개 두어 트래픽을 여러대로 나눌 시 아키텍처에 대한 복잡성은 증가 할 수 있으나 비용이 상대적으로 절감 된다.
        - **AWS에서는 Scale Out 방식을 기본적으로 권장함**

    - 수직 확장("Vertical Scale" or "Scale Up"): 인스턴스1(CPUx1, Memory 1gb) -> 인스턴스1(CPUx16, Mmeory16gb)
        - 즉, 인스턴스의 사양을 업그레이드 하여 모든 트래픽을 하나의 인스턴스에서 처리 하는 방식
        - 이 때 AWS에서 구성한 하드웨어의 크기가 서로 동일 하지만 사양이 증가된 진보한 과학 시스템이 적용 되어있어 비용이 상상 이상으로 발생 할 수 있다.


2. 인스턴스의 상태를 저장하지 않는 Stateless하며 고가용성을 확보하여 설계한다.

> "**고가용성과 Stateless란?**"

- 고가용성: 인스턴스 중 하나가 비정상 종료 되더라도 자동으로 복구가 가능 해야 함

- Stateless: 인스턴스가 상태에 의존적이지 않아야 함
    - 어떠한 인스턴스도 특정 정보를 특별하게 저장 하지 않아야 함 -> 예를 들면 유저의 세션 정보, 로그인 정보 등

    - 언제나 추가 될 수 있고 언제나 삭제 되어도 무방한 의존성이 없는 상태여야 함


> "**그래서 EC2를 잘 활용하려면?"**

언제나 비정상 종료가 될 수 있다는걸 인지하고 아래와 같은 체크리스트를 확인 해 볼것!

- [ ] 인스턴스를 자동으로 프로비전 할 수 있는 방법이 있는가?

- [ ] 인스턴스를 가용영역 별로 분산 할 수 있는 방법이 있는가?

- [ ] 인스턴스 클러스터에 트래픽을 분산할 수 있는 방법이 적용 되어 있는가?

- [ ] 인스턴스 각자가 상태를 저장하지 않고 언제나 삭제 되고 추가 되도 무방한 상태가 맞는가?