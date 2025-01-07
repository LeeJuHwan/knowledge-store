---
description: Terraform tutorial
---

# Tutorial

테라폼 시작하기

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

#### **Optional**

* **JetBrains IDE Plugin Install**&#x20;
  * **Plugins - "Terraform and HCL" Install**
* **Git repository ignore**&#x20;
  * **"intellij", "Terraform"**

{% embed url="https://www.toptal.com/developers/gitignore" %}



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



### Basic Terraform Structure Hands-On

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

{% hint style="info" %}
`region` 과 같이 중요한 정보는 하드코딩 하는 것이 아닌 환경 변수를 이용하는 것을 권장한다.
{% endhint %}
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

{% hint style="info" %}
resource의 `main`은 코드 레벨에서 참조하는 명칭이며 AWS에서 사용하는 이름은 `tags`의 `Name`을 이용해야한다.

Name 컨벤션은 다양하게 이용할 수 있지만 현재 단계에선 "회사명칭-리전" 으로 사용한다.
{% endhint %}
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

<summary>Hands-On</summary>

* [ ] 튜토리얼에 사용 될 기본 VPC를 테라폼 코드를 이용 하여 생성하기
* [ ] AWS Console에서 제대로 생성 되었는지 확인하기

</details>

<details>

<summary>Summary</summary>

* `providers.tf` 파일에 인프라 리소스가 구성 될 환경을 정의한다.
* `main.tf` 파일에 구성 하고자 하는 인프라 리소스를 정의 한다.
* 테라폼 워크플로우인 `terraform init` -> `terraform plan` -> `terraofrm apply` 의 순서대로 진행한다.

</details>



### Use HCL Variables Syntax

***

> #### _**"확장 가능한 테라폼 구성을 만드는 첫 번째 요소 "변수"**_

{% hint style="info" %}
#### Refactoring: re use resources by variables

_**"변수 값은 어디에 저장 할까?"**_
{% endhint %}

다양한 방법 중 가장 대표적으로 사용 되는 방법인 "`환경변수`", "`var`" 등을 사용하게 되면 코드로 남지 않는 단점이 존재한다.

변수를 관리할 수 있는 파일인 `terraform.tfvars` 을 생성 하여 해당 파일에서 관리할 수 있도록 구성한다.

{% tabs %}
{% tab title="main.tf" %}
```hcl
resource "aws_vpc" "main" {
  cidr_block = var.cidr_block

  tags = {
    Name = var.vpc_name
  }
}
```
{% endtab %}

{% tab title="variables.tf" %}
```hcl
variable "cidr_block" {
  description = "The cidr block of vpc"
}

variable "vpc_name" {
  description = "The name of vpc"
}
```

{% hint style="info" %}
변수를 활용할 때 _**description**_ 작성을 귀찮게 생각 하지 말고 다른 누군가와 협업 한다고 생각 하며 꼭 작성 할 것

* 만약 variables.tf 파일만 생성 했다면 `terraform plan` 단계에서 해당 변수에 대한 값을 입력 하도록 안내한다. 하지만, 이렇게 관리할 시 위에서 언급 했던 코드로 남지 않는 상황이 발생 하여 협업하는 데 다른 변수를 사용할 수 있기 때문에 `terraform.tfvars`에서 값을 할당하여 사용하는 것을 권장한다.
{% endhint %}
{% endtab %}

{% tab title="terraform.tfvars" %}
```hcl
vpc_name = "oimarket-apne2"
cidr_block = "10.0.0.0/16"

```
{% endtab %}
{% endtabs %}

<details>

<summary>Hands-On</summary>

* [ ] 기존 리소스에서 하드코딩 된 중요 정보를 변수화 하여 파일로 관리하기
* [ ] 테라폼 Plan을 통해 "No Changes" 가 나오는지 확인하기

</details>



> #### _**"확장 가능한 테라폼 구성을 만드는 두 번째 요소 "출력"**_

{% hint style="info" %}
#### Refactoring: reference other resources

**"**_**미리 정의한 리소스들의 정보를 재사용할 수 없을까?**_**"**
{% endhint %}

{% tabs %}
{% tab title="outputs.tf" %}
```hcl
output "vpc_id" {
    value = aws_vpc.main.id
}
```

{% hint style="info" %}
`main.tf` 에 위치한 `resource` 명칭과 코드 레벨에서 참조하는 명칭을 `output`의 값으로 할당하게 되면 해당 리소스가 참조 되어 필요한 값을 출력할 수 있다.
{% endhint %}
{% endtab %}
{% endtabs %}

<details>

<summary>Hands-On</summary>

<img src="../../.gitbook/assets/image.png" alt="" data-size="original">

* [ ] &#x20;위 코드를 작성 해보고 테라폼 워크플로우를 따라 VPC ID를 출력 해보기

</details>

<details>

<summary>Summary</summary>

* output 블록을 통해서 출력을 정의할 수 있다.
* 출력값은 향후 서로 다른 테라폼 구성 간 값을 참조할 때 유용하게 사용할 수 있다.

</details>

***



#### Resoucre Dependency

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

VPC 의 기본 골조를 갖췄으니 인터넷 망과 통신할 수 있는 IGW를 생성 하면서 사전에 만들어진 VPC가 먼저 생성 되어 있어야 하는 상황에서 의존성을 기반으로 리소스를 생성한다.

{% tabs %}
{% tab title="AS-IS(main.tf)" %}
```hcl
resource "aws_vpc" "main" {
  cidr_block = var.cidr_block

  tags = {
    Name = var.vpc_name
  }
}
```
{% endtab %}

{% tab title="TO-BE(main.tf)" %}
```hcl
resource "aws_vpc" "main" {
  cidr_block = var.cidr_block

  tags = {
    Name = var.vpc_name
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "${var.vpc_name}=igw"
  }
}
```
{% endtab %}
{% endtabs %}



> _**"암시적 의존성 vs 명시적 의존성"**_

{% tabs %}
{% tab title="암시적 의존성" %}
```hcl
resource "aws_vpc" "main" {
  cidr_block = var.cidr_block

  tags = {
    Name = var.vpc_name
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "${var.vpc_name}=igw"
  }
}
```
{% endtab %}

{% tab title="명시적 의존성" %}
```hcl
resource "aws_vpc" "main" {
  cidr_block = var.cidr_block

  tags = {
    Name = var.vpc_name
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  depends_on = [aws_vpc.main]
  
  tags = {
    Name = "${var.vpc_name}=igw"
  }
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
리소스가 자동으로 참조 하여 의존성을 띄기 때문에 실무 환경에서는 암시적 의존성을 선호 한다.
{% endhint %}

<details>

<summary>Hands-On</summary>

* [ ] 암시적 의존성 방식으로 리소스를 정의 하고 해당 VPC와 Attatch할 IGW 만들기
* [ ] AWS Console에서 제대로 생성 되었는지 확인하기

</details>



학습한 의존성을 바탕으로 VPC 대역에대 할당할 수 있는 IP 서브넷팅을 위한 외부망 서브넷을 생성 한다.

| AZ              | Host/Network |
| --------------- | ------------ |
| ap-northeast-2a | 10.0.1.0/24  |
| ap-northeast-2b | 10.0.2.0/24  |

{% tabs %}
{% tab title="main.tf" %}
```hcl
resource "aws_vpc" "main" {
  cidr_block = var.cidr_block

  tags = {
    Name = var.vpc_name
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "${var.vpc_name}-igw"
  }
}

resource "aws_subnet" "public_a" {
  vpc_id = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
  availability_zone = "ap-northeast-2a"
  map_public_ip_on_launch = true

  tags = {
    Name = "${var.vpc_name}-public-subnet-a"
  }
}

resource "aws_subnet" "public_b" {
  vpc_id = aws_vpc.main.id
  cidr_block = "10.0.2.0/24"
  availability_zone = "ap-northeast-2b"
  map_public_ip_on_launch = true

  tags = {
    Name = "${var.vpc_name}-public-subnet-b"
  }
}
```

{% hint style="info" %}
옵션 알아보기

* map\_public\_ip\_on\_lanuch: EC2를 생성할 때 자동으로 Public IP Allocate 할지 여부
{% endhint %}
{% endtab %}
{% endtabs %}

<details>

<summary>Hands-On</summary>

* [ ] 위 리소스 정의표에 맞춰 외부망 서브넷 2개를 서로 다른 가용영역에 생성 해보기
* [ ] AWS Console에서 제대로 생성 되었는지 확인하기

</details>

<details>

<summary>Summary</summary>

* 서로 다른 리소스를 참조 하며 먼저 생성 된 후 리소스를 만들어야 하는 의존성을 관리할 수 있다.
* 의존성은 대표적으로 암묵적과 명시적 의존성이 있으며 현업에서 암묵적 의존성 방식을 선호한다.

</details>



{% hint style="info" %}
#### Refactoring: use loop syntax

_**"비슷한 코드에서 살짝만 다른 리소스들 어떻게 편리하게 생성할 수 없을까?"**_
{% endhint %}

writing...







