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

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

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









