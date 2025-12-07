# 동시성을 벗어나게 해줄 Thread Local

### 스레드 로컬이란?

해당 스레드만 접근할 수 있는 특별한 저장소를 말한다. 이를 쉽게 해석하면 물건을 보관 하는 보관함으로, 여러 사람이 같은 보관함에 물건을 보관 하더라도 사용하는 사람 별로 자신만의 물건을 관리할 수 있게 되는 기능이다.

***

자바에서 동시성을 해결하기 위해 크게 두 가지 방식을 사용한다.

1. synchronized 예약어
2. ThreadLocal 객체

1번 같은 경우는 스레드가 자원에 접근하기 위해 락 경합을 해야 하는 단점이 있다.

2번은 스레드가 요청을 처리하는 동안 고유 데이터 영역을 보장해주는 사물함 같은 원리이다.



#### 장점

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

각 스레드마다 별도의 내부 저장소를 제공하여, 같은 인스턴스의 스레드 로컬 필드에 접근하더라도 동시성 문제가 발생하지 않는다.

&#x20;

#### 주의점

스레드의 생성과 종료를 모두 제어하는 환경에서는 스레드 로컬 객체는 GC에 의해 소멸된다.

하지만, <mark style="color:red;">**스레드 풀을 이용하여 재사용하는 경우 이전 요청에서 처리했던 데이터를 모두 지워주지 않으면 다음 요청에서 오염이 발생할 수 있다.**</mark>

그리고 스레드가 하나의 요청을 처리하는 과정에서 메서드 시그니처로 전달 하며 데이터를 사용하지 않고, 마치 전역으로 쓰듯 데이터를 사용하는 과정에서 데이터의 변경을 추적하기 까다로운 점이 있다.



#### 스레드 로컬을 사용하는 메모리 구조

<figure><img src="../../../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

스레드 로컬이 GC에 소멸된다니, 스레드 내부 영역에서 공유할 수 있는 데이터라고 했을 때 스택이라고 생각할 수 있지만 힙 영역에 스레드 로컬 맵 객체로 보관된다.

그렇기 때문에 스레드 로컬을 모두 사용 했다면 `remove` 와 같이 데이터 정리를 해주지 않는 한 GC 가 동작하지 않아 메모리 누수가 발생할 수 있다.



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



### 동작 과정

***

`ThreadLocal`은 스레드마다 독립적인 변수를 저장하기 위해, 현재 스레드의 정보를 `Key` 로 저장하는 `Map` 구조를 사용한다.&#x20;

`ThreadLocalMap` 구조는 `Entry` 타입의 16크기 만큼의 배열로 구성 되어 있으며, 배열의 인덱스를 결정하기 위해 `ThreadLocal` 객체 고유의 `threadLocalHashCode` 를 사용하여 배열의 크기와 `AND` 연산을 한 수행하고, 이 결과 값을 통해  배열에 접근한다.

{% tabs %}
{% tab title="get()" %}
```java
public T get() {
    return get(Thread.currentThread());
}

private T get(Thread t) {
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T) e.value;
            return result;
        }
    }
    return setInitialValue(t);
}
```
{% endtab %}

{% tab title="set()" %}

{% endtab %}
{% endtabs %}

* `get()`: `Entry` 타입의 배열에 접근 하여 특정 인덱스에 엔트리가 갖고 있는 값을 조회한다.
* `set()`: `Entry` 타입의 배열에 접근 하여 사용자가 저장하고자 하는 값을 저장한다.



### SpringSecurity 를 통해 알아보기

***

{% hint style="info" %}
#### SecurityContextHolder - spring security docs

By default, `SecurityContextHolder` uses a `ThreadLocal` to store these details, which means that the `SecurityContext` is always available to methods in the same thread, even if the `SecurityContext` is not explicitly passed around as an argument to those methods. Using a `ThreadLocal` in this way is quite safe if you take care to clear the thread after the present principal’s request is processed. Spring Security’s [FilterChainProxy](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-filterchainproxy) ensures that the `SecurityContext` is always cleared.
{% endhint %}

`SpringSecurity` 공식 문서에 나와 있듯 `SecurityContextHolder` 는 기본적으로 아무런 설정을 하지 않는다면 `ThreadLocalSecurityContextHolderStrategy` 라는 객체를 통해 `ThreadLocal` 의 기능을 지원받고 있다고 한다.

아래 코드를 보면, 내부적으로 `contextHolder` 라는 `ThreadLocal` 을 생성하여 `get & set` 메서드를 추상화 하여 제공하고 있다.

{% tabs %}
{% tab title="ThreadLocalSecurityContextHolderStrategy" %}
```java
final class ThreadLocalSecurityContextHolderStrategy implements SecurityContextHolderStrategy {

    // ThreadLocal 객체 생성 -> Map의 Key
    private static final ThreadLocal<SecurityContext> contextHolder = new ThreadLocal<>();

    @Override
    public void clearContext() {
        contextHolder.remove(); // 메모리 누수 방지 및 스레드 풀 오염 방지
    }

    @Override
    public SecurityContext getContext() {
        SecurityContext ctx = contextHolder.get(); // 현재 스레드 맵에서 조회
        if (ctx == null) {
            ctx = createEmptyContext();
            contextHolder.set(ctx);
        }
        return ctx;
    }

    @Override
    public void setContext(SecurityContext context) {
        Assert.notNull(context, "Only non-null SecurityContext instances are permitted");
        contextHolder.set(context); // 현재 스레드 맵에 저장
    }
    
    // ... 생략
}
```
{% endtab %}
{% endtabs %}



#### 실전 사이드 프로젝트 코드를 탐구하기

<sub>아래 사이드 프로젝트의 예제는 코틀린으로 구성 되어있다.</sub>

> SecurityContextHolder 를 감싼 스레드 로컬의 get() 은 어떻게 쓰이고 있을까?
>
> * 유저 프로필을 조회하는 리졸버 컴포넌트이다.

<pre class="language-java"><code class="lang-java">@Component
class ProfileArgumentResolver(
    private val profileService: ProfileService,
) : HandlerMethodArgumentResolver {

    override fun supportsParameter(parameter: MethodParameter): Boolean {
        return parameter.hasParameterAnnotation(UserProfile::class.java)
    }

    override fun resolveArgument(
        parameter: MethodParameter,
        mavContainer: ModelAndViewContainer?,
        webRequest: NativeWebRequest,
        binderFactory: WebDataBinderFactory?
    ): Any? {
<strong>        val user = SecurityContextHolder.getContext().authentication.principal as User
</strong>        return profileService.getProfileByUserId(user.id).id
    }
}
</code></pre>



> SecurityContextHolder 를 감싼 스레드 로컬의 set() 은 어떻게 쓰이고 있을까?
>
> * jwt 인증 필터 구성 정보이다.

<pre class="language-java"><code class="lang-java">class JwtAuthenticationFilter(
    private val tokenService: TokenService,
    private val authService: AuthService,
    private val handlerExceptionResolver: HandlerExceptionResolver,
    private val permitAllMatchers: List&#x3C;RequestMatcher>,
    private val allowEndpoints: ApiAllowEndpoints,
) : OncePerRequestFilter() {

    override fun doFilterInternal(
        request: HttpServletRequest,
        response: HttpServletResponse,
        filterChain: FilterChain
    ) {
        try {

            if (isPermitAllRequest(request) || doesNotAllowEndpoint(request)) {
                filterChain.doFilter(request, response)
                return
            }

            val accessToken: String = resolveToken(request) ?: throw JwtAuthenticationException("액세스 토큰을 찾을 수 없습니다.")

            if (tokenService.validate(accessToken)) {
                val memberId: UUID = tokenService.getSubject(accessToken)
                val user: User = authService.getUserById(memberId)
                val authenticationToken: Authentication = PreAuthenticatedAuthenticationToken(user, null, ArrayList())

<strong>                SecurityContextHolder.getContext().authentication = authenticationToken
</strong>
                filterChain.doFilter(request, response)
                return
            }

            throw JwtAuthenticationException("잘못된 토큰 정보입니다.")

        } catch (exception: Exception) {
            SecurityContextHolder.clearContext()
            handlerExceptionResolver.resolveException(request, response, null, exception)
        }
    }
</code></pre>

이렇게 적용된 코드는 사용자의 요청을 처리하기 까지 계속 유저 정보를 다른 컴포넌트와 공유하여 사용할 수 있게된다.

만약, 스레드 로컬을 사용하지 못했다면 사용자 인증을 거친 후 프로필 조회, 게시글 조회 등 유저 인증 정보가 필요한 모든 컴포넌트 마다 우리는 인자로 유저 인증 정보를 계속 전달하여 사용해야한다.



### 정리하기

***

* **동시성 문제 해결**: `ThreadLocal`은 스레드마다 고유한 저장소를 생성하므로, 멀티 스레드 환경에서도 다른 스레드의 영향을 받지 않고 안전하게 데이터를 관리할 수 있다.
* **비즈니스 로직 단순화**: 객체 간 협력 관계에서 불필요한 파라미터를 획기적으로 줄여주며, 스레드가 작업을 마칠 때까지 데이터를 전역적으로 공유할 수 있다.
* **리소스 정리**: 스레드 풀을 사용하는 현대 웹 애플리케이션 환경에서는 스레드가 재사용된다. 따라서 데이터 오염을 막기 위해 사용 후에는 반드시 `remove()` 메서드로 저장된 값을 지워야 한다.



<details>

<summary>참고 자료</summary>

* [https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-securitycontextholder](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-securitycontextholder)
* [https://github.com/Storeaad/storead-api/blob/main/src/main/kotlin/com/storead/config/security/jwt/JwtAuthenticationFilter.kt](https://github.com/Storeaad/storead-api/blob/main/src/main/kotlin/com/storead/config/security/jwt/JwtAuthenticationFilter.kt)
* [https://github.com/Storeaad/storead-api/blob/main/src/main/kotlin/com/storead/profile/ProfileArgumentResolver.kt](https://github.com/Storeaad/storead-api/blob/main/src/main/kotlin/com/storead/profile/ProfileArgumentResolver.kt)

</details>



