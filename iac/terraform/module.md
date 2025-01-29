# Module

## 테라폼 구성 리팩터링

튜토리얼을 진행 하며 퍼블릭 클라우드에 프로비저닝 하는 방식을 살펴 보았다면 해당 챕터에서는 비효율적인 부분을 개선합니다.



### Local scope variables

{% hint style="warning" %}
_**"불필요한 반복을 줄이고 변수에 있는 값을 최대한 재사용 할 수 없을까?"**_
{% endhint %}

이전에 진행 했던 <mark style="color:blue;">**Tutorial**</mark> 챕터에서 서브넷과 IGW, NAT Gateway 등을 생성할 때 변수를 분리하여 내부망과 외부망을 분리 시켰다.&#x20;

하지만, 사용하는 값이 비슷했기 때문에 가독성을 더 향상 시키기 위해 변수를 활용 하여 서브넷 그룹을 하나로 묶는 방법을 선택할 수 있다.



**Examples**

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

{% tabs %}
{% tab title="AS-IS" %}
```hcl
resource "aws_subnet" "public" {
  for_each                = var.public_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = each.value.cidr_block
  availability_zone       = each.value.availability_zone
  map_public_ip_on_launch = true

  tags = {
    Name = each.key
  }
}

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
```
{% endtab %}

{% tab title="TO-BE" %}
```hcl
resource "aws_subnet" "subnets" {
  for_each                = var.subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = each.value.cidr_block
  availability_zone       = each.value.availability_zone
  map_public_ip_on_launch = lookup(each.value, "nat_gateway_subnet", null) != null ? false : true

  tags = {
    Name = each.key
  }
}
```

{% hint style="success" %}
**코드에서 사용된&#x20;**<mark style="color:purple;">**`Lookup`**</mark>**&#x20;함수 알아보기**

<mark style="color:purple;">**`Lookup`**</mark> 함수는 <mark style="color:green;">Map type</mark> 에 있는 요소를 키로 접근하는 함수이다. 마치 파이썬의 "_Dictionary to get"_ 과 다름 없다.



> _python examples_

```python
a = {"a": "ay", "b": "bee"}

print(a.get("a")) # "ay"
print(a.get("c", None))  # None
```

> _terraform examples_

```hcl
lookup({a="ay", b="bee"}, "a", "what?")  # ay
lookup({a="ay", b="bee"}, "c", "what?")  # what?
```
{% endhint %}
{% endtab %}
{% endtabs %}



> _**"서브넷 그룹에서 Private 서브넷만 필요한데 어떻게 가져오지?"**_

테라폼 문법에서 제공 하는 `for` expression 을 살펴보면 반복문 수행 중 키의 값을 새로운 값으로 할당할 수 있는데 이 문법을 활용 하여 private subnet 을 지역 변수로 구성한다.

{% tabs %}
{% tab title="main.tf" %}
```hcl
locals {
  private_subnets = {
    for key, subnet in var.subnets :
    key => subnet
    if(lookup(subnet, "nat_gateway_subnet", null)) != null
  }
}
```
{% endtab %}
{% endtabs %}

:bulb: 지역 변수를 할당한 후 제대로 구성 되었는지 확인하고 싶다면 아래와 같은 명령어를 통해 확인할 수 있다.

**Command**&#x20;

```sh
terraform console

> local.private_subnets
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

그래서 현재 테라폼 구성을 살펴보면 서브넷이 하나의 그룹으로 통합 되어 있고 지역 변수로 내부망 서브넷을 별도로 분리 해두었다.

기존 내부망 서브넷을 사용하고 있던 NAT 관련 리소스는 리팩터링으로 인해 동작하지 않기 때문에 이 부분도 새롭게 수정한다.



{% tabs %}
{% tab title="AS-IS" %}
```hcl
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

{% tab title="TO-BE" %}
```hcl
resource "aws_eip" "nat_ips" {
  for_each = local.private_subnets
}

resource "aws_nat_gateway" "gateways" {
  for_each = local.private_subnets

  allocation_id = aws_eip.nat_ips[each.key].id
  subnet_id     = aws_subnet.subnets[each.value.nat_gateway_subnet].id
}
```
{% endtab %}
{% endtabs %}



테라폼 구성은 동일 하지만 가독성을 위해 리팩터링을 모두 마쳤다면 테라폼 실행 계획을 살펴본 후 생성 후 재 생성 하는 테라폼 상태 파일을 제어 하여 변경 사항 없음으로 만들어준다.

```sh
> terraform state list
> terraform state mv aws_subnet.public\[\"oimarket-apne2-public-subnet-b\"\] aws_subnet.subnets\[\"oimarket-apne2-public-subnet-b\"\]
> terraform state mv aws_subnet.public\[\"oimarket-apne2-public-subnet-a\"\] aws_subnet.subnets\[\"oimarket-apne2-public-subnet-a\"\]
# ... 나머지 구성 변경 사항도 동일하게 적용
> terraform plan
```

튜토리얼에서 비용 발생에 대한 우려로 시나리오만 진행 하여 NAT Gateway와 Private Subnet을 만들지 않은 상태라면 동일하게 6개의 리소스가 새롭게 생성 되는 화면이 나오면된다.&#x20;

<details>

<summary>Summary</summary>

이렇게 테라폼을 구성할 때 리소스를 가독성 있게 구성하는 방식이 중요한데, 그 중 첫번째 인 지역변수를 활용한 방식을 사용해보았다.&#x20;

* 리팩터링 시 <mark style="color:purple;">**terraform state mv**</mark> 명령어를 사용하여 기존에 만들어둔 리소스를 이전해야 한다.
* 유사한 변수를 하나로 합친 후 로컬 변수를 사용하여 리팩터링 할 수 있다.
* <mark style="color:purple;">**lookup()**</mark> 함수를 사용하여 <mark style="color:green;">Map type</mark> 의 변수에서 특정 키의 값을 조회 할 수 있다.

</details>



### Routing table

{% hint style="warning" %}
_**"VPC 구성의 종착지인 라우팅 테이블을 구성하다"**_
{% endhint %}

기존 Public subnet 은 IGW를 통해 인터넷 통신을 이루고 Private subnet은 NAT Gateway를 통해 인터넷 통신을 이루도록 리스소를 정의했다.

이렇게 서브넷을 분리 시켰지만 별도의 라우팅 테이블을 만들지 않는다면 AWS VPC 정책에 의해 기본적으로 모두 로컬 라우팅 테이블에 포함되어 인터넷을 사용할 수 있게 된다.&#x20;

그렇기 때문에 라우팅 테이블을 별도로 생성 하여 특정 서브넷에 대한 제어를 하도록 만든다.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
**라우팅 테이블을 만들기 위해 필요한 세 개의 리소스**

1. route\_table: 라우팅 룰을 담고 있는 테이블
2. route: 라우팅 룰, 주로 어떤 라우트 테이블에 속해 있으며 어떤 게이트웨이를 사용 하여 통신할지 정의
3. route\_table\_association: 라우팅 룰에 포함 될 서브넷들
{% endhint %}



{% tabs %}
{% tab title="main.tf" %}
```hcl
resource "aws_route_table" "route_tables" {
  for_each = var.subnets
  vpc_id   = aws_vpc.main.id

  tags = {
    Name = "${each.key}-route-table"
  }
}

resource "aws_route" "routes" {
  for_each = var.subnets

  route_table_id         = aws_route_table.route_tables[each.key].id
  destination_cidr_block = "0.0.0.0/0"

  gateway_id     = lookup(each.value, "nat_gateway_subnet", null) == null ? aws_internet_gateway.main.id : null
  nat_gateway_id = lookup(each.value, "nat_gateway_subnet", null) != null ? aws_nat_gateway.gateways[each.key].id : null
}

resource "aws_route_table_association" "associations" {
  for_each = var.subnets

  subnet_id      = aws_subnet.subnets[each.key].id
  route_table_id = aws_route_table.route_tables[each.key].id
}

```
{% endtab %}
{% endtabs %}

<details>

<summary>Summary</summary>

* <mark style="color:purple;">**lookup()**</mark> 함수와 조건문을 조합해서 Private subnet, Public subnet일 때 게이트웨이를 다르게 설정할 수 있다.

</details>



### Dynamic block with SG

{% hint style="warning" %}
_**"리소스 설정 중 유연하게 추가 또는 삭제를 해야할 때 어떻게 해야할까?"**_
{% endhint %}

동적 블럭을 활용하기 위해서 보안 그룹 규칙을 생성할 때 사용 하며 <mark style="color:green;">map(object) type</mark>, <mark style="color:purple;">for\_each</mark>, <mark style="color:purple;">dynamic</mark> 등을 활용한다



> _**"만약 동적 블럭을 구성하지 않고 보안 그룹을 생성하면 어떻게 될까?"**_

동적 블럭 없이 리소스를 정의 하면 리소스 정의 코드가 길어지며 그에 걸맞는 다양한 변수를 사용해야한다.

{% tabs %}
{% tab title="Example" %}
```hcl
resource "aws_security_group" "permit_ssh_access" {

  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Permit SSH from anywhere"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow all traffic"
  }

  tags = {
    Name = "permit_ssh_sg"
  }

}


resource "aws_security_group" "permit_http_" {

  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Permit HTTP from anywhere"
  }
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Permit HTTP from anywhere"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow all traffic"
  }

  tags = {
    Name = "permit_http_sg"
  }

}
```
{% endtab %}
{% endtabs %}

두 코드는 거의 비슷하며 지금 단 두개의 보안 그룹 규칙을 생성했다. 만약, 4개, 5개 일 때 이와 같이 구성할 수 있겠는가? 이러한 비효율적인 코드를 줄이고 가독성을 높히는 것이 목적이기 때문에 동적 블럭을 활용하여 이 코드를 간소화할 수 있다.



{% tabs %}
{% tab title="main.tf" %}
```hcl
resource "aws_security_group" "groups" {
  for_each = var.security_groups

  name   = each.key
  vpc_id = aws_vpc.main.id

  dynamic "ingress" {
    for_each = each.value.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }

  dynamic "egress" {
    for_each = each.value.egress_rules
    content {
      from_port   = egress.value.from_port
      to_port     = egress.value.to_port
      protocol    = egress.value.protocol
      cidr_blocks = egress.value.cidr_blocks
    }
  }

  tags = {
    Name : each.key
  }
}
```
{% endtab %}

{% tab title="variables.tf" %}
```hcl
variable "security_groups" {
  description = "Map of security groups with rules"

  type = map(object({
    ingress_rules = list(object({
      from_port   = number
      to_port     = number
      protocol    = string
      cidr_blocks = list(string)
    }))
    egress_rules = list(object({
      from_port   = number
      to_port     = number
      protocol    = string
      cidr_blocks = list(string)
    }))
  }))

}

```
{% endtab %}

{% tab title="terraform.tfvars" %}
```hcl
security_groups = {
  "oimarket-apne2-permit-ssh-security-group" = {
    ingress_rules = [
      {
        cidr_blocks = ["0.0.0.0/0"]
        from_port   = 22
        protocol    = "tcp"
        to_port     = 22
      }
    ]

    egress_rules = [{
      cidr_blocks = ["0.0.0.0/0"]
      from_port   = 0
      protocol    = "-1"
      to_port     = 0
    }]
  },

  "oimarket-apne2-permit-http-security-group" = {
    ingress_rules = [
      {
        cidr_blocks = ["0.0.0.0/0"]
        from_port   = 80
        protocol    = "tcp"
        to_port     = 80
      },
      {
        cidr_blocks = ["0.0.0.0/0"]
        from_port   = 443
        protocol    = "tcp"
        to_port     = 443
      }
    ]

    egress_rules = [{
      cidr_blocks = ["0.0.0.0/0"]
      from_port   = 0
      protocol    = "-1"
      to_port     = 0
    }]
  }
}

```
{% endtab %}
{% endtabs %}

<details>

<summary>Summary</summary>

* <mark style="color:purple;">**dynamic**</mark> 지시자와 <mark style="color:purple;">**for\_each**</mark>를 조합해서 동적 블럭을 생성 하면 불필요한 코드를 효율적으로 리팩터링 할 수 있다.

</details>



### Module

{% hint style="warning" %}
_**"하나 이상의 리소스와 관련된 코드 블록을 재사용 가능하게 만든 템플릿"**_
{% endhint %}

> _**"모듈은 언제 사용할까?"**_

여러 리전에 걸쳐 위에서 만든 VPC를 재사용해야 한다고 하면 이런 시나리오를 만들어낼 수 있다.

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

위 코드를 Copy & Paste 로 사용할 경우 추후 변경되는 리소스에 대한 동기화를 위해 각 리전에서 정의 된 변수 파일을 탐색 하며 수정해야한다. 이 때 휴먼에러가 발생할 확률이 굉장히 높기 때문에 코드화 하는 의미가 많이 퇴색된다.



바람직하게 모듈을 이용하여 미리 정의된 리소스를 참조 하도록 만들기 위해서 아래와 같이 구성할 수 있다.

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

> **"모듈을 참조하는 방법"**

기존 변수를 담고 있던 terraform.tfvars 파일은 이제 필요가 없어졌다. 정의된 리소스에 맞게 변수에 값을 할당 하는 것은 해당 모듈을 참조하는 main.tf로 책임을 넘겼기 때문이다. 그래서 아래 모듈 코드를 보면 변수를 길게 설정해둔 것을 볼 수 있다.

{% tabs %}
{% tab title="vpc/main.tf" %}
```hcl
module "vpc" {
  source = "../../modules/vpc"

  vpc_name           = "oimarket-apne2"
  cidr_block         = "10.0.0.0/16"
  availability_zones = ["a", "b"]

  subnets = {
    "oimarket-apne2-public-subnet-a" = {
      availability_zone = "ap-northeast-2a"
      cidr_block        = "10.0.1.0/24"
    },
    "oimarket-apne2-public-subnet-b" = {
      availability_zone = "ap-northeast-2b"
      cidr_block        = "10.0.2.0/24"
    },
    "oimarket-apne2-private-subnet-a" = {
      availability_zone  = "ap-northeast-2a"
      cidr_block         = "10.0.11.0/24"
      nat_gateway_subnet = "oimarket-apne2-public-subnet-a"
    }
    "oimarket-apne2-private-subnet-b" = {
      availability_zone  = "ap-northeast-2b"
      cidr_block         = "10.0.12.0/24"
      nat_gateway_subnet = "oimarket-apne2-public-subnet-b"
    },
  }

  security_groups = {
    "oimarket-apne2-permit-ssh-security-group" = {
      ingress_rules = [
        {
          cidr_blocks = ["0.0.0.0/0"]
          from_port   = 22
          protocol    = "tcp"
          to_port     = 22
        }
      ]

      egress_rules = [{
        cidr_blocks = ["0.0.0.0/0"]
        from_port   = 0
        protocol    = "-1"
        to_port     = 0
      }]
    },

    "oimarket-apne2-permit-http-security-group" = {
      ingress_rules = [
        {
          cidr_blocks = ["0.0.0.0/0"]
          from_port   = 80
          protocol    = "tcp"
          to_port     = 80
        },
        {
          cidr_blocks = ["0.0.0.0/0"]
          from_port   = 443
          protocol    = "tcp"
          to_port     = 443
        }
      ]

      egress_rules = [{
        cidr_blocks = ["0.0.0.0/0"]
        from_port   = 0
        protocol    = "-1"
        to_port     = 0
      }]
    }
  }

}
```
{% endtab %}
{% endtabs %}

위 모듈은 하위 모듈 파일을 참조하도록 선언하고, 해당 모듈이 사용하는 변수를 그대로 선언하여 사용한다.

이렇게 기존 리소스 구성에서 모듈화를 진행 하면 테라폼을 사용할 준비를 다시 해야하기 때문에 워크플로우를 익혀보자면아래와 같다.



{% stepper %}
{% step %}
### 모듈 참조 구성 및 Terraform init

기존 리소스에서 사용하던 상태 파일은 모듈로 넘어가있기 때문에 참조된 곳에 테라폼을 사용하기 위한 준비를 마친다.
{% endstep %}

{% step %}
### Terraform plan

이 때, 실행 계획을 살펴보면 기존에 만들어둔 리소스가 모두 삭제되고 새롭게 재생성 되는 것을 볼 수 있는데 이러한 이유는 기존에는 모듈에서 참조하지 않았기 때문에 "aws\_vpc.main"이었다면 모듈에 참조된 후 "module.aws\_vpc.main"이 되어 상태파일에 변경이 생긴다.
{% endstep %}

{% step %}
### Terraform state mv

{% code overflow="wrap" %}
```sh
terraform show -json | jq -r ".values.root_module.resources[] | .address" | awk '{gsub(/"/, "\\\"");print "terraform state mv "$1" module.vpc."$1}' | /bin/bash
```
{% endcode %}

* terraform show -json: 테라폼의 상태 파일을 JSON으로 확인
*   jq -r: JSON Pretty print로 출력하고 읽음

    <figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

    * ".values.root\_module.resources\[] | address" : 상태 파일의 리소스 이름만 출력
    * awk: 파일의 내용을 데이터화 하는 데 사용
    * gsub(/"/, "\\\\\\""): " 로 시작하는 문자를 찾아 \\" 로 변경
    * print "terraform state mv "$1": address에서 추출한 이름을 변경 대상으로 사용
    * module.vpc."$1: module.vpc.\\"address 에서 추출한 이름으로 변경
    * bin/bash: 실행되는 코드를 라인마다 읽어 들여 Bash 로 실행 -> 이 때 반영 됨
{% endstep %}
{% endstepper %}

<details>

<summary>Summary</summary>

* 모듈을 만들면 테라폼 구성을 위한 코드 블록을 재사용할 수 있다.
* 리팩터링을 하면 언제나 상태를 옮기는 작업이 필수적으로 수행되어야 한다.

</details>



### Variables In Yaml File

{% hint style="warning" %}
_**"YAML 파일을 읽어서 변수 값을 지정하여 가독성을 향상 시킨다"**_
{% endhint %}



> _**"왜 YAML 파일을 활용해야 할까?"**_

<figure><img src="../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

기존 변수 파일로 관리 했을 때 변수를 선언하고 해당 변수에 대한 값을 할당 해서 사용 해야 했지만 YAML을 활용하게 되면 **테라폼을 구성하는 데이터들과 실제 테라폼 로직을 분리**할 수 있어 가독성이 좋아진다. 또한, 테라폼을 잘 모르더라도 YAML 파일만 수정하면 되는 유연함이 있다.



{% tabs %}
{% tab title="AS-IS" %}
```hcl
module "vpc" {
  source = "../../modules/vpc"

  vpc_name           = "oimarket-apne2"
  cidr_block         = "10.0.0.0/16"
  availability_zones = ["a", "b"]

  subnets = {
    "oimarket-apne2-public-subnet-a" = {
      availability_zone = "ap-northeast-2a"
      cidr_block        = "10.0.1.0/24"
    },
    "oimarket-apne2-public-subnet-b" = {
      availability_zone = "ap-northeast-2b"
      cidr_block        = "10.0.2.0/24"
    },
    "oimarket-apne2-private-subnet-a" = {
      availability_zone  = "ap-northeast-2a"
      cidr_block         = "10.0.11.0/24"
      nat_gateway_subnet = "oimarket-apne2-public-subnet-a"
    }
    "oimarket-apne2-private-subnet-b" = {
      availability_zone  = "ap-northeast-2b"
      cidr_block         = "10.0.12.0/24"
      nat_gateway_subnet = "oimarket-apne2-public-subnet-b"
    },
  }

  security_groups = {
    "oimarket-apne2-permit-ssh-security-group" = {
      ingress_rules = [
        {
          cidr_blocks = ["0.0.0.0/0"]
          from_port   = 22
          protocol    = "tcp"
          to_port     = 22
        }
      ]

      egress_rules = [{
        cidr_blocks = ["0.0.0.0/0"]
        from_port   = 0
        protocol    = "-1"
        to_port     = 0
      }]
    },

    "oimarket-apne2-permit-http-security-group" = {
      ingress_rules = [
        {
          cidr_blocks = ["0.0.0.0/0"]
          from_port   = 80
          protocol    = "tcp"
          to_port     = 80
        },
        {
          cidr_blocks = ["0.0.0.0/0"]
          from_port   = 443
          protocol    = "tcp"
          to_port     = 443
        }
      ]

      egress_rules = [{
        cidr_blocks = ["0.0.0.0/0"]
        from_port   = 0
        protocol    = "-1"
        to_port     = 0
      }]
    }
  }

}
```
{% endtab %}

{% tab title="TO-BE" %}
```hcl
locals {
  config = yamldecode(file("config.yaml"))
}

module "vpc" {
  source = "../../modules/vpc"

  vpc_name           = local.config.vpc_name
  cidr_block         = local.config.cidr_block
  subnets            = local.config.subnets
  security_groups    = local.config.security_groups
  availability_zones = local.config.availability_zones
}

```
{% endtab %}
{% endtabs %}

<details>

<summary>Summary</summary>

* 테라폼이 제공해 주는 <mark style="color:purple;">**yamldecode()**</mark> 함수를 사용하면 YAML 파일을 읽을 수 있다.
* YAML 파일 활용하면 테라폼 구성의 가독성을 높일 수 있고 데이터와 로직을 분리할 수 있다.

</details>



### Dynamic Syntax Optimization

{% hint style="danger" %}
_**"일부 보안그룹 규칙만 수정 했는데 전체 규칙이 변경되는 이슈가 발생 하다"**_

위 과정을 실습하다보면 제대로 수정이 되었는지 확인하기 위해 보안 그룹 443을 543으로 바꾸어보면 443 보안그룹만 영향을 받는것이 아니라 그 외 다른 보안그룹에도 영향을 끼친다.

이 경우 삭제 후 재생성이 되는데, 다시 재생성 되기 때문에 변화가 없다고 생각할 수 있지만 **잠깐의 다운타임으로 서비스에 장애를 만들어낼 수 있는 여지가 있는것이다.**
{% endhint %}

기존에 사용중이던 보안 그룹 규칙은 테라폼 구성의 Inline-block 으로 Ingress, Egress를 분리 했지만 이렇게 사용할 경우 공식문서에서 다음과 같은 이슈가 있다고 설명한다.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

그렇기 때문에 가장 좋은 방법은 aws\_vpc\_security\_egress\_rule 과 ingress\_rule 리소스를 사용해서 CIDR 블럭을 관리하라고 설명한다.

현재 문제 상황에 가장 적합한 이슈로 이 공식문서가 가이드하는 방식대로 리소스를 새롭게 정의하여 동적 블럭에서 한 개의 리소스를 수정 했을 때 다른 리소스도 영향이 받지 않는지 확인 하는데, 이 때 기존에 사용중이던 변수 타입도 분리해야한다.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% tabs %}
{% tab title="AS-IS(main.tf)" %}
```hcl
resource "aws_security_group" "groups" {
  for_each = var.security_groups

  name   = each.key
  vpc_id = aws_vpc.main.id

  dynamic "ingress" {
    for_each = each.value.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }

  dynamic "egress" {
    for_each = each.value.egress_rules
    content {
      from_port   = egress.value.from_port
      to_port     = egress.value.to_port
      protocol    = egress.value.protocol
      cidr_blocks = egress.value.cidr_blocks
    }
  }

  tags = {
    Name : each.key
  }
}
```
{% endtab %}

{% tab title="AS-IS(variables.tf)" %}
```hcl
variable "security_groups" {
  description = "Map of security groups with rules"

  type = map(object({
    ingress_rules = list(object({
      from_port   = number
      to_port     = number
      protocol    = string
      cidr_blocks = list(string)
    }))
    egress_rules = list(object({
      from_port   = number
      to_port     = number
      protocol    = string
      cidr_blocks = list(string)
    }))
  }))

}

```
{% endtab %}

{% tab title="TO-BE(main.tf)" %}
```hcl
resource "aws_security_group" "groups" {
  for_each = var.security_groups

  name   = each.key
  vpc_id = aws_vpc.main.id

  tags = {
    Name : each.key
  }
}

resource "aws_vpc_security_group_ingress_rule" "rules" {
  for_each = var.ingress_rules

  security_group_id = aws_security_group.groups[each.value.security_group_name].id
  cidr_ipv4         = each.value.cidr_ipv4
  from_port         = each.value.from_port
  ip_protocol       = each.value.ip_protocol
  to_port           = each.value.to_port
}


resource "aws_vpc_security_group_egress_rule" "rules" {
  for_each = var.egress_rules

  security_group_id = aws_security_group.groups[each.value.security_group_name].id
  cidr_ipv4         = each.value.cidr_ipv4
  from_port         = each.value.from_port
  ip_protocol       = each.value.ip_protocol
  to_port           = each.value.to_port
}
```
{% endtab %}

{% tab title="TO-BE(variables.tf)" %}
```hcl
variable "security_groups" {
  description = "Map of security groups with rules"

  type = map(object({}))

}

variable "ingress_rules" {
  type = map(object({
    security_group_name = string
    cidr_ipv4           = string
    from_port           = optional(number)
    ip_protocol         = string
    to_port             = optional(number)
  }))
}

variable "egress_rules" {
  type = map(object({
    security_group_name = string
    cidr_ipv4           = string
    from_port           = optional(number)
    ip_protocol         = string
    to_port             = optional(number)
  }))
}

```
{% endtab %}

{% tab title="config.yaml" %}
```yaml
security_groups:
  oimarket-apne2-permit-http-security-group: null
  oimarket-apne2-permit-ssh-security-group: null
ingress_rules:
  oimarket-apne2-permit-http-security-group-allow-80:
    security_group_name: "oimarket-apne2-permit-http-security-group"
    cidr_ipv4: "0.0.0.0/0"
    from_port: 80
    ip_protocol: "tcp"
    to_port: 80
  oimarket-apne2-permit-http-security-group-allow-443:
    security_group_name: "oimarket-apne2-permit-http-security-group"
    cidr_ipv4: "0.0.0.0/0"
    from_port: 443
    ip_protocol: "tcp"
    to_port: 443
  oimarket-apne2-permit-ssh-security-group-allow-22:
    security_group_name: "oimarket-apne2-permit-ssh-security-group"
    cidr_ipv4: "0.0.0.0/0"
    from_port: 22
    ip_protocol: "tcp"
    to_port: 22
egress_rules:
  oimarket-apne2-permit-http-security-group-allow-permit:
    security_group_name: "oimarket-apne2-permit-http-security-group"
    cidr_ipv4: "0.0.0.0/0"
    ip_protocol: "-1"
  oimarket-apne2-permit-ssh-security-group-allow-permit:
    security_group_name: "oimarket-apne2-permit-ssh-security-group"
    cidr_ipv4: "0.0.0.0/0"
    ip_protocol: "-1"
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
**AWS 리소스가 이미 존재하여 에러가 나는 경우**

이후 테라폼 실행 계획을 통해 리소스 추가에 대해 확인 하고, 만약 NAT Gateway부터 미리 생성하면서 학습을 했다면 테라폼 구성에 이미 SG가 생성 되어 있어 에러가 발생한다.

이 때, 테라폼 상태 파일을 프로바이더에 있는 기존 리소스에서 가져올 수 있는 terraform import 명령어를 통해 현재 변경되는 사항과 동기화 하는 과정을 거친다.

_**"terraform import {테라폼 생성 명칭} {프로바이더 리소스 아이디}"**_

```sh
terraform import module.vpc.aws_vpc_security_group_ingress_rule.rules\[\"oimarket-apne2-permit-http-security-group-allow-443\"\] sgr-00baa41a10525cd7a
```
{% endhint %}

문제가 되었던 다이나믹 블럭의 보안 그룹을 수정했을 때 다른 리소스에 영향이 가지 않는지 테스트를 한 번 해보면 정상적으로 수정된 보안 그룹 규칙만 영향을 받는 것을 알 수 있다.

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>Summary</summary>

* 원하는대로 동작하지 않을때는 테라폼 공식 문서를 참고해서 확인해 봅니다.
* <mark style="color:purple;">**terraform import**</mark> 명령을 사용하면 이미 프로비저닝 되어 있는 리소스를 테라폼으로 관리할 수 있게 됩니다.

</details>







