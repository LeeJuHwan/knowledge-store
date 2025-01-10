---
description: Bastion Host 대체
---

# SSM: Session Manager

## SSM Session Manager: Bastion Hosts

> "인프라를 구성 할 때 Bastion host먼저 제거 할 것"

인스턴스를 프로비저닝 한 Bastion host는 우선 서버 유지 비용, 이중화(HA) 되지 않은 유일한 접근 통로이다. 서버가 급하게 다운 되어 들어가야 할 문이 잠겼다면 모든 개발자들은 문이 열릴 때 까지 기다려야 할까?

그렇게 외부 감사 로깅을 하며 보안을 신경 쓴 서버에 이중화 작업과 적절한 스펙을 부여 해야한다. 또한, 접근 하려 하는 모든 개발자들에게 Pem키 파일을 전달 해야 하고 이 파일이 외부에 유출 되었을 때 일괄적으로 변경 해야 한다.

간략한 단점 몇 가지만으로 Bastion host가 인스턴스로 구축 되어있을 때 불편함을 알아볼 수 있는데 이 것을 대체 하기 위해 AWS에서 **Sesion Manager**를 제공한다.



Session Manager는 IAM Role 기반으로 접근을 허용하고 블럭 할 수 있다. 또한, 감사 로깅도 CloudWatch + S3로 관리가 가능하며 세션에 접근한 유저의 기록을 JSON 데이터로 보관 하고 있다.&#x20;

Bastion 서버에서 직접 구축한 로깅만큼 이쁘게 나오진 않지만 사용자의 커맨드와 결과를 직접 확인 할 수 있는 장점이 있다.

여기서 더 해보자면, Session Manager는 EC2 프로비저닝 해서 할당 받은 리소스의 비용을 받지 않는다.

그렇다면 Session Manager가 정상적으로 작동하기 위해 EC2가 갖춰야 할 Policy는 무엇일까?

1. [AmazonSSMManagedInstanceCore](https://us-east-1.console.aws.amazon.com/iam/home?region=ap-northeast-2#/policies/details/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FAmazonSSMManagedInstanceCore)
2. [CloudWatchLogsFullAccess](https://us-east-1.console.aws.amazon.com/iam/home?region=ap-northeast-2#/policies/details/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FCloudWatchLogsFullAccess)

프로덕션 서버에 SSM Agent가 설치 되어있으며, 관리자 작업 컴퓨터에 AWS shell이 설치 되어있다고 가정 한다.

위 두 가지의 정책이 적용 된 EC를 프로비저닝 했을 때 관리자는 아래와 같은 명령어로 접근 할 수 있다.



#### Session Manager가 접근하기 위한 EC2 필수 설정

1. SG-Outbound: HTTPS allows any(0.0.0.0) 443 port 허용
   * SSM Agent와 통신을 이루기 위한 보안그룹 설정
2. Public IP enable
3. SSM Agent가 사전 설치 된 AMI 또는 수동 설치 된 환경

<details>

<summary>SSM Agent 사전 설치 된 AMI 목록</summary>

[공식문서](https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/ami-preinstalled-agent.html)

* 2017년 9월 이후의 Amazon Linux 1 Base AMI

- Amazon Linux 2
- Amazon Linux 2 ECS 최적화 기본 AMIs
- Amazon Linux 2023(AL2023)
- Amazon EKS 최적화 Amazon Linux AMIs
- macOS 10.14.x(Mojave), 10.15.x(Catalina), 11.x(Big Sur), 12.x(Monterey), 13.x(Ventura), 14.x(Sonoma)
- SUSE Linux Enterprise Server(SLES) 12 및 15
- Ubuntu Server 16.04, 18.04, 20.04 및 22.04
- Windows Server 2008-2012 R2 AMIs는 2016년 11월 이후에 게시되었습니다.
- Windows Server 2016, 2019 및 2022(Nano 버전 제외)

</details>





```
aws ssm start-session --target "{InstanceId}"
```

<details>

<summary>Session history</summary>

<img src="../../.gitbook/assets/image (20).png" alt="" data-size="original">

</details>

<details>

<summary>CloudWatch Logs</summary>

![](<../../.gitbook/assets/image (21).png>)

</details>
