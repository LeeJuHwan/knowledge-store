# Readable code

> **인지적 경제성을 추구하자**


> 정리 하는 뇌 - 대니얼 J.레비틴
> "정리 시스템에서 중요한 과제는 **최소의 인지적 노력으로 최대의 정보를 제공** 하는 것이다."
> - 특정 대상을 특징에 따라 분류 하는 범주화를 기준으로 최소한의 정보로 최대한을 기억 해낼 수 있다.

> 도둑맞은 집중력 - 요한 하리
> "뇌는 **한 번에 한가지 일** 밖에 하지 못한다. 멀티태스킹? 그건 저글링일 뿐."

---

<br></br>

### Early Return으로 else의 사용을 지양하자

```java
if (a > 3) {
	code();
} else if (a <= 3 && b > 1) {
	code2();
} else {
	code3();
}

```

위와 같은 코드가 있을 때 코드를 읽는 대상은 조건문의 흐름도에 따라 선수 조건이 만족 했는지에 대한 정보를 지속적으로 기억하고 있어야한다. 그렇게 매우 긴 코드를 거쳐 마지막 `else` 문을 읽을 때 쯤 되면 첫 조건을 잊을 수도 있다. 그렇다면 다시 돌아가야 할까?

**메서드 추출 및 Early return 사용하기**

```java

void extracted() {
	if (a > 3) {
		code();
		return
	} 
	if (a <= 3 && b > 1) {
		code2();
		return
	} 
	code3();
	return
}
```

앞선 조건을 다시 생각 해보자면 첫번째 조건이 만족 해야 한다는 정보가 필요 없어지고 개별 조건에 따른 정보만 얻으면 된다. 이렇게 `else` 는 내가 처리하지 못하는 예외 케이스를 접근하게 하기도 하며 까다로운 상황을 만들 수 있는 요인이 되기 때문에 최대한 지양하는 방법을 사용해보자.