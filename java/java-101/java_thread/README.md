---
description: 자바의 버전이 증가하면서 나날이 발전한 스레드 관리 방법을 알아보기
---

# 자바에서 스레드를 조작하는 방법

## 스레드의 생명 주기

<figure><img src="../../../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

#### NEW

스레드 객체가 새로 생성된 상태

```java
Thread thread = new Thread(() -> {...});
```

#### RUNNABLE

스레드가 실행 가능하도록 준비가 완료된 상태로, JVM 이 해당 상태에 있는 스레드를 OS 스케줄링 대상으로 요청한다.

```java
Thread thread = new Thread(() -> {...});
thread.start();
```

#### BLOCKED

`synchronized`를 통해 모니터락 획득을 대기할 때 `BLOCKED` 상태에서 대기한다

#### WAITING

스레드가 특별한 연산하는 행위 없이 무작정 기다리는 상태로, 다른 스레드에 의해 깨어나길 기다린다.

* `Object.wait();`
* `Thread.join();`
* `LockSupport.park();`

```java
Thread thread = new Thread(() -> {...});
thread.start();
thread.join();  // <- 이 때 WAITING 상태로 변경
```

#### TIME\_WAITING

특정 시간동안 기다리고 있는 상태로, 시간이 지나거나 다른 스레드에 의해 깨어나면 현재 상태에서 벗어난다.

* `Object.wait(ms);`
* `Thread.join(ms);`
* `Thread.sleep(ms);`

```java
Thread thread = new Thread(() -> {Thread.sleep(1000)});
thread.start();
```

#### TERMINATED

스레드가 실행해야 할 연산이 모두 완료 되었거나 예외가 발생하여 실행이 종료된 상태로, 종료되면 스레드를 다시 살릴 수 없다.



## 버전 별 자바에서 스레드를 조작하는 방법

자바의 버전이 증가하면서 어떤 고수준의 스레드 관련 API 를 제공하고, 어떻게 발전 해왔는지 살펴본다.

```
├─ Java 5 이전: Thread, Runnable
├─ Java 5: Callable, Future, Executor, ExecutorService, Executors
├─ Java 7: Fork/Join, RecursiveTask
├─ Java 8: CompletableFuture
├─ Java 9: Flow
└─ Java 21: Virtual Thread
```

### [Java 5 이전, Thread 와 Runnable](thread-runnable.md)

자바 최초의 멀티 스레드 모델로 등장하여, 스레드를 직접 생성 및 제어하며 다른 스레드를 조작할 수 있게 되었다.

이로 인해 스레드가 작업 중 블락 상태로 대기하고 있는 경우, 다른 스레드를 통해 다른 작업을 수행할 수 있게 되었지만 작업의 반환 값과 예외 처리가 없는 치명적인 단점이 존재했다.

### [Java 5, Future 와 Callable 그리고 ExeuctorService 스레드 풀](future-callable.md)

스레드를 생성하고 종료하는 데 있어 자원을 낭비하던 단점을 보완하여, 스레드 풀을 활용하여 스레드를 재사용하였다.

그와 동시에, `Callable` 의 등장으로 인해 스레드의 작업 결과를 반환할 수 있게 되었으며 비동기 특성 상 언제 끝나는지 알 수 없는 시점에서 결과를 제어 하기 위해 `Future` 인터페이스를 통해 대기 후 값을 반환 받을 수 있도록 제공하였다.

또한 `Executors` 팩터리 클래스를 통해 여러 스레드 풀을 제공하여 사용자 환경에 맞는 스레드 풀을 선택하여 사용할 수 있도록 제공했다.

* 하지만, `Exeuctors` 는 사용자 편의상 제공하는 클래스이기 때문에 운영 환경에서는 무한히 스레드 풀이 증가하며 메모리 사용량 증가를 주의해야한다.

### [Java 7, 새로운 스레드 풀을 디자인하다.](fork-join-pool.md)

***

공용 큐를 사용하는 스레드 풀이 서로 작업을 할당 받기 위해 대기가 발생 하여 스레드가 일을 하지 않는 상황이 있다.

이런 상황에서 더 성능적 최적화를 위해 ForkJoinPool 방식이 등장 하였고, 각 작업 스레드 마다 고유의 큐를 보유하며 만약 작업 스레드가 대기 상태로 전환 되는 경우 쉬지 않고 다른 스레드의 작업을 가져와 남은 작업들을 마무리 하는데 돕는 알고리즘을 활용하였다.

### [Java 8, Future 를 개선한 CompletableFuture](future-completablefuture.md)

***

여러 작업이 작업 간의 연결이 어렵고, 결과를 제어하기 위해 작업이 끝나기 까지 기다리는 블락킹 상태를 개선하기 위해 `CompletableFuture` 가 등장하였다.

여러 콜백 메서드를 제공하여 작업 할당과 동시에 다른 작업을 수행할 수 있도록 조합이 가능했으며, 콜백을 사용하는 경우 조합된 작업 간의 블락킹이 생기지 않아 논 블락킹 구조로 동작하는 성능적 이점을 챙겼다.

그리고, 사용자 정의 스레드 풀을 사용했던 `Future` 와 달리 `ForkJoinPool` 의 공용 스레드 풀을 이용하며 작업 스틸 알고리즘을 통해 스레드가 쉬는 시간 없이 동작할 수 있도록 최적화 되었다.





{% embed url="https://www.baeldung.com/java-thread-lifecycle" %}
