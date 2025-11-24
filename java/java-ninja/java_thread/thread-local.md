# 동시성을 벗어나게 해줄 Thread Local



### 스레드 로컬이란?

***

자바에서 동시성을 해결하기 위해 크게 두 가지 방식을 사용한다.

1. synchronized 예약어
2. ThreadLocal 객체

1번 같은 경우는 스레드가 자원에 접근하기 위해 락 경합을 해야 하는 단점이 있다.

2번은 스레드가 요청을 처리하는 동안 고유 데이터 영역을 보장해주는 사물함 같은 원리이다.



#### 장점

* 동시성을 피하기 위해 스레드 간 락 경합 없이 고유의 영역을 보장 받아 데이터에 접근할 수 있다.

&#x20;

#### 주의점

스레드의 생성과 종료를 모두 제어하는 환경에서는 스레드 로컬 객체는 GC에 의해 소멸된다.

하지만, 스레드 풀을 이용하여 재사용하는 경우 이전 요청에서 처리했던 데이터를 모두 지워주지 않으면 다음 요청에서 오염이 발생할 수 있다.

그리고 스레드가 하나의 요청을 처리하는 과정에서 메서드 시그니처로 전달 하며 데이터를 사용하지 않고, 마치 전역으로 쓰듯 데이터를 사용하는 과정에서 데이터의 변경을 추적하기 까다로운 점이 있다.



#### 스레드 로컬을 사용하는 메모리 구조

<figure><img src="../../../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

스레드 로컬이 GC에 소멸된다니, 스레드 내부 영역에서 공유할 수 있는 데이터라고 했을 때 스택이라고 생각할 수 있지만 힙 영역에 스레드 로컬 맵 객체로 보관된다.

그렇기 때문에 스레드 로컬을 모두 사용 했다면 `remove` 와 같이 데이터 정리를 해주지 않는 한 GC 가 동작하지 않아 메모리 누수가 발생할 수 있다.



ThreadLocal 동작 과정 설명 \[작성중]



### 사용 방법

***

{% tabs %}
{% tab title="스레드를 직접 생성하고 종료" %}
```java
static class CustomThread extends Thread {
	// ThreadLocal 선언
	private static final ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0);
	private final Integer number;

	public CustomThread(Integer number) {
		this.number = number;
	}

	@Override
	public void run() {
		// thread 로컬에 있는 초기값 get()
		int initValue = threadLocal.get();
		System.out.println("[" + number + " Thread]\nThreadLocal initValue: " + initValue );
		// thread 로컬에 값 set()
		threadLocal.set(number);
		// thread 로컬에 셋팅된 값 get()
		int setValue = threadLocal.get();
		// 출력으로 확인해보기
		System.out.println("[" + number + " Thread]\nThreadLocal setValue: " + setValue );
	}
}
```

<figure><img src="../../../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>
{% endtab %}

{% tab title="스레드 풀 사용 후 제거 X" %}
```java
@SpringBootApplication
public class ThreadLocalApplication {
	// pool 선언
	private static final ExecutorService executorService = Executors.newFixedThreadPool(3);

	public static void main(String[] args) {
		// thread 시작
		for (int number = 1; number <= 5; number++) {
			final CustomThread thread = new CustomThread(number);
			executorService.execute(thread);
		}
		// 종료
		executorService.shutdown();
	}
}
```

<figure><img src="../../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>
{% endtab %}

{% tab title="스레드 풀 사용 후 제거 O" %}
```java
@Override
public void run() {
	// ThreadLocal에 있는 초기값 get()
	int initValue = threadLocal.get();
	// ThreadLocal에 값 set()
	threadLocal.set(number);
	// ThreadLocal에 셋팅된 값 get()
	int setValue = threadLocal.get();
	// 출력으로 확인해보기
	System.out.println("[" + number + " Thread]\nThreadLocal initValue: " + initValue + " setValue:" + setValue );
	threadLocal.remove(); <--- remove() 추가
}
```
{% endtab %}
{% endtabs %}

스레드 로컬을 사용하면, 스프링에서 사용자의 요청이 하나의 스레드에서 직접 처리하는 동안 메서드 시그니처를 통해 공유해야 할 데이터를 전달하지 않고, 스레드 로컬을 통해 값을 꺼낼 수 있다.

이로 인해 자바 생태계에서도 자주 쓰이는데 대표적인 예시 중 하나는 `SpringSecurity` 가 있다.



### 스프링 시큐리티 예시 \[작성중]

***



