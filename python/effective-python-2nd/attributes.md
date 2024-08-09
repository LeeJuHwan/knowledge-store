<details>

<summary>Properties</summary>

:pencil:2024.03.24

:page_facing_up:[파이썬 코딩의 기술: 똑똑하게 코딩 하는 법 2판](https://product.kyobobook.co.kr/detail/S000001834494)

:paperclip: **BETTER WAY 45**

</details>


# Introduction

- 세터와 게터 메서드 대신 평범한 애트리뷰트를 사용하라

# Goal

- 파이썬에서 게터-세터를 다루는 방법을 이해하고, 공개적 애트리뷰트를 사용하여 간편한 인터페이스를 구현하자.

- 프로퍼티 데코레이터를 이용한 다양한 연산을 적용한 애트리뷰트를 반환하자





<aside>Worst case</aside>

> 통상적인 게터-세터를 사용한 코드이지만, 파이써닉하지 않은 코드의 예시이다.


```python
class OldResistor:
	def __init__(self, ohms):
		self._ohms = ohms

	def get_ohms(self):
		return self._ohms

	def set_ohms(self, ohms):
		return self._ohms = ohms

if __name__ == "__main__":
	r0 = OldResistor(50e-3)
	print(f"이전: {r0.get_ohms()}")
	
	r0.set_ohms(10e3)
	print(f"{r0.get_hms()}")  
```
- get - set구조의 유틸리티 메서드는 코드를 지저분하게 한다. 하지만, 기능을 캡슐화 하는 경계를 설정하기 쉽기 때문에 자주 사용된다.
- 파이썬에서는 위 처럼 코드를 구현할 필요가 전혀 없다.


<aside>Solution</aside>


> 공개 애트리뷰트를 이용하기

```python
class Resitor:
	def __init__(self, ohms):
		self.ohms = ohms
		self.voltage = 0
		self.current = 0

if __name__ == "__main__":
	r1 = Resistor(50e-3)
	r1.ohms = 10e3
	r1.ohms += 5e3
```
- 위 코드는 유틸리티 게터 세터를 사용하지 않은 채 공개 애트리뷰트를 이용했다. 코드가 굉장히 깔끔해지는 것을 볼 수 있다.
- 만약, 애트리뷰트를 설정할 때 특별한 기능을 수행 해야 한다면 프로퍼티 데코레이터를 이용 할 수 있다.




<aside><mark style='background:var(--mk-color-blue)'><span style='color:var(--mk-color-yellow)'>Best case - 1</span></mark></aside>


> property decorator 사용 하여 기능 수행하기

```python
class VoltageResistance(Resistor):
	def __init__(self, ohms):
		super().__init__(ohms)
		self._voltage = 0

	@property
	def voltage(self):
		return self._voltage

	@voltage.setter
	def voltage(self, voltage):
		self._voltage = voltage
		self.current = self._voltage / self.ohms

if __name__ == "__main__":
	r2 = VoltageResistance(1e3)
	print(f"이전: {r2.current:.2f} 암페어")

	r2.voltage = 10
	print(f"이후: {r2.current:.2f} 암페어")
```


<aside><mark style='background:var(--mk-color-blue)'><span style='color:var(--mk-color-yellow)'>Best case - 2</span></mark></aside>

> Property를 활용 하여 데이터 검증하기
> 
```python
class BoundedResistance(Resistor):
	def __init__(self, ohms):
		super().__init__(ohms)

	@property
	def ohms(self):
		return self._ohms

	@ohms.setter
	def ohms(self, ohms):
		if ohms <= 0:
			raise ValueError(f"저항 > 0이어야 합니다. 실제 값: {ohms}")
		self._ohms = ohms

if __name__ == "__main__":
	r3 = BoundedResistance(-5)
	r3.ohms = 0

# >> ValueError!
```


<aside><h2>Review</h2></aside>


* 게터나 세터를 정의 할 때 가장 좋은 정책 관련이 있는 객체 상태를 `property setter`메서드 안에서만 변경 한다.
* 더 복잡하거나 느린 연상의 경우에는 일반적인 메서드를 사용하라.
	* I/O, DB DML



# Summary

- 새로운 클래스 인터페이스를 정의 할 때는 간단한 공개 애트리뷰트에서 시작하고, 세터나 게터 메서드를 사용하지 말라.
- 객체에 접근하는 애트류비트의 특정한 기능이 필요하다면 property decorator를 활용하라.
- property decorator를 구성할 땐 메서드가 빠르게 실행 되도록 유지하라.