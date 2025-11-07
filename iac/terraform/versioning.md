# Versioning

## 원격 상태 파일 저장소

{% hint style="info" %}
**테라폼 상태 파일이란?**

테라폼이 관리하는 인프라의 현재 상태를 저장하는 파일 <mark style="color:blue;">**terraform.tfstate**</mark>

JSON 형태로 리소스 정보, 프로바이더 정보 등 여러 섹션으로 구성 되어 있으며 <mark style="color:red;">**절대 수동으로 파일을 수정하면 안되고**</mark> <mark style="color:purple;">**terraform state**</mark> 명령을 사용해서 조작해야 한다.

<img src="../../.gitbook/assets/image (37).png" alt="" data-size="original">
{% endhint %}

### Backend Block

{% hint style="warning" %}
_**"테라폼 상태 파일을 어디에 저장하고 관리할지 설정"**_
{% endhint %}

기본적으로 백엔드 블록은 <mark style="color:red;">local</mark> 을 사용하도록 설정 되어 있고, 협업을 위해 해당 챕터에서는 <mark style="color:red;">S3</mark>를 사용한다. 테라폼 공식문서는 아래 이미지 처럼 백엔드 블록에서 어떤 저장소를 사용할 수 있는지 명시 되어있다.

<div data-full-width="false"><figure><img src="../../.gitbook/assets/image (38).png" alt="" width="361"><figcaption></figcaption></figure></div>

> "_**Local backend block을 사용하는 도중 작업 하는 환경이 바뀐다면?**_"

집에서 개인 맥북으로 작업 하다가 맥북을 두고 외부 일정을 나가서 테라폼을 작성 한다면 모두 새로 생성하는 것 처럼 인식한다.

또, 내가 아닌 다른 사람이 작업 하면 상태 파일을 Git 으로 관리하거나 파일을 전달해야 하는걸까?

> _**"원격 저장소 구성의 장점"**_

원격 저장소에 있는 상태 파일은 계속 동기화 되어 있기 때문에 작업자 A가 어디 까지 작업을 했는지에 대해 다른 작업자가 쉽게 알 수 있다.

> _**"같은 파일을 동시에 작업한다면?"**_

원격 저장소 내에 A 작업자가 SG를 작업 하고 있고 다른 작업자가 해당 SG 파일에 포트를 추가 한다거나 등 동시 작업을 수행 할 경우 충돌이 발생한다.

이걸 방지하기 위해 1 명만 작업할 수 있도록 Lock이 필요하다.

{% hint style="info" %}
DynamoDB State Locking

<img src="../../.gitbook/assets/image (39).png" alt="" data-size="original">
{% endhint %}

**Remote Backend Example**

<figure><img src="../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

> _**"최초의 원격 저장소를 위한 테라폼 구성은 어디에 저장할까?"**_

아래의 방식을 선호에 따라 사용하면 되며 각 조직에서 추구하는 시스템을 따르면 된다. 해당 챕터에서는 AWS에서 잘 만들어준 GUI 환경에서 생성하기 위해 콘솔 방식을 선택했다.

1. 로컬 테라폼 작성
2. 콘솔(S3, DynamoDB Table) 작업 후 공유

**AWS Console 환경에서 원격 저장소 구성을 위한 리소스 생성하기**

{% stepper %}
{% step %}
#### S3 Bucket 생성

S3 Bucket을 생성 할 때 원하는 <mark style="color:red;">**이름만 작성한 뒤 모두 기본값**</mark>을 사용하여 생성한다.

<figure><img src="../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
#### Dynamo DB Table 생성

Table, Partition Key(String type) 만 작성한 뒤 모두 기본값을 사용하여 생성한다.

{% hint style="info" %}
**이 때, 테라폼 공식문서에서 안내 하듯이 Partition Key는 LockID (String) 으로 생성해야한다.**
{% endhint %}

<figure><img src="../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
#### Terraform Init

백엔드 구성 파일이 없는 경우 기본값으로 "local"을 사용했지만 현재 S3를 사용하기 때문에 새롭게 terraform init을 통해 원격 저장소를 구성한다. 이 때, 기본 로컬 백엔드를 사용하고 있었다면 현재 구성을 변경할 것인지 물어보는데 "yes"라고 하면 정상적으로 변경이 완료된다.

<figure><img src="../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
#### Terraform Lock

테스트를 위해 기본 <mark style="color:blue;">**`config.yaml`**</mark> 파일에서 SG 구성의 HTTP 허용 포트를 443 -> 543으로 변경한 후 <mark style="color:purple;">**terraform apply**</mark>를 하고 있다고 가정한다.

또 다른 사용자가 위 상황을 인지하지 못한 채 새로운 리소스를 적용하여 apply를 시도한다면 아래 콘솔 에러처럼 Lock 상태임을 확인할 수 있다.

<figure><img src="../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>
{% endstep %}
{% endstepper %}

<details>

<summary>Summary</summary>

* <mark style="color:blue;">backend.tf</mark> 파일을 만들고 backend 블록을 만들어서 상태 파일을 어떻게 관리할지 정의할 수 있습니다.
* 원격 저장소를 통해 상태 파일을 관리하면 다수의 인원이 협업할 때 효과적으로 할 수 있습니다.
* 원격 저장소를 사용할 때는 동시에 상태 파일에 작업하지 않도록 상태 잠금도 필요 합니다.

</details>

### Remote Data

{% hint style="warning" %}
_**"외부에서 관리하는 인프라 데이터를 현재의 테라폼 구성에서 재사용 하는 기능"**_
{% endhint %}

> _**"기존에 만든\*\*\*\*****&#x20;**<mark style="color:green;">**VPC**</mark>**에**\*\* \*\*<mark style="color:green;">**EC2**</mark>**를 생성하려면?"**_

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<mark style="color:green;">subnet id</mark>, <mark style="color:green;">security group id</mark> 의 값을 넣기 위해 아래 이미지 처럼 AWS Console에서 직접 값을 복사할 수도 있다.

하지만, 기존에 만들었던 <mark style="color:green;">VPC</mark> 정보에 모두 포함되어있다. 이 것을 사용하면 되지 않을까?

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
<mark style="color:green;">**terraform remote staste**</mark>**&#x20;사용하기**

S3 Backend 구성을 통해 원격 저장소에 상태 파일을 관리 했다면 해당 상태 파일이 출력하는 값들을 외부 모듈에서 사용할 수 있다.
{% endhint %}

{% tabs %}
{% tab title="output.tf" %}
```hcl
output "vpc_id" {
  value = module.vpc.vpc_id
}

output "subnets" {
  value = module.vpc.subnets
}

output "security_groups" {
  value = module.vpc.security_groups
}
```

{% hint style="info" %}
modules 에 있는 vpc output과 vpc 디렉터리에 있는 oimarket-apne2의 output이 동일해야한다.
{% endhint %}
{% endtab %}

{% tab title="datasources.tf" %}
```hcl
data "terraform_remote_state" "vpc" {
  backend = "s3"

  config = {
    bucket = "oimarket-terraform-study-remote-state"
    key    = "vpc/oimarket-apne2/terraform.tfstate"
    region = "ap-northeast-2"
  }
}
```
{% endtab %}
{% endtabs %}

<mark style="color:purple;">output</mark>을 적용 했다면 <mark style="color:purple;">terraform pla</mark>n으로 출력 부분이 변경 되었는지 확인하고 적용한다.

그 후 해당 출력 값을 사용할 수 있는 리소스를 정의 해서 참조하면 되는데, 해당 챕터는 <mark style="color:green;">Bastion EC2</mark>를 생성한다.

> "외부 모듈에서 원격 저장소 데이터 사용하기"

EC2 Bastion을 생성하기 위해 작성한 테라폼 구성은 위 <mark style="color:blue;">datasources</mark> 값을 기준으로 작성한다.

{% tabs %}
{% tab title="main.tf" %}
```hcl
resource "aws_instance" "bastion" {
  ami           = "ami-0a998385ed9f45655"
  instance_type = "t3.micro"
  subnet_id     = data.terraform_remote_state.vpc.outputs.subnets["oimarket-apne2-public-subnet-a"].id
  vpc_security_group_ids = [
    data.terraform_remote_state.vpc.outputs.security_groups["oimarket-apne2-permit-ssh-security-group"].id
  ]

  tags = {
    Name = "oimarket-apne2-bastion"
  }
}
```
{% endtab %}

{% tab title="datasources.tf" %}
```hcl
data "terraform_remote_state" "vpc" {
  backend = "s3"

  config = {
    bucket = "oimarket-terraform-study-remote-state"
    key    = "vpc/oimarket-apne2/terraform.tfstate"
    region = "ap-northeast-2"
  }
}
```
{% endtab %}
{% endtabs %}

<details>

<summary>Summary</summary>

* <mark style="color:green;">data</mark>와 <mark style="color:green;">terraform\_remote\_state</mark> 지시자를 사용해서 원격 저장소에 위치한 상태 파일의 출력들을 사용할 수 있다.
* 이를 통해서 리소스 참조를 더 효율적으로 할 수 있습니다.

</details>

### Provider Version Spec

{% hint style="warning" %}
_**"버전 관리를 통해 안정성과 협업 중요시하게 여기기"**_
{% endhint %}

> **"Terraform version"**

<figure><img src="../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

> **"Provider version"**

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
**버전 연산자**

> _<mark style="color:purple;">**`~>`**</mark>_

* 최소 버전과 최대 버전을 함께 지정하며 마이너 버전만 제한하는 연산자
* 5.82.0 -> 5.82.x 까지 허용

> _<mark style="color:purple;">**`>=`**</mark>_

* 최소 버전만 지정하며 최대 버전이 없는 연산자
* 5.82.0 -> 5.82.0 이상 모든 버전 가능
{% endhint %}

**위 처럼&#x20;**<mark style="color:blue;">**providers.tf**</mark>**&#x20;에 특정 버전을 명시하지 않는다면&#x20;**<mark style="color:purple;">**terraform init**</mark>**&#x20;단계에서 가장 최신 버전을 다운로드 받아서 사용하게 된다.**

> _**"버전 충돌이 발생한 경우 어떻게 해결 해야할까?"**_

{% tabs %}
{% tab title="providers.tf" %}
```hcl
provider "aws" {
  region = "ap-northeast-2"
}

terraform {
  required_version = "= 1.9.5"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "= 5.82.0"
    }
  }
}
```
{% endtab %}
{% endtabs %}

해당 버전은 hashcorp/aws 버전이 "**5.82.0**"만 허용이 된 상황이다. 하지만, 현재 테라폼 구성에 hashcorp/aws 버전이 **5.82.2**이면 당연히 **충돌**이 발생할텐데, 이 때 어떻게 해결해야 할까?

{% stepper %}
{% step %}
#### Terraform plan

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
#### Terraform init -uprade

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
{% endstep %}
{% endstepper %}

<details>

<summary>Summary</summary>

* <mark style="color:blue;">providers.tf</mark> 에서 테라폼과 각 프로바이더의 버전을 지정할 수 있습니다.할 수 있다.
* 테라폼 버전은 <mark style="color:purple;">required\_version</mark> 지시자로 지정할 수 있습니다.
* 각 프로바이더의 버전은 <mark style="color:purple;">required\_providers</mark> 블록 밑에 version 지시자로 지정할 수 있습니다.

</details>
