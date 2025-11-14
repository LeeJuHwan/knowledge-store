---
description: 자바 5 버전
---

# Future 와 Callable 그리고 스레드 풀

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

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

과거 버전에서는 `Thread()` 의 `start()` 메서드가 호출 되면서 무조건 한 개의 스레드가 생성되고 종료되기를 반복하는 사이클로 오버헤드에 대한 문제가 있었다.

이 과정을 개선하고자 미리 스레드를 만들어두고 매번 생성하지 않고 재사용할 수 있는 스레드 풀 개념이 착안되었다.

***

#### Executor

`Runnable` 객체로 구현한 작업의 내용을 실행 해주는 인터페이스로 `execute()` 메서드 하나만을 제공한다.

과거 버전에서는 스레드를 직접 개발자의 논리적 흐름 안에서 관리했다면, `Executor` 인터페이스를 구현한 구현체 내부에서 스레드를 관리하도록 변경하고, 오로지 작업에 대한 수행에 초점을 맞출 수 있도록 한 단계 더 추상화 하였다.

{% tabs %}
{% tab title="Executor 정의" %}
```java
public interface Executor {

    /**
     * Executes the given command at some time in the future.  The command
     * may execute in a new thread, in a pooled thread, or in the calling
     * thread, at the discretion of the {@code Executor} implementation.
     *
     * @param command the runnable task
     * @throws RejectedExecutionException if this task cannot be
     * accepted for execution
     * @throws NullPointerException if command is null
     */
    void execute(Runnable command);
}
```
{% endtab %}

{% tab title="Executor 예시" %}
```java
public class ThreadExample implements Executor {

    @Override
    public void execute(Runnable command) {
        new Thread(command).start();
    }
    

Executor executor = new ThreadExample();
executor.execute(
        () -> System.out.println("Executor in thread: " + Thread.currentThread().getName())
);

```
{% endtab %}
{% endtabs %}

`Thread` 와 동작하는 방식에 있어 다를 바 없으며, 실행 방식도 비슷하다.

하지만, `Runnable` 은 작업 그 자체를 의미하고, `Executor`는 작업을 받아 실제 스레드에 할당하는 관리자 역할을 한다.

즉, 이 예제는 `Executor`의 구조를 보여주기 위한 가장 기초적인 구현일 뿐이며, 실제 프로덕션 환경에서는 이 인터페이스를 구현한 스레드 풀을 사용하여 스레드 생성 비용 문제를 해결한다.

#### ExecutorService

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

`ExecutorService`는 `Executor`를 상속받아, 작업 실행뿐만 아니라 스레드 풀의 종료와 생성된 스레드의 생명주기 관리 기능을 추가로 제공한다.



#### ExecuteService 가 지원하는 비동기 작업 기능

**submit**

* 작업을 수행하는 메서드이며, 결과로 Future 를 반환한다. 수행한 함수의 결과를 얻기 위해 `Future` 객체가 제공하는 메서드(`get`)를 이용해야 한다.

**invokeAll**

* 여러 작업을 동시에 요청하고 모든 작업이 완료될 때까지 대기 한다. 실행 결과로 `List<Future>`를 반환하는데, 리스트에 담긴 결과의 순서는 인자로 전달한 작업의 순서와 동일함이 보장된다.

{% hint style="info" %}
**예시**

* input: \[Task A, Task B, Task C]
* execution: `Task B`가 0.1초,  `Task A`는 1초가 걸렸다.
* output: \[Task A Future, Task B Future, Task C Future] 가 반환된다.
{% endhint %}

```java
  /**
     * Executes the given tasks, returning a list of Futures holding
     * their status and results when all complete.
     * {@link Future#isDone} is {@code true} for each
     * element of the returned list.
     ...
  **/
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
     throws InterruptedException;

<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                          long timeout, TimeUnit unit)
throws InterruptedException;
```

**invokeAny**

* 여러 작업을 제공했을 때 가장 빨리 완료된 한 개의 작업 결과가 나올 때 까지 블로킹 방식으로 요청하며, <mark style="color:red;">**모든 작업을 동시에 수행 하지만 가장 빨리 완료된 결과 하나만 작업의 반환 타입으로 반환하고, 가장 빨리 호출된 작업 외에 나머지 작업들은 모두 취소된다**</mark>.

```java
 /**
     * Executes the given tasks, returning the result
     * of one that has completed successfully (i.e., without throwing
     * an exception), if any do. Upon normal or exceptional return,
     * tasks that have not completed are cancelled.
     * The results of this method are undefined if the given
     * collection is modified while this operation is in progress.
     ...
**/
<T> T invokeAny(Collection<? extends Callable<T>> tasks)
    throws InterruptedException, ExecutionException;

<T> T invokeAny(Collection<? extends Callable<T>> tasks,
                long timeout, TimeUnit unit)
    throws InterruptedException, ExecutionException, TimeoutException;

```



{% hint style="info" %}
#### submit 과 execute 의 차이

* `execute`: 반환 타입이 `void`로 작업의 실행 결과나 작업의 상태를 알 수 없다.
* `submit`: 작업을 할당하고 `Future` 타입의 결과값을 받는다. 결과가 반환 되어야해서 `Callable`을 구현한 작업을 인자로 지정하지만, 메서드 오버로딩을 통해 `Runnable` 객체를 사용할 수 있도록 제공한다.
{% endhint %}

{% tabs %}
{% tab title="ExeuctorService 의 submit 메서드 오버로딩" %}
```java
<T> Future<T> submit(Callable<T> task);
<T> Future<T> submit(Runnable task, T result);
Future<?> submit(Runnable task);
```
{% endtab %}

{% tab title="다양한 라이프사이클 관리 메서드 제공" %}
```java
void shutdown();
List<Runnable> shutdownNow();
boolean isShutdown();
boolean isTerminated();
boolean awaitTermination(long timeout, TimeUnit unit)
    throws InterruptedException;
```

* `shutdown`: 새로운 작업을 더이상 받지 않고 실행중인 작업이 완전히 종료되면 안전하게 종료
* `shutdownNow`: 현재 작업중인 작업을 모두 중단하고, 아직 실행되지 못하고 큐에 쌓여있던 작업들을 반환함
* `isShutdown`: 작업 수행 종료 여부
* `isTerminated`: `shutdown` 메서드를 호출 했을 때 실행중이던 작업들마저 모두 종료 되었는지 여부 반환
* `awaitTermination`: `shutdown` 메서드 호출 후 일정 시간까지 대기하며, 일정 시간 내 작업이 모두 종료 되었는지 여부 반환
{% endtab %}
{% endtabs %}

#### ExecutorService 의 작업 종료

**shutdown**

`ExecutorService` 를 통해 작업을 실행하는 경우, 명시적으로 `shutdown` 을 호출하지 않으면 메인 스레드에서 다음 작업을 무한정 대기하는 상태로 전환되어 프로세스가 종료되지 않는다.

작업이 끝났다면 명시적으로 `shutdown` 메서드를 호출하여 정상적으로 프로세스를 종료 해주어야한다.

* 하지만, 작업이 계속 실행 중이라면 작업이 끝날 때 까지 기다리는 `Graceful shutdown` 방식을 사용하기 때문에 `shutdown` 메서드로 작업이 종료되지 않을 수 있다.



**shutdownNow**

인터럽트를 발생 시키기 때문에 스레드 내부 로직에서 인터럽트가 발생할 경우에 대한 후처리 로직을 추가할 수 있다.

{% tabs %}
{% tab title="예시" %}
```java
public class ExecuteServiceMain {

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        executorService.execute(
                () -> {
                    System.out.println("Executor in thread: " + Thread.currentThread().getName());
                    while (true) {
                        if (Thread.currentThread().isInterrupted()) {
                            System.out.println("Interrupted");
                            break;
                        }
                    }
                }
        );
        executorService.shutdownNow();
    }
}

// 콘솔 출력
// Executor in thread: pool-1-thread-1
// Interrupted
```
{% endtab %}
{% endtabs %}

#### ExecutorService 스레드 풀

{% hint style="info" %}
#### ScheduledExecutorService 인터페이스

내용
{% endhint %}







#### Executors



{% embed url="https://www.baeldung.com/java-executor-service-tutorial" %}

{% embed url="https://www.codingshuttle.com/blogs/java-executor-framework-tutorial-simplifying-multithreading-with-executor-service/" %}
