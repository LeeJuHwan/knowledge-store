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



#### **start() 메서드를 호출하면 JVM 에서 일어나는 과정**

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

* 유저 스레드와 커널 스레드가 1:1 매핑되어 블락이 가능하고, 블락 시 다른 스레드를 준비 큐에서 실행 시킬 수 있다.
* `start()` 명령어를 호출하면 JNI 를 통해 OS 에게 시스템 콜을 보내어 스레드를 생성하고 매핑하는 과정이 오버헤드가 발생한다.
* `run()` 메서드를 오버라이드 하고, run() 메서드가 아닌 `start()` 메서드를 실행 해야 멀티 스레드 환경에서 코드를 실행할 수 있다.
* `Thread` 는 새로운 객체의 상속 관계를 만들며, 다중 상속이 불가능한 자바에서 상속할 수 있는 부모 타입이 제한된다.



#### Runnable

스레드를 조작하는 `run()` 메서드만 제공하는 함수형 인터페이스이다. 람다식으로 구현할 수도 있고, 새로운 객체를 생성하여 구현할 수도 있다.

{% tabs %}
{% tab title="Runnable.java" %}
```java
/**
 * Represents an operation that does not return a result.
 *
 * <p> This is a {@linkplain java.util.function functional interface}
 * whose functional method is {@link #run()}.
 *
 * @author  Arthur van Hoff
 * @see     java.util.concurrent.Callable
 * @since   1.0
 */
@FunctionalInterface
public interface Runnable {
    /**
     * Runs this operation.
     */
    void run();
}
```
{% endtab %}
{% endtabs %}

`Runnable` 인터페이스는 결과를 반환하지 않는 메서드를 제공한다. 그렇다는건 새로운 객체를 생성하여 `run()` 을 직접 구현하더라도, 스레드가 실행한 흐름에 대한 결과를 반환하지 못한다.

#### 스레드를 생성하는 방법

{% tabs %}
{% tab title="Runnable 구현" %}
```java
public class ThreadExample implements Runnable{

    @Override
    public void run() {
        System.out.println("스레드 예시 실행");
    }
}

...
Thread thread = new Thread(new ThreadExample());
thread.start();
```
{% endtab %}

{% tab title="람다 사용" %}
```java
public class ThreadMain {

    public static void main(String[] args) {
        Thread thread = new Thread(() -> System.out.println("스레드 예시 실행"));
        thread.start();
    }
}
```
{% endtab %}
{% endtabs %}

자바에서 단순하게 스레드를 다루어야 한다면, 부모의 많은 기능을 상속 받는 `Thread` 보다 `Runnable` 을 익명 객체로 제공하든, 새로운 객체를 생성하여 구현체로 다루든 한 가지 메서드에 대한 기능만 구현하는 것이 더 효율적이다.

그렇기 때문에 대부분의 상황에서 `Runnable` 인터페이스를 구현하여 문제를 해결할 수 있다.



#### Thread 와 Runnable 의 한계

1. `run()` 메서드를 `void` 로 제공하기 때문에 반환 값을 사용할 수 없다.
2. `start()` 명령어가 실행 되면 JNI 의 OS 시스템 콜로 인해 매번 스레드를 생성하는 오버헤드가 발생한다.



### Java 5

***

자바 5 이전에서 `Thread` 와 `Runnable` 이 갖고 있는 한계점을 보완하며 `Callable` 과 `Future` 가 등장하였으며, 이러한 스레드를 관리할 수 있는 여러 객체들도 같이 등장하였다.



#### Callable

기존 문제점에서 `Runnable` 인터페이스를 구현했을 때 반환 값을 사용할 수 없다는 단점을 보완하며 등장하였다.

`Callable` 은 제네릭 타입을 선언하여 반환 값을 지정할 수 있는 함수형 인터페이스이다.

`Runnable` 처럼 작업 단위를 구성하는 인터페이스로, 실제 구현된 내용의 실행을 제어하지 못하고 다른 호출자에 의해 실행되는 구조이다.

{% tabs %}
{% tab title="Callable 정의" %}
```java
/**
 * A task that returns a result and may throw an exception.
 * Implementors define a single method with no arguments called
 * {@code call}.
 *
 * <p>The {@code Callable} interface is similar to {@link
 * java.lang.Runnable}, in that both are designed for classes whose
 * instances are potentially executed by another thread.  A
 * {@code Runnable}, however, does not return a result and cannot
 * throw a checked exception.
 *
 * <p>The {@link Executors} class contains utility methods to
 * convert from other common forms to {@code Callable} classes.
 *
 * @see Executor
 * @since 1.5
 * @author Doug Lea
 * @param <V> the result type of method {@code call}
 */
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```
{% endtab %}

{% tab title="Callable 예시" %}
```java
public class CallableMain {

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        Callable<String> callable = () -> {
            final String result = "Callable into Thread: " + Thread.currentThread().getName();
            System.out.println(result);
            return result;
        };

        Future<String> future = executorService.submit(callable);
        try {
            String result = future.get();
            System.out.println("result = " + result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        } finally {
            executorService.shutdown();
        }

    }

}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
#### Callable vs Runnable

1. `Runnable` 은 메서드의 반환 값을 사용할 수 없음
2. `Callable` 은 메서드의 예외 처리를 통해 `CheckedException` 처리 가능
{% endhint %}

`Callable` 은 자신의 `call()` 메서드를 통해 결과 값을 제네릭으로 반환할 수 있는데, 비동기로 수행되는 경우 이 작업이 언제 끝날지 모르기 때문에 값을 직접적으로 반환하지 않고 `Future` 객체를 통해 반환한다.

#### Future

`Callable` 이 구현하는 작업은 비동기로 동작하는 경우 함수가 반환되는 시점이 명확하지 않기 때문에 `Future` 가 필요한데, 이 `Future` 는 비동기 작업의 미래에 완료될 결과를 나타내는 인터페이스이다. 작업이 완료될 때까지 기다리거나, 작업 완료 여부를 확인하고, 필요 시 작업을 취소할 수 있는 기능을 제공한다.

{% tabs %}
{% tab title="Future 정의" %}
```java
public interface Future<V> {

    boolean cancel(boolean mayInterruptIfRunning);

    boolean isCancelled();

    boolean isDone();

    V get() throws InterruptedException, ExecutionException;

    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

* get(): 작업이 완료될 때까지 기다린 후 결과를 반환하며, 만약 작업이 실패하면 예외를 발생시킨다.
* isDone(): 작업의 완료 여부를 반환한다.
* isCancelled(): 작업의 취소 여부를 반환한다.
* cancel(): 작업을 취소 시키며,그 후 `isDone()` 메서드는 항상 `true` 를 반환한다.
{% endtab %}

{% tab title="Future 예제" %}
```java
public class CallableMain {

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        Callable<String> callable = () -> {
            final String result = "Callable into Thread: " + Thread.currentThread().getName();
            System.out.println(result);
            return result;
        };

        Future<String> future = executorService.submit(callable);
        try {
            String result = future.get(); // Callable 의 작업의 실제 반환 값
            System.out.println("result = " + result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        } finally {
            executorService.shutdown();
        }

    }

}
```
{% endtab %}
{% endtabs %}



#### 자바 5 에서 제공하는 스레드의 생성과 관리를 위한 스레드 풀

과거 버전에서는 `Thread()` 의 `start()` 메서드가 호출 되면서 무조건 한 개의 스레드가 생성되고 종료되기를 반복하는 사이클로 오버헤드에 대한 문제가 있었다.

이 과정을 개선하고자 미리 스레드를 만들어두고 매번 생성하지 않고 재사용할 수 있는 스레드 풀 개념이 착안되었다.

***

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



