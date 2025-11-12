---
description: 자바의 버전이 증가하면서 나날이 발전한 스레드 관리 방법을 알아보기
---

# 자바에서 스레드를 조작하는 방법

## 스레드의 생명 주기

<figure><img src="../../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

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



{% embed url="https://www.baeldung.com/java-thread-lifecycle" %}



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



### Java 5 이전

***

#### Thread

스레드 생성을 위해 자바에서 미리 구현해둔 클래스이며, 유저 수준의 스레드이다.



#### **스레드를 생성하는 방법**

스레드 객체를 활용하여 멀티 스레드로 논리적 흐름을 분리하고 싶다면 `Thread` 상속 및 `run` 메서드 오버라이딩을 통해 새로운 스레드에서 코드를 실행할 수 있다.

{% tabs %}
{% tab title="ThreadExample.java" %}
```java
public class ThreadExample extends Thread{

    @Override
    public void run() {
        System.out.println("스레드 예시 실행");
    }
}

...

ThreadExample thread = new ThreadExample();
thread.start();

```
{% endtab %}
{% endtabs %}

> _**`start()`****&#x20;****메서드를 호출 했는데 어떻게****&#x20;****`run()`****&#x20;****메서드가 실행 되는걸까?**_

```java
    public void start() {
        synchronized (this) {
            // zero status corresponds to state "NEW".
            if (holder.threadStatus != 0)
                throw new IllegalThreadStateException();
            start0();
        }
    }

```

단순 오버라이딩 된 메서드인 run() 을 호출하면, 메인 스레드에서 객체의 메서드를 호출하는 것 과 다름없다.

메인 스레드와 분리 되어 새로운 스레드의 공간을 할당하고, 코드를 실행하는 데에 있어 JVM 의 도움이 필요하다.



**start() 메서드를 호출하면 JVM 에서 일어나는 과정**

{% stepper %}
{% step %}
JNI를 통해 OS의 커널에 스레드 생성을 요청
{% endstep %}

{% step %}
스레드 생성을 위한 공간 할당

* OS는 새로운 스레드를 위해 커널 스택과 TCB 등의 자원 할당
{% endstep %}

{% step %}
커널 스레드의 정보를 자바 스레드 객체에 업데이트

* 커널 스레드의 스레드 ID, 상태, 이름 등
{% endstep %}

{% step %}
JVM은 해당 스레드가 자바 코드를 실행하기 위한 런타임 데이터 영역 초기화
{% endstep %}

{% step %}
OS 스케줄러 큐에 해당 스레드 배치

* 스레드 생성이 완료되면 OS 스케줄러의 준비 큐에 배치하고, 자바 스레드 상태로는 `RUNNABLE` 상태로 전환
{% endstep %}

{% step %}
스케줄링 차례가 오면 CPU 자원을 할당 받아 `run()` 메서드 실행
{% endstep %}
{% endstepper %}

{% embed url="https://medium.com/@unmeshvjoshi/how-java-thread-maps-to-os-thread-e280a9fb2e06" %}



#### 주요 특징

*





#### Runnable



### Java 5

***

#### Callable

#### Future

#### Executor

#### ExecutorService

#### Executors



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



