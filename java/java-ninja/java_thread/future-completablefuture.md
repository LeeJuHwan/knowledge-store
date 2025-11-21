---
description: 자바 8 버전
---

# Future 를 개선한 CompletableFuture

자바 5 버전에서 등장한 Future 는 혁신적인 기능들을 갖고 왔지만, 그럼에도 불구하고 실용적인 부분에서 [개선해야 할 여지](future-callable.md#future-3)가 보였다.

자바 8 에서는 위 Future 가 갖고 있는 단점을 보완한 새로운 동시성 모델이 등장하였다.

### CompletableFuture

***

`Future` 의 단점을 보완한 버전으로 `Future` 가 갖고 있는 속성을 그대로 구현하고 있으면서 추가적으로 `CompletionStage` 라는 인터페이스를 통해 비동기 연산의 결과를 연결할 수 있도록 구성 되어있다.

{% tabs %}
{% tab title="내부 코드" %}
```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {
...
}
```

* `Future`: 비동기 작업의 미래에 완료될 결과를 나타내는 인터페이스
* `CompletionStage`: 비동기 작업의 결과를 연결할 수 있도록 구성하는 인터페이스
{% endtab %}

{% tab title="사용 예시" %}
```java
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    System.out.println("Hello " + Thread.currentThread().getName());
    return "Hello";
});
String futureResult = future2.get();
System.out.println("futureResult = " + futureResult);
```

```
Hello ForkJoinPool.commonPool-worker-1
futureResult = Hello
```
{% endtab %}
{% endtabs %}

#### Future vs CompletableFuture

<table><thead><tr><th width="100" align="center">구분</th><th align="center">Future(Java5)</th><th align="center">CompletableFuture(Java8)</th></tr></thead><tbody><tr><td align="center">동작 방식</td><td align="center">Blocking(get 호출 시 대기)</td><td align="center">Non-blocking(Callback 등록 가능)</td></tr><tr><td align="center">조합/연결</td><td align="center">어려움 (get으로 꺼내서 다시 넣어야 함)</td><td align="center"><code>thenApply()</code>, <code>thenCompose()</code> 등으로 파이프라인 구성 가능</td></tr><tr><td align="center">예외 처리</td><td align="center">try-catch 강제</td><td align="center"><code>exceptionally()</code>, <code>handle()</code>을 통한 예외 처리</td></tr><tr><td align="center">수동 완료</td><td align="center">불가능</td><td align="center"><code>complete()</code> 메서드로 강제 완료 가능</td></tr></tbody></table>

#### 비동기 작업 실행

***

> 반환 값이 없는 비동기 실행 메서드 - `runAsync()`

```java
CompletableFuture<Void> future1 = CompletableFuture.runAsync(() -> {
    System.out.println("Hello Run Async " + Thread.currentThread().getName());
});

future1.get();
```

> 반환 값이 존재하는 비동기 실행 메서드 - `supplyAsync()`

```java
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    System.out.println("Hello Supply Async " + Thread.currentThread().getName());
    return "Hello";
});
String futureResult = future2.get();
System.out.println("futureResult = " + futureResult);
```

**실행 결과**

```
Hello Run Async ForkJoinPool.commonPool-worker-1
Hello Supply Async ForkJoinPool.commonPool-worker-1
futureResult = Hello
```

* 반환 값 없는 경우 : `runAsync()`
* 반환 값 있는 경우 : `supplyAsync()`

스레드 풀은 기본적으로 `ForkJoinPool` 의 `commonPool()` 에서 얻어서 스레드를 획득함, 스레드 풀을 직접 제어하고 싶다면 메서드 시그니처에 `Executor` 를 추가하면 해당 스레드 풀에서 스레드를 획득한다.

<details>

<summary>사용자 정의 스레드 풀 사용 예시</summary>

```java
ExecutorService executorService = Executors.newFixedThreadPool(5);

CompletableFuture<Void> future1 = CompletableFuture.runAsync(() -> {
    System.out.println("Hello Run Async " + Thread.currentThread().getName());
},  executorService);

future1.get();

CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    System.out.println("Hello Supply Async " + Thread.currentThread().getName());
    return "Hello";
}, executorService);
String futureResult = future2.get();
System.out.println("futureResult = " + futureResult);
```

```
Hello Run Async pool-1-thread-1
Hello Supply Async pool-1-thread-2
futureResult = Hello
```

</details>

{% hint style="warning" %}
#### CompletableFuture 는 왜 ForkJoinPool 을 사용할까?

> `FokrJoinPool` 원리 간단하게 이해 하기

`ForkJoinPool` 의 특성 상 분할 정복 구조를 떠올려보면 모든 스레드가 각자의 큐에 작업을 대기 시켜놓은 뒤 실행한다.

그런 매커니즘을 기반으로 `Work Stealing` 알고리즘으로 동작하기 때문에, 다른 스레드가 작업 중이어서 큐에 대기중인 작업이 있다면, 다른 스레드가 유휴 시간을 갖지 않도록 작업을 가져와서 대신 처리 해주는 역할을 한다.



`CompletableFuture` 는 스레드가 작업을 수행한 뒤 결과가 나오기 까지 대기할 수 있지만, 작업 콜백 방식을 사용하고 있기 때문에 여러 작업이 수시로 실행되고 변경되는 특징 덕에 `ExecutorService` 에서 스레드는 작업이 할당 되기 까지 기다리는 시간을 최적화 하기 위해 `ForkJoinPool` 의 스레드를 사용한다.
{% endhint %}



#### 결과 값을 반환하는 메서드의 차이(get vs join)

***

> get()

`CompletableFuture` 객체가 `get()` 메서드를 호출하면, 스레드의 작업 결과 값이 반환되기 까지 기다린다. 이 상황에서 스레드는 Blocking 되어 다른 스레드가 점유할 수 없으며, 만약 실행 도중 예외가 발생하면 `CheckedException` 예외를 발생 시킨다.

`get()` 메서드는 `Future` 인터페이스와 호환 되는 다른 클래스 모두 구현하여 사용중이고, `CompletableFuture` 가 등장할 당시 하위 버전에서도 동일한 기능을 제공하기 위해 해당 메서드를 그대로 구현하여 남겨두었다.



> join()

`CompletableFuture` 객체가 `join()` 메서드를 호출하면, `get()` 메서드와 실행 방식은 동일하게 스레드가 작업의 결과를 반환 받기 위해 블락킹 상태에 있지만, 실행 도중 예외가 발생할 경우 `UncheckedException` 이 발생한다.

`join()` 메서드는 `CompletableFuture` 클래스에만 존재하며, 만약 다른 `Future` 인터페이스를 사용한다면 `get()` 메서드를 통해 호환을 보장해야한다. 하지만, `CompletableFuture` 만 사용한다면 `join()` 메서드를 사용하는 것을 권장한다.

<table data-header-hidden><thead><tr><th width="88.359375" align="center">메서드</th><th>예외 처리 (Exception)</th><th>인터럽트 (Interrupt) 처리</th><th>사용 권장 상황</th></tr></thead><tbody><tr><td align="center">get()</td><td><p>Checked Exception</p><p>(ExecutionException, InterruptedException)</p></td><td>인터럽트 발생 시 즉시 예외(<code>InterruptedException</code>)를 던짐. 반드시 <code>try-catch</code> 필요</td><td>일반적인 스레드 제어나 <code>Future</code> 인터페이스 호환성이 필요할 때</td></tr><tr><td align="center">join()</td><td><p>Unchecked Exception</p><p>(CompletionException)</p></td><td>인터럽트 발생 시 Checked 예외를 던지지 않고 <code>CompletionException</code>으로 감쌈</td><td>Stream API나 람다식 내부에서 사용하여 코드를 간결하게 만들 때</td></tr></tbody></table>



#### 작업 콜백

***

아래 메서드는 모두 이전 단계의 결과 값을 함수형 인터페이스 인수로 사용한다.

* `thenApply`: 이전 결과 값을 활용하여 새로운 연산을 통해 값을 반환한다.
* `thenAccept`: 이전 결과 값을 활용하여 새로운 연산을 하되 값을 반환하지 못한다.
* `thenRun`: 이전 결과 값을 인수로 활용하지 않고, 새로운 작업을 실행 한다.

{% tabs %}
{% tab title="thenApply" %}
```java
/**
 * 인자로 받은 Function을 사용하여 다음 연산 처리
 * Function의 반환 값을 가지고 있는 CompletableFuture<U> 반환
 */
public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn) {
    return uniApplyStage(null, fn);
}
```

> 예시

```java
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    System.out.println("Hello Supply Async " + Thread.currentThread().getName());
    return "Hello";
}).thenApply(
    result -> result + " thenApply - " + Thread.currentThread().getName()
);
String futureResult = future2.get();
System.out.println("futureResult = " + futureResult);
```

```
futureResult = Hello thenApply - main
```
{% endtab %}

{% tab title="thenAccept" %}
```java
/**
 * Consumer를 인자로 받고, 결과를 CompletableFuture<Void> 로 반환
 * get() 호출 시 연산을 처리하고 Void 유형의 인스턴스를 반환
 */
public CompletableFuture<Void> thenAccept(Consumer<? super T> action) {
    return uniAcceptStage(null, action);
}
```

> 예시

```java
CompletableFuture<Void> future2 = CompletableFuture.supplyAsync(() -> {
    System.out.println("Hello Supply Async " + Thread.currentThread().getName());
    return "Hello";
}).thenAccept(result -> System.out.println(result + " thenAccept - " + Thread.currentThread().getName()));
future2.get();
```

```
Hello thenAccept - main
```
{% endtab %}

{% tab title="thenRun" %}
```java
/**
 * Runnable을 인자로 받고, 결과를 CompletableFuture<Void> 로 반환
 * 이전 작업의 결과값에 의존하지 않고, 작업이 완료되었다는 사실만 트리거로 삼아 실행한다.
 */
public CompletableFuture<Void> thenRun(Runnable action) {
    return uniRunStage(null, action);
}
```

> 예시

```java
CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> {
    return "Thread: " + Thread.currentThread().getName();
}).thenRun(() -> {
    System.out.println("Thread: " + Thread.currentThread().getName());
});

future.get();
```

```
thenRun Thread: main
```
{% endtab %}
{% endtabs %}

<table><thead><tr><th width="157.484375" align="center">메서드</th><th>사용 목적</th><th>예시</th></tr></thead><tbody><tr><td align="center"><code>thenApply</code></td><td>이전 결과를 활용하여 반환 타입이 <mark style="color:red;">있는</mark> 로직을 호출할 때</td><td>이전 결과 값을 활용하여 새로운 객체를 만들 때</td></tr><tr><td align="center"><code>thenAccept</code></td><td>이전 결과를 활용하여 반환 타입이 <mark style="color:red;">없는</mark> 로직을 호출할 때</td><td>이전 작업의 결과 값과 함께 로그를 남길 때</td></tr><tr><td align="center"><code>thenRun</code></td><td>이전 결과의 반환 값을 사용하지 않고 다른 작업을 수행해야 할 때</td><td>비동기 작업 블럭에서 공유되는 자원을 활용하여 <code>thenRun</code> 메서드로 로그를 남길 때</td></tr></tbody></table>

{% hint style="info" %}
#### 작업 콜백의 비동기 메서드와 동기 메서드의 차이

`thenApplyAsync`, `theAcceptAsync`, `thenRunAsync` 와 같이 작업 콜백 메서드를 비동기로 지원하는 메서드도 같이 제공한다.

이 때 동기 메서드와 비동기 메서드의 차이는 **후속 작업을 어떤 스레드에서 실행할 것인가**에서 나뉜다.

* 동기:이전 작업을 수행한 스레드를 그대로 재사용하려고 시도 -> 단, 너무 빨리 끝날 경우 호출한 스레드에서 실행
* 비동기: 무조건 스레드 풀에 새로운 작업을 제출하여 실행함
{% endhint %}



#### 비동기 작업 조합 (작성중)

***

**thenCompose()**



**thenCombine()**



#### 예외 처리 (작성중)

***

**handle()**

**exceptionally()**



#### 병렬 처리 (작성중)

***

**allof()**

**anyOf()**





<details>

<summary>참고 자료</summary>

* [https://www.javacodegeeks.com/guide-to-completablefuture-join-vs-get.html](https://www.javacodegeeks.com/guide-to-completablefuture-join-vs-get.html)
* [https://www.baeldung.com/java-completablefuture-join-vs-get](https://www.baeldung.com/java-completablefuture-join-vs-get)
* [https://11st-tech.github.io/2024/01/04/completablefuture/](https://11st-tech.github.io/2024/01/04/completablefuture/)
* [https://mangkyu.tistory.com/263](https://mangkyu.tistory.com/263)

</details>
