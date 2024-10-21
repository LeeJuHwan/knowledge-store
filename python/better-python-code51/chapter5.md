# You can do it, but it means you don't have to

<details>

<summary>Properties</summary>

:pencil:2024.08.22

:page\_facing\_up:[똑똑하게 코딩하는법 코딩의 기술 51](https://product.kyobobook.co.kr/detail/S000213798363)

:paperclip: **CHAPTER 5. 할 수 있다고 해서 해야 한다는 뜻은 아니다**

</details>

\
\


#### 5.2 Monkey patch

* :bookmark: page. 188

***

동적인 환경에서 쉽게 접할 수 있는 개발 방법으로 `Ruby` 에서 많이 차용되어 쓰여왔다. 하지만 이런 방식은 기존 소스코드의 실행 동작을 망칠 수 있기 때문에 사용을 자제 하는 편이나, `Ruby on Rails` 프레임워크 내부 코드는 대개 많은 동작들이 `MonkeyPatch` 형식으로 이루어져 있기 때문에 교체하지 못하는 히스토리를 가지고 있다.

> Why do not use?

`MonkeyPatch` 의 실행 동작을 살펴 보자면 아래 예시와 같다.

```python
"""
monkey_patch.py

monkey patch module
"""

import re
from re import match as _match

def match_positive(pat, s, flags=0):
	return _match(rf"\+{pat}", s, flags)

re.match = match_positive
```

```python
"""
accounts.py

print has 5k balance
"""

import monkey_patch
import re
import sys

def get_5k_balances(rows):
	for row in rows:
		if re.match("5\d{3}\.\d{2}", row):
			balance, account_num, owner = row.split(",")
			yield (f"Account: {account_num}\n"
				f" Owner: {owner.strip()}\n"
				f"Balance: ${float(balance):,.2f}\n")


if __name__ == "__main__":
	for account in get_5k_balances(open(sys.argv[1])):
		print(account)
```

여기까지 코드를 보면 데이터를 알기 때문에 잘 작동하던 코드가 있었고, 그 데이터가 갑작스레 변경 되었다. 그러한 요구사항을 소스코드의 변경 없이 동작을 유지 시키려면 외부 라이브러리를 참조 하고 있는 함수를 원하는 요구사항에 맞게 덮어씌우는 형태로 고칠 수 있다. 이 것이 `MonkeyPatch` 로 불리우는데 이런 경우는 **실제 잘 동작하던 다른 코드도 영향을 받을 수 있으며, 타인이 해당 코드를 읽을 때 동작을 유추하기 어렵다** 는 단점이 분명하게 존재 하기에 사용 하지 말아야한다.

\


```python
"""
accounts2.py

Error!
"""
import monkey_patch
import re
import argparse  

def get_5k_balances(rows):
	for row in rows:
		if re.match("5\d{3}\.\d{2}", row):
			balance, account_num, owner = row.split(",")
			yield (f"Account: {account_num}\n"
				f" Owner: {owner.strip()}\n"
				f"Balance: ${float(balance):,.2f}\n")


if __name__ == "__main__":
	parser = argparse.ArgumentParser()
	parser.add_argument("datafile", type=str, help="CSV data file")
	parser.add_argument("-d", "--datafile", type=str, help="CSV data file")
	parser.add_argument("-f", "--function", action="store", default="get_5k_balances")
	args = parser.parse_args()
	func = eval(args.function)
	
	for account in get_5k_balances(open(args.datafile)):
		print(account)
```

위에선 정상 동작 하는 코드로 구현 되었지만 실제 이와 같이 에러가 나는 상황을 보면, 흔히 사용 하는 라이브러리에서 `MonkeyPatch`로 오버라이딩 한 메서드를 사용하는 경우가 있다. 그 라이브러리의 모듈 조차 영향을 받는 상황을 만들어낼 수 있는 점이 있다.

\


> What's the conclusion

`MonkeyPatch` 를 피해야 하는 이유를 이해 하는 것이 중요하다. 실제 코드에서 과연 누가 외부 라이브러리의 이름을 그대로 주입하여 실행 동작을 변경 시킬 것인가? 이렇게 인위적인 상황을 예시로 들었지만 `MonkeyPatch` 를 피하고 새로운 함수를 정의 하는 것이 올바른 방법이라고 생각한다.
