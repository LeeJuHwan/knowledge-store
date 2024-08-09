<details>

<summary>Properties</summary>

:pencil:2024.03.31

:page_facing_up:[파이썬 코딩의 기술: 똑똑하게 코딩 하는 법 2판](https://product.kyobobook.co.kr/detail/S000001834494)

:paperclip: **BETTER WAY 44**

</details>


# Introduction

- 애트리뷰트를 관리하는 메서드를 생성하지 않고 프로퍼티 데코레이터를 통해 애트리뷰트를 다시 정의 할 수 있다.

# Goal

- 기존 클래스를 호출하는 코드를 전혀 바꾸지 않고 애트리뷰트의 기존 동작을 변경하는 방법



<aside>Worst case</aside>

> 애트리뷰트를 관리하는 메서드가 예외 상황 처리에 대해 애매한 경우


```python

# NOTE: Old versions
class Bucket:
	def __init__(self, period: int):
	self.period_delta = timedelta(seconds=period)
	self.reset_time = datetime.now()
	self.quota = 0

	def __str__(self):
		return f"Bucket(quota={self.quota})"


def fill(bucket: Bucket, amount: int):
	"""
	가용 용량을 소비할 때 마다 시간을 검사 하고, 주기가 달라진 경우 미사용한
	가용 용량이 새로운 주기로 넘어오지 못하게 함
	"""
	now = datetime.now()
	
	if (now - bucket.reset_time) > bucket.period_delta:
		bucket.quota = 0
		bucket.reset_time = now
		bucket.quota += amount
  

def deduct(bucket: Bucket, amount: int):
	"""
	버킷의 가용 용량에서 필요한 분량 사용하기
	"""
	now = datetime.now()

	if (now - bucket.reset_time) > bucket.period_delta:
		return False # NOTE: 새 주기가 시작 됐는데 아직 버킷 할당량이 재설정 되지 않음

	if (bucket.quota - amount) < 0:
		return False # NOTE: 버킷의 가용 용량이 충분하지 않음
	else:
		bucket.quota -= amount
	return True # NOTE: 버킷의 가용 용량이 충분하여 필요한 분량 사용
  

def old_version_callable():
	
	"""
	이 구현의 문제점:
		1. 버킷이 시작 할때 가용 용량을 알 수 없는 것
		2. 가용 용량을 초과해서 사용 하는 경우 가용 용량이 0인지, 사용 용량이 초과된 것인지 알 수 없음
	
	이 구현의 해결 방법:
		1. 주기에 재설정 된 가용 용량 인자 추가 -> max_quota
		2. 소비한 용량의 합계 추가 -> quota_consumed
	
		* 위 해결 방법을 기준으로 클래스 구조 변경
	"""

	bucket = Bucket(60)

	fill(bucket, 100) # NOTE: 버킷을 사용 하기 전 필요 가용 용량 할당
	print(bucket)

	used: int = 99

	if deduct(bucket, used): # NOTE: 필요한 용량 사용
		print(f"{used} 용량 사용")
	else:
		print("가용 용량이 작아서 99용량을 처리 할 수 없음")
	print(bucket)

	# NOTE: 남은 가용 용량보다 더 많은 용량을 사용하려 하는 경우
	used = 3
	if deduct(bucket, 3):
		print(f"{used} 용량 사용")
	else:
		print(f"가용 용량이 작아서 {used} 용량을 처리할 수 없음")
	print(bucket)
```
- 위 버킷 클래스의 문제점은 버킷이 시작할 때 가용 용량을 알 수 없다는 것, deduct에서 관리 하게 되면 다른 모듈에서 호출 할 때 공통 비즈니스 로직을 사용 해야만 한다. 하지만, 요구사항이 달라지는 경우 조건분기가 많아지는 단점이 생긴다.

- 그렇다면 생성하는 클래스에서 처리할 수 있도록 프로퍼티 데코레이터를 사용하면 원인을 해결 할 수 있을 것이다.

<aside>Solution</aside>


> 프로퍼티 데코레이터를 이용 하여 객체 내부에서 관리하기

```python
class NewBucket:
	def __init__(self, period: int):
		self.period_delta = timedelta(seconds=period)
		self.reset_time = datetime.now()
		self.max_quota = 0
		self.quota_consumed = 0

	def __str__(self):
		return (f"NewBucket(quota={self.max_quota}), "
				f"quota_consumed={self.quota_consumed}")

	@property
	def quota(self) -> int:
		return self.max_quota - self.quota_consumed

	@quota.setter
	def quota(self, amount: int):
		delta = self.max_quota - amount

		if amount == 0: # NOTE: 새로운 주기가 되고 가용 용량을 재설정함
			self.quota_consumed = 0
			self.max_quota = 0
		elif delta < 0: # NOTE: 새로운 주기가 되고 가용 용량을 추가 하는 경우
			assert self.quota_consumed == 0
			self.max_quota = amount
		else: # NOTE: 주기 안에서 가용 용량을 정상적으로 소비하는 경우
			assert self.max_quota >= self.quota_consumed
			self.quota_consumed += delta
  

def new_version_callable():
	bucket = NewBucket(60)
	print(f"최초: {bucket}")
	
	fill(bucket, 100)
	print(f"보충 후: {bucket}")

	used: int = 99

	if deduct(bucket, used):
		print(f"{used} 용량 사용")
	else:
		print(f"가용 용량이 작아서 {used} 용량을 처리할 수 없음")
		print(f"사용 후: {bucket}")

	# NOTE: 남은 가용 용량을 초과해서 사용하는 경우
	used = 3
	if deduct(bucket, used):
		print(f"{used} 용량 사용")
	else:
		print(f"가용 용량이 작아서 {used} 용량을 처리할 수 없음")

	print(f"여전히 {bucket}")
```
- 이렇게 프로퍼티 데코레이터를 이용한 애트리뷰트는 여러 클래스가 생성 되어도 각자 요구하는 비즈니스 로직이 담길 수 있다. 공통 유틸 함수를 수정하려 할 때를 생각 해보면 굉장히 깔끔해진 것을 알 수 있다.
- 하지만, 프로퍼티 데코레이터도 반복해서 확장 하고 있다면 클래스 구조의 단점을 감추기 위한 작업이란 것을 잊지 말아야한다. 그럴 땐 꼭 리팩터링을 고려해라.


# Summary

- 데코레이터 프로퍼티는 실제 문제를 해결 할 때 요구사항에 맞게 처리하기 알맞은 도움을 준다. 하지만, 과한 것 중 좋은 것은 없다.
- 프로퍼티 데코레이터 메서드를 확장하고 있다면 리팩터링 할 때다.