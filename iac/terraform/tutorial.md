---
description: Terraform tutorial
---

# Tutorial

## 테라폼 시작하기

테라폼을 시작하기에 앞서 해당 챕터에서는 이런 내용을 다룹니다.

> _**"테라폼 구성을 작성하는 방법"**_

간단하게 AWS VPC를 생성하는 테라폼 구성으로 시작하여 해당 구성을 개선합니다.

테라폼 버전은 **1.9.5 버전**을 이용 하며 설치 방법과 문법을 해당 챕터에서 다루게 됩니다.

### Install with Mac OS

테라폼을 설치하기 위해 테라폼을 직접적으로 설치할 수 있지만 테라폼의 다양한 버전을 관리할 수 있는 오픈소스를 활용합니다.

```sh
brew install tfenv

tfenv install 1.9.5
tfenv use 1.9.5 
```

#### Pre Setting

***

AWS VPC 를 생성해야 하기 때문에 권한이 있는(자격 증명) AWS 인증 정보가 필요합니다.

```sh
export AWS_ACCESS_KEY_ID={secret}
export AWS_SECRET_ACCESS_KEY={secret}
```



### Terraform Config

테라폼 구성을 작성하기 위해 알아야 할 파일 구성과 워크 플로우 참고

[https://developer.hashicorp.com/terraform/language](https://developer.hashicorp.com/terraform/language)

***

#### Files

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
**인프라 리소스가 구성될 환경**

* `providers.tf` : AWS, GCP, Azure 등 퍼블릭 클라우드 서비스 중 어떤 것을 이용할지 정의 하는 파일
{% endhint %}

{% hint style="info" %}
**구성 하고자 하는 인프라 리소스**

* `variables.tf`: 변수들에 대한 정의, 변수에 대한 Description, Type 정보
* `outputs.tf`: 리소스를 구성하면서 만들어지는 상태들의 정보

구성해야 할 리소스가 많은 경우 `main.tf` 파일에 모두 담고 있으면 가독성이 많이 떨어지기 때문에 의미에 맞게 파일을 분리한다.

* `network.tf`: 서비스가 사용할 네트워크(VPC, SG 등)
* `storage.tf`: EBS, S3 등 다양하게 사용할 네트워크
{% endhint %}

{% hint style="info" %}
**구성된 인프라 리소스의 상태 저장 방법**

* `backend.tf`: local, S3, GCS 등 저장할 수 있는 위치는 다양하기 때문에 상태를 어디에 저장할 것인지 관리
{% endhint %}



#### Workflows

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
**Write | Init**

```sh
terraform init
```

1. `providers.tf` 에 정의된 프라이바이더들의 코드를 로컬 환경에 다운로드
2. `backend.tf` 에 정의된 백엔드를 사용하도록 환경 구성
   * 아무런 파일이 없다면 "local"을 기본 값으로 사용
   * "S3"를 사용한다면 상태 파일 생성, 키 생성, 버킷 정보 확인

테라폼을 적용하기 위한 전처리 단계에 해당
{% endhint %}

{% hint style="info" %}
**Plan**

```
terraform plan
```

정의되어 있는 테라폼 구성을 바탕으로 실제 어떤 변경사항들이 발생하는지 확인하는 과정
{% endhint %}

{% hint style="info" %}
**Apply**

```
terraform apply
```

테라폼 구성에 정의되어 있는 인프라 리소스를 실제 인프라에 반영하고 상태 파일을 업데이트 하는 과정

* 실제 Apply가 완료 되면 AWS 와 같은 프로바이더에 변경이 일어남
{% endhint %}

<details>

<summary>Summary</summary>

**테라폼 구성은 주로 `providers.tf`, `backend.tf`, `main.tf`, `outputs.tf`, `variables.tf` 등으로 구성**

* `providers.tf`: 인프라 리소스가 구성될 환경(AWS, GCP, 쿠버네티스 등)을 정의
* `main.tf`: 구성하고자 하는 인프라 리소스를 정의
* `backend.tf`: 구성된 인프라 리소스의 상태 저장 방법(로컬, S3, GCS 등)을 정의



**테라폼 워크 플로우는 `terraform init`, `terraform plan`, `terraform apply`로 구성**

* `terraform init`: 프로바이더의 코드와 백엔드 환경 구성
* `terraform plan`: 테라폼에 정의된 리소스의 상태를 바탕으로 어떤 변경사항들이 발생할지 정보 제공
* `terraform apply`: 테라폼에 정의된 리소스의 상태를 실제 인프라에 반영

</details>



### Basic Terraform Structure Settings

***

{% stepper %}
{% step %}
### AWS VPC 기본 구성하기&#x20;

{% tabs %}
{% tab title="providers.tf" %}
```hcl
provider "aws" {
  region = "ap-northeast-2"
}

terraform {
  required_version = "= 1.9.5"
}
```
{% endtab %}

{% tab title="main.tf" %}
```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "oimarket-apne2"
  }
}
```

resource의 `main`은 코드 레벨에서 참조하는 명칭이며 AWS에서 사용하는 이름은 `tags`의 `Name`을 이용해야한다.

Name 컨벤션은 다양하게 이용할 수 있지만 현재 단계에선 "회사명칭-리전" 으로 사용한다.
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### Terraform Init

```sh
terraform init
```

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

위 이론에서 학습 했듯이 AWS 프로바이더를 어디로 설정 하는지, 상태를 어디에 저장할 것인지 초기 작업을 진행 하고 테라폼이 수행 되기 위한 숨김 파일을 생성한다.


{% endstep %}

{% step %}
### Terraform Plan

```shell
terraform plan
```

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

AWS ACCESS KEY, SECRET ACCESS KEY를 환경 변수에 등록 되어 있다면 위 처럼 정상적으로 어떤 리소스를 생성할 것인지 또는 어떤 작업을 수행하는지에 대한 내용이 나온다.
{% endstep %}

{% step %}
### Terraform Apply

```sh
terraform apply
```

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
### AWS Console

Apply로 변경사항을 적용 했다면 콘솔에서 아래와 같이 생성된 VPC를 확인할 수 있다.

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>
{% endstep %}
{% endstepper %}

<details>

<summary>Summary</summary>

* `providers.tf` 파일에 인프라 리소스가 구성 될 환경을 정의한다.
* `main.tf` 파일에 구성 하고자 하는 인프라 리소스를 정의 한다.
* 테라폼 워크플로우인 `terraform init` -> `terraform plan` -> `terraofrm apply` 의 순서대로 진행한다.

</details>



