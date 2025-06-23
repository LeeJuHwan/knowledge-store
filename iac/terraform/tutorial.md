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

#### **Optional**

* **JetBrains IDE Plugin Install**&#x20;
  * **Plugins - "Terraform and HCL" Install**
* **Visual Studio Code Plugin Install&#x20;**<mark style="color:green;">**`recommend 👍`**</mark>
  * **Plugins -  "Hashi Corp Terraform"**
  * **Editor - Auto save to formatter**
    * [follow article step](https://medium.com/nerd-for-tech/how-to-auto-format-hcl-terraform-code-in-visual-studio-code-6fa0e7afbb5e)
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

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

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



### Basic Terraform Structure

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

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

위 이론에서 학습 했듯이 AWS 프로바이더를 어디로 설정 하는지, 상태를 어디에 저장할 것인지 초기 작업을 진행 하고 테라폼이 수행 되기 위한 숨김 파일을 생성한다.


{% endstep %}

{% step %}
### Terraform Plan

```shell
terraform plan
```

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

AWS ACCESS KEY, SECRET ACCESS KEY를 환경 변수에 등록 되어 있다면 위 처럼 정상적으로 어떤 리소스를 생성할 것인지 또는 어떤 작업을 수행하는지에 대한 내용이 나온다.
{% endstep %}

{% step %}
### Terraform Apply

```sh
terraform apply
```

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
### AWS Console

Apply로 변경사항을 적용 했다면 콘솔에서 아래와 같이 생성된 VPC를 확인할 수 있다.

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>
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



## How to use HCL?

***

### Re use resources by variables

{% hint style="warning" %}
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



### Reference other resources

{% hint style="warning" %}
#### _"미리 정의한 리소스들의 정보를 재사용할 수 없을까?"_
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

<img src="../../.gitbook/assets/image (13) (1).png" alt="" data-size="original">

* [ ] &#x20;위 코드를 작성 해보고 테라폼 워크플로우를 따라 VPC ID를 출력 해보기

</details>

<details>

<summary>Summary</summary>

* output 블록을 통해서 출력을 정의할 수 있다.
* 출력값은 향후 서로 다른 테라폼 구성 간 값을 참조할 때 유용하게 사용할 수 있다.

</details>

***



### Resoucre Dependency

<figure><img src="../../.gitbook/assets/image (14) (1).png" alt=""><figcaption></figcaption></figure>

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
<pre class="language-hcl"><code class="lang-hcl">resource "aws_vpc" "main" {
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

<strong>resource "aws_subnet" "public_a" {
</strong>  vpc_id = aws_vpc.main.id
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
</code></pre>

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



### Use loop syntax

{% hint style="warning" %}
_**"비슷한 코드에서 살짝만 다른 리소스들 어떻게 편리하게 생성할 수 없을까?"**_
{% endhint %}

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

동일한 리소스에서 가용 영역만 다른 두 서브넷이 있다. 만약 4개, 8개 등 더 많은 서브넷을 생성 해야 한다면 `main.tf`가 굉장히 길어지기 때문에 이런 경우 반복문을 활용하면 쉽게 리소스를 정의할 수 있다.



> _**"변수와 Count 지시자를 활용한 반복문 사용 방법"**_

{% tabs %}
{% tab title="AS-IS(main.tf)" %}
```hcl

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
{% endtab %}

{% tab title="TO-BE(main.tf)" %}
```hcl
resource "aws_subnet" "public" {
  vpc_id = aws_vpc.main.id
  count = length(var.availability_zones)
  cidr_block = cidrsubnet(var.cidr_block, 8, count.index + 1)
  availability_zone = "ap-northeast-2${var.availability_zones[count.index]}"
  map_public_ip_on_launch = true

  tags = {
    Name = "${var.vpc_name}-public-subnet-${var.availability_zones[count.index]}"
  }
}
```
{% endtab %}

{% tab title="TO-BE(variables.tf)" %}
```hcl
variable "cidr_block" {
  description = "The cidr block of vpc"
}

variable "vpc_name" {
  description = "The name of vpc"
}

variable "availability_zones" {
  type = list(string)
  description = "az of subnets"
}
```
{% endtab %}

{% tab title="TO-BE(terraform.tfvars)" %}
```hcl
vpc_name = "oimarket-apne2"
cidr_block = "10.0.0.0/16"
availability_zones = [ "a", "b" ]
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
반복문을 사용하는 방법 중 `for_each` 도 있지만 현재 단계에서 `List(string)` 타입에 있는 변수의 개 수를 기준으로 반복 하는 방법을 사용 하였다.

* 이로인해 `["a", "b"]` 는 2개의 카운트를 갖을 수 있고 `index`는 0, 1을 갖고 있다.

또한, 서브넷의 CIDR Block을 할당 하기 위해 `cidr_block` 함수를 이용 하였는데 사용 방법은 이렇다.

`cidrsubnet(VPC CIDR, 추가 비트, 인덱스)`

* 사용 예시
* VPC: 192.168.0.0/24 (C class)
* subnets
  * 192.168.0.0/26 (0 \~ 63)
  * 192.168.0.64/26 (64 \~ 127)

`cidrsubnet(var.cidr_block, 2(24 + 2 = 26), count.index + 1)`
{% endhint %}

<details>

<summary>Hands-On</summary>

* [ ] 반복문을 사용하여 외부망 서브넷 2개를 서로 다른 가용영역에 생성 해보기
* [ ] Terraform Plan으로 리소스가 삭제 및 생성 계획인지 확인하기

</details>



이렇게 작성한 리소스를 AWS에 적용 하기 위해 <mark style="color:purple;">**`terraform plan`**</mark>을 입력하는 순간 테라폼은 별도의 리소스로 확인 하고 삭제 및 생성을 한다. 기존과 같았으면 "No Changes"가 나와야 하지만 그렇지 않다는 것을 아래 이미지로 확인할 수 있다.

<figure><img src="../../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

_**테라폼 구성 리팩터링 과정에서 발생할 수 있는 가장 흔한 이슈**_ 중 하나인데, 이는 똑같은 리소스 코드를 정의 했지만 위 이미지 처럼 삭제하고 다시 생성하는 것이다.

<details>

<summary>Summary</summary>

* count 지시자를 사용해서 반복문을 사용할 수 있다.
* 반복문을 사용하면 테라폼 구성을 더 효율적으로 만들 수 있다.

</details>



### State file control

{% hint style="warning" %}
_**"테라폼 구성을 변경할 때 같은 리소스이지만 자꾸 삭제 후 생성 하는데 어떻게 해야할까?"**_
{% endhint %}

"왜, 위와 같이 같은 리소스를 정의 했지만 삭제 후 재생성이 되었을까?" 이 질문에 대한 답을 찾기 쉬운 방법은 Plan에서 삭제 되는 출력과 생성 되는 출력을 1차적으로 비교해 볼 수 있다. 그럼 이와 같은 결과가 나오게 된다.

<details>

<summary>Console</summary>

![](<../../.gitbook/assets/image (4) (1) (1) (1) (1) (1).png>)

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

이렇게 바뀐 이유는 서브넷을 생성 하는 코드에서 명시적으로 `public_a`, `public_b` 를 정의 했지만 리팩터링 단계에서 이를 반복문으로 교체하며 배열의 인덱스로 참조 했기 때문이다. 관련 코드는 해당 페이지의 ["Use loop syntax"](https://1eejuhwany.gitbook.io/studylog/iac/terraform/tutorial#use-loop-syntax) 의 코드 블럭을 확인 해보면 된다.

> _**"어떻게 해결할까?"**_

테라폼은 기본적으로 상태파일을 관리하며 이 상태에 따라 프로바이더에 적용할 지 계획을 세우게 되는데, 그 상태를 관리할 수 있는 명령어를 통해 문제를 해결할 수 있다.



**Command**

```shell
terraform state mv aws_subnet.public_a aws_subnet.public\[0\]
terraform state mv aws_subnet.public_b aws_subnet.public\[1\]
```

위 커맨드를 입력한 후 다시 terraform plan을 입력 해보면 "No changes"가 나오는 확인할 수 있는데 결정적으로 리팩터링 과정에서 리소스의 명칭이 변경 된다면 이와 같은 이슈를 만날 수 있다는 것을 꼭 인지 해야한다.

이 이슈를 마주했을 땐 상태 파일을 제어 하여 해결할 수 있다는 것을 기억하고 Plan 출력을 무시하지 말아야한다.

{% tabs %}
{% tab title="AS-IS" %}
<mark style="color:purple;">**`terraform state list`**</mark>

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<mark style="color:purple;">**`terraform state show aws_subnet.public_a`**</mark>&#x20;

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>
{% endtab %}

{% tab title="TO-BE" %}
<mark style="color:purple;">**`terraform state plan`**</mark>

<figure><img src="../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

<mark style="color:purple;">**`terraform state list`**</mark>&#x20;

<figure><img src="../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>
{% endtab %}
{% endtabs %}

<details>

<summary>Hands-On</summary>

* [ ] 테라폼 상태 파일에서 같은 리소스를 사용하는 상태를 변경 해보기
* [ ] Terraform Plan으로 "No changes" 출력 확인 해보기

</details>



**추가적인 옵션 알아보기**

> _**"테라폼 상태에서 삭제하기"**_
>
> <mark style="color:purple;">**`terraform state rm`**</mark>

* 더이상 테라폼을 사용해서 리소스를 관리하지 않을 때, 이는 리소스를 제거하는 것이 아닌 상태 파일에서 제거 하는 것이기 때문에 더 이상 Plan 에서 changed를 관리 하고 싶지 않을 때 사용한다.

```sh
terraform state rm aws_subnet.public\[1\]
```

<figure><img src="../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
**상태파일 제거**

상태파일을 제거한 뒤 상태 목록을 확인 해보면 당연히 파일이 없고 plan을 입력했을 때 새롭게 생성되는 것을 확인할 수 있다.
{% endhint %}



> _**"테라폼 상태로 가져오기"**_
>
> <mark style="color:purple;">**`terraform import`**</mark>

* 리소스를 테라폼을 사용해 관리하기 위할 때, 이는 기존 콘솔에서 관리하던 리소스를 테라폼으로 관리해야 할 때 사용한다.

리소스를 가져올 때 입력해야 하는 값이 리소스 별로 다르기 때문에 항상 공식문서에서 어떤 값이 필요한지 확인 해보고 입력 해야한다. [서브넷 가져오기 공식문서](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/subnet#import)

```sh
terraform import aws_subnet.public\[1\] subnet-05dba3eb205ad6dd6
```

<figure><img src="../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
상태파일 가져오기

상태파일을 가져올 때 리소스 별로 필요한 항목이 다르기 때문에 공식문서를 확인해야 하며 위 서브넷을 가져올 땐 상태파일 이름, AWS Subnet ID가 필요했고 정상적으로 가져와진 후 plan에서 "no changes"가 나오면 정상이다.
{% endhint %}

<details>

<summary>Hands-On</summary>

* [ ] 테라폼 상태파일을 이용하여 서브넷 상태파일 하나를 삭제 한 뒤 plan에서 생성되는지 확인하기
* [ ] 테라폼 상태파일을 이용하여 위에서 삭제된 서브넷을 AWS 서브넷 아이디를 통해 가져온 뒤 plan에서 변경사항 없음이 제대로 나오는지 확인하기

</details>

<details>

<summary>Summary</summary>

* `terraform state` 명령을 사용해서 테라폼 상태파일을 조작할 수 있다.
* `terraform state mv` 명령을 사용해서 리소스 상태를 이전할 수 있다.
  * 코드 구성을 리팩터링 하면서 테라폼 구성 내에서 이름이 바뀌는 경우 많이 사용된다
* `terraform state rm` 명령을 사용해서 더이상 테라폼으로 리소스를 관리하지 않게 제거할 수 있다.
* `terraform import` 명령을 사용해서 리소스의 상태 정보를 테라폼 상태파일로 가져올 수 있다.

</details>



### Data types

{% hint style="warning" %}
_**"복잡한 타입의 변수를 사용 해서 효율적으로 리소스를 관리하고 싶은데?"**_
{% endhint %}

| Type       | Description           |
| ---------- | --------------------- |
| string     | string                |
| number     | integer               |
| bool       | true / false          |
| list       | arrays                |
| set        | unique value in array |
| map/object | key:value             |

_HCL도 코드의 영역이기 때문에 <mark style="color:red;">**가독성을 향상**</mark> 시키는 방법 중 다양한 자료구조를 활용해서 알고리즘과 같은 형식을 단순 변수 사용으로 변경 하는 과정_



> **list(object) type**&#x20;
>
> * <mark style="color:red;">**순서가 매우 중요한 자료형**</mark>

{% tabs %}
{% tab title="main.tf" %}
```hcl
resource "aws_subnet" "public" {
  vpc_id = aws_vpc.main.id
  count = length(var.public_subnets)
  cidr_block = var.public_subnets[count.index].cidr_block
  availability_zone = var.public_subnets[count.index].availability_zone
  map_public_ip_on_launch = true

  tags = {
    Name = var.public_subnets[count.index].name
  }
}
```
{% endtab %}

{% tab title="variables.tf" %}
```hcl
variable "public_subnets" {
  type = list(object({
    name = string
    availability_zone = string
    cidr_block = string
  }))
}
```
{% endtab %}

{% tab title="terraform.tfvars" %}
```hcl
public_subnets = [
  {
    name = "oimarket-apne2-public-subnet-a"
    availability_zone = "ap-northeast-2a"
    cidr_block = "10.0.1.0/24"
  },
  {
    name = "oimarket-apne2-public-subnet-b"
    availability_zone = "ap-northeast-2b"
    cidr_block = "10.0.2.0/24"
  }
]
```
{% endtab %}
{% endtabs %}

<details>

<summary>Hands-On</summary>

* [ ] <mark style="color:green;">**`List[object] type`**</mark>의 변수를 활용 하여 서브넷 리소스의 코드 가독성을 향상 시키기
* [ ] <mark style="color:purple;">`plan`</mark>에서 변경사항 없음이 제대로 나오는지 확인하기

</details>

{% hint style="danger" %}
**List type의 치명적인 단점**

_아래와 같이 서브넷의 순서가 변경 되었다면 테라폼은 변경이 있음을 알아 차리고 <mark style="color:purple;">`Plan`</mark>에서 삭제 후 재생성이 발생한다._

이유는 0번 인덱스에 있는 리소스가 "b"로 변경 되었기 때문에 상태 파일과 달라서 발생하는 상황이다.

```hcl
public_subnets = [
  {
    name = "oimarket-apne2-public-subnet-b"
    availability_zone = "ap-northeast-2b"
    cidr_block = "10.0.2.0/24"
  },
  {
    name = "oimarket-apne2-public-subnet-a"
    availability_zone = "ap-northeast-2a"
    cidr_block = "10.0.1.0/24"
  }
]
```

![](<../../.gitbook/assets/image (4) (1) (1) (1) (1).png>)
{% endhint %}

`List`의 치명적 단점으로 중요한 리소스는 `List`가 아닌 `Map`으로 관리하는 것이 안정적이다.



> **map(object) type**
>
> * <mark style="color:red;">**count 반복 형식에서 for\_each 반복 형식을 활용**</mark>

{% tabs %}
{% tab title="main.tf" %}
```hcl
resource "aws_subnet" "public" {
  vpc_id = aws_vpc.main.id
  for_each = var.public_subnets
  cidr_block = each.value.cidr_block
  availability_zone = each.value.availability_zone
  map_public_ip_on_launch = true

  tags = {
    Name = each.key
  }
}
```
{% endtab %}

{% tab title="variables.tf" %}
```hcl
variable "public_subnets" {
  type = map(object({
    availability_zone = string
    cidr_block = string
  }))
}
```
{% endtab %}

{% tab title="terraform.tfvars" %}
```hcl
public_subnets = {
  "oimarket-apne2-public-subnet-a" = {
    availability_zone = "ap-northeast-2a"
    cidr_block = "10.0.1.0/24"
  },
  "oimarket-apne2-public-subnet-b" = {
    availability_zone = "ap-northeast-2b"
    cidr_block = "10.0.2.0/24"
  }

}
```
{% endtab %}
{% endtabs %}

코드를 수정하고 다시 한 번 terraform plan을 입력 해보면 삭제 후 재생성 하는 이슈를 다시 만날 수 있다.

이에 대한 이유는 기존 List자료형은 상태 파일을 저장할 때 <mark style="color:red;">`aws_subnet_public[0]`</mark> 이런식으로 인덱스를 기반으로 해서 상태파일이 저장 되었었다.

하지만, Map은 Key를 기준으로 상태파일을 생성하기 때문에 <mark style="color:green;">`aws_subnet.public["oimarket-apne2-public-subnet-a"]`</mark> 로 변경 되어 리소스를 삭제 후 재생성한다.

이러한 이슈를 해결하는 방법을 우린 위에서 이미 배웠기 때문에 당황하지 않고 상태를 관리하면 된다.

```sh
terraform state mv aws_subnet.public\[0\] aws_subnet.public\[\"oimarket-apne2-public-subnet-a\"\]
terraform state mv aws_subnet.public\[1\] aws_subnet.public\[\"oimarket-apne2-public-subnet-b\"\]
```

상태파일을 변경했다면 <mark style="color:purple;">**Plan**</mark>을 확인 해보면 성공적으로 리팩터링이 완료된 것을 알 수 있다.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> _**"그렇다면 변수에 선언한 값의 순서를 바꿔도 동일할까?"**_

```hcl
public_subnets = {
  "oimarket-apne2-public-subnet-b" = {
    availability_zone = "ap-northeast-2b"
    cidr_block = "10.0.2.0/24"
  },
  "oimarket-apne2-public-subnet-a" = {
    availability_zone = "ap-northeast-2a"
    cidr_block = "10.0.1.0/24"
  }

}
```

:bulb: <mark style="color:blue;">**당연히**</mark>**&#x20;**<mark style="color:green;">**`Map[object]`**</mark><mark style="color:blue;">**의 Key 값으로 접근하기 때문에 동일하다.**</mark>

<details>

<summary>Hands-On</summary>

* [ ] <mark style="color:green;">**Map\[object] type**</mark> 의 변수를 활용 하여 서브넷 리소스의 순서와 관계 없이 항상 일정한 리소스를 생성하도록 수정하기
* [ ] 상태파일을 관리 하여  <mark style="color:purple;">`plan`</mark>에서 변경사항 없음이 제대로 나오는지 확인하기

</details>

<details>

<summary>Summary</summary>

* 테라폼은 string, number, bool, list, set, map 등 다양한 타입의 변수를 지원한다
* <mark style="color:green;">list(object)</mark>는 반복문을 사용할 때 <mark style="color:purple;">`count`</mark> 로 사용하며 인덱스로 접근하기 때문에 순서가 매우 중요하다
* <mark style="color:green;">map(object)</mark>는 반복문을 사용할 때 <mark style="color:purple;">`for_each`</mark>로 사용하며 순서에 영향을 받지 않는다

</details>



### Conditional

{% hint style="warning" %}
_**"조건에 따라 다른 리소스를 생성할 수 없을까?"**_
{% endhint %}

매번 리소스를 정의할 때 한가지 상황만 가지고 정의할 수 없다. 만약, 테라폼 구성을 할 때 "개발" 환경과 "운영" 환경의 차이가 있다면 어떻게 구성을 분리하여 리소스를 정의할 수 있을지 고민하게 된다.&#x20;

그럴 때 프로그래밍 단계에서의 꽃이라고 불리울 수 있는 조건을 이용한 리소스 정의 방식을 이용한다.



**NAT Gateway를 조건문으로 추가하는 시나리오 만들어보기**

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>senario</p></figcaption></figure>

실제 NAT Gateway를 생성 하는 시점 이후 부터 비용이 발생한다. 그렇기 때문에 프로비저닝 하지 않고 시나리오를 통해 어떤 방식으로 조건문을 활용하는지 알아보는 방식으로 학습한다.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> **조건문 사용 방법**
>
> <mark style="color:purple;">`condition ? true : false`</mark>

{% tabs %}
{% tab title="example" %}
```hcl
resource "aws_instance" "example" {
    instance_type = var.environment == "prod" ? "t3.medium" : "t3.micro"
    ami           = "ami-example12345678"
}
```
{% endtab %}
{% endtabs %}



NAT Gateway를 위 조건문을 활용해서 한 번 만들어본다면 이렇게 만들어볼 수 있다.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

_**여기서 잠깐!**_

> _**"Map(object) type 을 접근할 때 인덱스로 접근할 수 없을까?"**_**&#x20;🤔**

서브넷 아이디는 Map(object)이기 때문에 인덱스로 접근할 수 없다. 그렇다면 어떻게 NAT Gateway가 인터넷망으로 통신 할 수 있도록 외부망 서브넷과 연결할 수 있을까? 아래 테라폼 공식문서에서 제시하는 방향대로 특정 함수를 이용하면 된다.

{% hint style="info" %}
**Map(object) type 을 List(object) type 으로 형 변환 하기**

![](<../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png>)

[테라폼 공식문서에서 확인하기](https://developer.hashicorp.com/terraform/language/functions/tolist#examples)
{% endhint %}



**Code**

{% tabs %}
{% tab title="main.tf" %}
```hcl
resource "aws_subnet" "private" {
  for_each                = var.private_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = each.value.cidr_block
  availability_zone       = each.value.availability_zone
  map_public_ip_on_launch = false

  tags = {
    Name = each.key
  }
}

resource "aws_eip" "nat_ips" {
  count = var.private_subnets != {} ? length(var.private_subnets) : 0
}

resource "aws_nat_gateway" "gateways" {
  count = var.private_subnets != {} ? length(var.private_subnets) : 0

  allocation_id = aws_eip.nat_ips[count.index].id
  subnet_id     = tolist(values(aws_subnet.public))[count.index].id
}
```
{% endtab %}

{% tab title="variables.tf" %}
```hcl
variable "private_subnets" {
    type = map(object({
    availability_zone = string
    cidr_block = string
  }))
}
```
{% endtab %}

{% tab title="terraform.tfvars" %}
```hcl
private_subnets = {
  "oimarket-apne2-private-subnet-b" = {
    availability_zone = "ap-northeast-2b"
    cidr_block = "10.0.11.0/24"
  },
  "oimarket-apne2-private-subnet-a" = {
    availability_zone = "ap-northeast-2a"
    cidr_block = "10.0.2.0/24"
  }
}
```
{% endtab %}
{% endtabs %}

내부망에 정의된 서브넷이 없다면 NAT Gateway를 생성 하지 않고 있다면 생성하는 내용이다. 이렇게 특정 상황일 때 조건문을 통해 리소스를 정의하면 테라폼 구성을 관리하기 편리하다.

위에 있듯이 NAT Gateway를 사용하기 위해서 공용 IP가 필요하다. NAT가 외부망으로 나가기 위해 공용 IP로 나가야 하며, 이 때 SNAT를 통해 나가기 때문이다.

* [SNAT 와 DNAT 개념 더 알아보기](https://zigispace.net/1121)



위 코드를 작성 했다면 <mark style="color:purple;">**`Plan`**</mark>을 살펴보자. 그렇다면 생성 항목이 6개가 나온다면 정상이다. 하지만, 이 코드를 AWS에 반영하여 프로비저닝 하진 않고 실행 계획에서 어떤 리소스가 생성 되는지만 살펴볼 것이다.

{% hint style="info" %}
**생성 리소스 대상**

* Elastic IP: 2 개
* NAT Gateway: 2 개
* Private subnet 2 개
{% endhint %}

<details>

<summary>Summary</summary>

* 테라폼에서 조건문은 <mark style="color:purple;">**`condition ? true : false`**</mark> 형태로 사용할 수 있다.

</details>







