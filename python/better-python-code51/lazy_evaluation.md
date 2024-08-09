> Lazy Evaluation
<aside>Closure Lazy Evaluation And Eager Evaluation</aside>

* 함수 내부에 지역 객체를 생성하고 폐쇄적인 범위를 사용 하는 일급 함수를 다룰 때 지연 연산을 잊지 말아야한다.

	```python
	def make_adders(addends):
	    funcs = []
	    for addend in addends:
	        print(f"for iter addend: {addend}")
	        funcs.append(lambda x: print(f"x: {x}, addend: {addend}") or x + addend)
	    return funcs

	if __name__ == "__main__":
	    adders = make_adders([10, 100, 1000])
	    for adder in adders:
	        print(f"adder: {adder}")
	        print(adder(5))
	```

	Lazy Evaluation에 의하여 `addend` 는 반복문에 가장 마지막 값인 1000으로 산정 된다. 그 이유는 클로저가 폐쇄적인 범위를 사용 하는 일급 함수라는 것에 있다.

- 동작 순서
1. 반복문을 순회 하며 `funcs` 라는 List에 함수의 오브젝트 주소를 담아둔다.
2. 반복문 순회가 끝나면 함수의 주소값이 담겨있는 List를 반환한다.
3. `adders` 를 다시 순회 하면서 내부에 존재하는 함수 오브젝트 주소 값을 이용 하여 Callable 객체를 실행 시킨다.
	- 오브젝트 주소가 담겨져 있으며 최종적으로 메모리에 남아 있는 `addend` 값인 1000을 이용 하고 x에 파라미터 정보인 5를 사용 하게 된다.

- 결과

	```
	for iter addend: 10
	for iter addend: 100
	for iter addend: 1000
	adder: <function make_adders.<locals>.<lambda> at 0x7fd5d929e200>
	x: 5, addend: 1000
	1005
	adder: <function make_adders.<locals>.<lambda> at 0x7fd5d929e290>
	x: 5, addend: 1000
	1005
	adder: <function make_adders.<locals>.<lambda> at 0x7fd5d929e320>
	x: 5, addend: 1000
	1005
	x: 5, addend: 1000
	```


> Eager Evaluation

- Lazy Evaluation 방식을 강제로 Eager Evaluation으로 변경 할 수 있다. 메모리의 마지막 값을 사용 했던 방식을 빗대어 보면 default 값을 지정 해주었을 때 상쇄 시킬 수 있을 것이다.
- 기억 해보자면 파이썬은 언제나 객체를 참조하는 값을 사용하여 재사용률을 높힌다.

	```python
	def make_adders(addends):
	    funcs = []
	    for addend in addends:
	        print(f"for iter addend: {addend}")
	        funcs.append(lambda x, _addend=addend: print(f"x: {x}, addend: {_addend}") or x + _addend)
	    return funcs
	
	
	if __name__ == "__main__":
	    adders = make_adders([10, 100, 1000])
	    for adder in adders:
	        print(f"adder: {adder}")
	        print(adder(5))
	```

- 달라진 점
	- 여기서 중요한 포인트는 함수 파라미터의 기본 값을 사용 했다는 점이다. `_addend=addend` 는 현재 값을 함수 내부에서 사용 할 수 있도록 객체 참조값을 넣어주었다.
	- Lambda 내부에서 Print 할 수 있다는 것에 또 한번 놀라운 사실임을 알 수 있다.


- 결과
	```
	addend: 10
	addend: 100
	addend: 1000
	adder: <function make_adders.<locals>.<lambda> at 0x7f76727ee200>
	x: 5, addend: 10
	15
	adder: <function make_adders.<locals>.<lambda> at 0x7f76727ee290>
	x: 5, addend: 100
	105
	adder: <function make_adders.<locals>.<lambda> at 0x7f76727ee320>
	x: 5, addend: 1000
	1005
	```
