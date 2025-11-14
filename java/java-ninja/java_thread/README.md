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

### [Java 5, Future 와 Callable 그리고 ExeuctorService 스레드 풀](future-callable.md)

### Java 7

***

#### Fork/Join

#### RecursiveTask



### Java 8

***

#### CompletableFuture



### Java 9

#### Flow



### Java 21

#### Virtual Thread





{% embed url="https://www.baeldung.com/java-thread-lifecycle" %}
