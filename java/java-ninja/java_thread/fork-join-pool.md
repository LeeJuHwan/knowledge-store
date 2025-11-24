---
description: Java 7
---

# 스레드 풀을 새롭게 디자인하다

자바 5 버전에서 `Executors` 를 활용하면 편리하게 스레드 풀의 코어 사이즈만 인자로 넘겨 스레드 풀을 제공받을 수 있었다.

하지만 내부적으로 `BlockingQueue` 설정이 불가능 했기 때문에 운영 환경에서는 적합하지 않았으며, 이에 따라 `ThreadPoolExecutor` 를 사용하여 세세하게 설정하여 사용했었어야 했다.

그리고, 모든 스레드가 하나의 큐에서 스케줄링 되어 작업을 할당 받았기 때문에 특정 작업이 끝나지 않으면 할당 받는 데 까지 대기를 해야 하는 성능적 이슈도 존재했다.



### ForkJoinPool

***

<figure><img src="../../../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

`ForkJoinPool` 은 기본적으로 분할 정복 방식을 통해 모든 스레드가 개별 큐를 보유하며, 해당 큐에서 스케줄링 하여 작업을 할당한다.

자신의 스레드에 속한 큐에서만 작업을 할당하면 작업의 실행 단위에 따라 늦게 끝나는 큐는 계속 대기를 하게 되는 성능적으로 불편한 사항이 존재한다.

그래서, 성능 최적화를 위해 `Work Stealing` 알고리즘을 적용하여 스레드가 쉬는 시간 없이 계속 일할 수 있도록 다른 큐에 있는 작업을 훔쳐온다.

`ForkJoinPool` 을 아무런 인자 없이 생성하면, 사용자의 CPU 프로세서 수 만큼의 큐 사이즈를 생성한다.

```java
    public ForkJoinPool() {
        this(Math.min(MAX_CAP, Runtime.getRuntime().availableProcessors()),
             defaultForkJoinWorkerThreadFactory, null, false,
             0, MAX_CAP, 1, null, DEFAULT_KEEPALIVE, TimeUnit.MILLISECONDS);
    }

```

#### Fork & Join

***

<figure><img src="../../../.gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

작업을 수행하면 작업을 하위 작업으로 분할 하여 작업을 담당하는 스레드의 Deqeue 에 저장한다.&#x20;

{% tabs %}
{% tab title="fork()" %}
```java
public final ForkJoinTask<V> fork() {
    Thread t; ForkJoinWorkerThread wt;
    ForkJoinPool p; ForkJoinPool.WorkQueue q;
    U.storeStoreFence();  // ensure safely publishable
    if ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread) {
        p = (wt = (ForkJoinWorkerThread)t).pool;
        q = wt.workQueue;
    }
    else
        q = (p = ForkJoinPool.common).submissionQueue(false);
    q.push(this, p, true);
    return this;
}

```
{% endtab %}

{% tab title="join()" %}
```java
public final V join() {
    int s;
    if ((s = status) >= 0)
        s = awaitDone(s & POOLSUBMIT, 0L);
    if ((s & ABNORMAL) != 0)
        reportException(s);
    return getRawResult();
}
```
{% endtab %}
{% endtabs %}

* `fork()`: 비동기로 작업을 실행 시키며 현재 스레드는 계속 실행 됨
* `join()`: `fork()` 로 실행된 작업의 결과를 기다림
* `invoke()`: `fork()`와 `join()`을 결합한 메서드로 작업을 제출하고, 결과가 반환될 때까지 기다림



`fork` 시 작업 스레드인 경우 자신의 스레드에 할당된 큐에 작업을 저장하고, 외부 스레드에서 수행되는 경우 공용 풀에 저장하여 관리한다.

* `ForkJoinWorkerThread`: 자신의 `WorkQueue(Deque)`에 작업을 저장하며 락 경합 없음
* `외부 스레드 (main 등)`: `ForkJoinPool`의 `SubmissionQueue`에 작업을 저장함

이렇게 큐가 나뉘게 되면서 현재 작업을 진행중인 스레드는 자신의 스레드에서 처리해야 할 작업에만 집중할 수 있고, 공용 풀에 존재하는 큐가 별도로 있음으로써 모든 작업 스레드가 작업을 할당 받기 위해 락 경합을 할 필요가 사라진다.

`Join` 과정에서 `awaitDone()` 메서드를 통해 작업이 완료될 때 까지 현재 스레드를 대기 상태로 만든다.

이 때, 현재 작업 스레드가 대기하면서 유휴 상태가 되지 않도록 다른 스레드의 큐를 탐색하여 남은 작업을 도와주는 작업훔치기 알고리즘을 수행한다.

그렇게 작업 스레드에 할당 된 모든 작업이 완료 되면 이 결과들을 종합하여 최종 결과를 반환한다.



#### WorkStealing

***

<figure><img src="../../../.gitbook/assets/image (100).png" alt=""><figcaption></figcaption></figure>

동작 매커니즘은 쉬는 스레드가 있는 경우, 다른 스레드의 작업을 자신의 큐로 가져와 할당하는 방식이다.

이 때, 스레드 1번 처럼 다른 스레드의 작업을 훔치는 입장은 큐의 `Tail` 에 있는 작업을 가져가고, 작업을 수행하는 스레드는 `Head` 에 있는 작업을 수행하여 남은 할 일을 줄여주는 과정이다.

#### RecursiveAction & RecursiveTask

***

`RecursiveAction` 과 `RecursiveTask` 모두 멀티 스레드 환경에서 병렬 처리를 돕는 추상 클래스이다.

{% tabs %}
{% tab title="RecursiveAction" %}
```java
public abstract class RecursiveAction extends ForkJoinTask<Void> {
    private static final long serialVersionUID = 5232453952276485070L;

    /**
     * Constructor for subclasses to call.
     */
    public RecursiveAction() {}

    /**
     * The main computation performed by this task.
     */
    protected abstract void compute();

    /**
     * Always returns {@code null}.
     *
     * @return {@code null} always
     */
    public final Void getRawResult() { return null; }

    /**
     * Requires null completion value.
     */
    protected final void setRawResult(Void mustBeNull) { }

    /**
     * Implements execution conventions for RecursiveActions.
     */
    protected final boolean exec() {
        compute();
        return true;
    }

}
```
{% endtab %}

{% tab title="RecursiveTask" %}
```java
public abstract class RecursiveTask<V> extends ForkJoinTask<V> {
    private static final long serialVersionUID = 5232453952276485270L;

    /**
     * Constructor for subclasses to call.
     */
    public RecursiveTask() {}

    /**
     * The result of the computation.
     */
    @SuppressWarnings("serial") // Conditionally serializable
    V result;

    /**
     * The main computation performed by this task.
     * @return the result of the computation
     */
    protected abstract V compute();

    public final V getRawResult() {
        return result;
    }

    protected final void setRawResult(V value) {
        result = value;
    }

    /**
     * Implements execution conventions for RecursiveTask.
     */
    protected final boolean exec() {
        result = compute();
        return true;
    }

}
```
{% endtab %}
{% endtabs %}

인터페이스의 상속 관계만 보더라도, 제네릭 타입이 서로 다른 것에서 부터 유추할 수 있는 특징이 있다.

<table><thead><tr><th width="160.20703125">인터페이스</th><th>용도</th><th width="138.9765625">오버라이딩</th></tr></thead><tbody><tr><td><code>RecursiveAction</code></td><td>반환 값이 없는 작업을 수행할 때 사용</td><td><code>compute()</code></td></tr><tr><td><code>RecursiveTask</code></td><td>작업 완료 후 특정 타입의 결과를 반환할 때 사용</td><td><code>compute()</code></td></tr></tbody></table>

이 인터페이스를 직접 구현하는 경우 `compute()` 메서드를 구현 해야 하는 데 작업의 정의 외에도 효율적인 스레드 사용을 위해 분할 정복 로직이 반드시 포함 되어야한다.



> RecursiveTask

자바에서는 `DualPivotQuickSort` 에서 병합 정렬을 사용할 때 `RunMerger` 클래스를 통해 분할 정복 알고리즘을 구현해두었다.

그 외에도 아래 코드 처럼 활용할 수 있으며, 대규모 데이터 처리 및 병렬 작업에서 최적화 된 성능으로 작업을 할 수 있다.

{% tabs %}
{% tab title="예시" %}
```java
// 1부터 n까지의 합을 구하는 예제 (분할 정복 적용)
public class ForkJoinExample {
    public static void main(String[] args) {
        ForkJoinPool pool = new ForkJoinPool();
        // 1부터 100까지 합
        SumTask task = new SumTask(1, 100); 
        Long result = pool.invoke(task);
        System.out.println("Sum of 1~100 is: " + result);
    }
}

class SumTask extends RecursiveTask<Long> {
    private static final int THRESHOLD = 10; // 임계값 (이보다 작으면 직접 계산)
    private final int start;
    private final int end;

    SumTask(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        int length = end - start;
        
        if (length <= THRESHOLD) {
            long sum = 0;
            for (int i = start; i <= end; i++) {
                sum += i;
            }
            return sum;
        } 
        
        int mid = (start + end) / 2;
        SumTask leftTask = new SumTask(start, mid);
        SumTask rightTask = new SumTask(mid + 1, end);

        leftTask.fork(); 
        return rightTask.compute() + leftTask.join();
    }
}
```
{% endtab %}
{% endtabs %}

1부터 100까지 더하는 코드를 작성할 때, 중간 값을 기준으로 작은 수 들은 다른 스레드가 더하고 큰 수는 메인 스레드가 더하여 서로 다 더한 값을 결과로 반환하는 코드이다.

만약 크기가 작은 데이터인 경우 되려 다른 스레드에게 작업을 할당하고 값을 반환 받기 까지 기다린 후 더하는 과정이 추가로 들기 때문에 단일 스레드에 비해 성능이 저하될 수 있다.



### 마무리

***

결론적으로 병렬 처리에 최적화 된 스레드 조작 방식으로, 작업 하고 있는 스레드가 대기가 발생할 경우 다른 스레드의 작업을 가져와 쉬지 않고 작업 하며 효율적인 성능을 나타낸다.





<details>

<summary>참고 자료</summary>

* [https://dev.to/headf1rst/discover-how-forkjoinpool-powers-javas-high-performance-parallel-processing-3pn7](https://dev.to/headf1rst/discover-how-forkjoinpool-powers-javas-high-performance-parallel-processing-3pn7)
* [https://www.geeksforgeeks.org/java/forkjoinpool-class-in-java-with-examples/](https://www.geeksforgeeks.org/java/forkjoinpool-class-in-java-with-examples/)
* [https://upcurvewave.tistory.com/653](https://upcurvewave.tistory.com/653)

</details>

