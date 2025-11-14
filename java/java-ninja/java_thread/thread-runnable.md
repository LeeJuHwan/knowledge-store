---
description: 자바 5 이전
---

# Thread 와 Runnable

### Thread

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



### Runnable

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



### Thread 와 Runnable 의 한계

1. `run()` 메서드를 `void` 로 제공하기 때문에 반환 값을 사용할 수 없다.
2. `start()` 명령어가 실행 되면 JNI 의 OS 시스템 콜로 인해 매번 스레드를 생성하는 오버헤드가 발생한다.
