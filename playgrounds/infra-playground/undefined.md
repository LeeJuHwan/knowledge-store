---
description: HTTP Cache, gzip,Servlet,Thread에 대한 학습 테스트를 진행 합니다.
---

# 화면 성능 개선 전 학습 테스트

## 학습 테스트

* [깃허브 학습 테스트 자료](https://github.com/brainbackdoor/study?tab=readme-ov-file)
* [학습 테스트 풀이 자료](https://github.com/LeeJuHwan/infra-workshop/tree/main/learn-test/subwaymap)



### HTTP Cache

***

{% hint style="info" %}
_**Cache-Control 관리하기**_

* HTTP 응답 헤더에 Cache-Control 이 없으면 웹 브라우저가 휴리스틱 캐싱에 따른 암시적 캐싱을 한다.
* 의도하지 않은 캐싱을 막기 위해 모든 응답의 헤더에 Cache-Control: no-cache를 명시한다.
* 또한, 쿠키나 사용자 개인 정보 유출을 막기 위해 private도 추가한다.



**휴리스틱 캐싱에 대해 자세히 알아보려면** [**해당 블로그**](https://medium.com/a-day-of-a-programmer/cache-control-expires%EB%A5%BC-%EB%B9%A0%EB%9C%A8%EB%A6%AC%EB%A9%B4-%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80%EC%97%90%EC%84%9C-%EB%AC%B4%EC%8A%A8-%EC%9D%BC%EC%9D%B4-%EC%9D%BC%EC%96%B4%EB%82%A0%EA%B9%8C%EC%9A%94-756158e22f2d)**에 분석 사례 확인하기**
{% endhint %}

{% tabs %}
{% tab title="config/WebMvcConfig.java" %}
```java
@Configuration
@EnableWebMvc
public class WebMvcConfig implements WebMvcConfigurer {
    public static final String PREFIX_STATIC_RESOURCES = "/resources";


    private final ResourceVersion version;

    @Autowired
    public WebMvcConfig(ResourceVersion version) {
        this.version = version;
    }

    @Override
    public void addInterceptors(final InterceptorRegistry registry) {
        WebContentInterceptor interceptor = new WebContentInterceptor();
        interceptor.addCacheMapping(CacheControl.noCache().cachePrivate(), "/");
        registry.addInterceptor(interceptor);
    }
```
{% endtab %}
{% endtabs %}

모든 응답에 대해 처리 해야 했기 때문에 특정 컨트롤러에서 헤더에 정보를 추가하지 않고 설정 파일에서 캐시 값을 적용하도록 수정하였다.



{% hint style="info" %}
_**HTTP 압축 알고리즘 적용 with gzip**_



* 스프링 부트 설정을 통해 gzip과 같은 HTTP 압축 알고리즘을 적용 시킬 수 있다.
  * [Spring 공식문서](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto.webserver.enable-response-compression)
* 스프링에서 데이터를 압축할 때 기본 2KB부터 적용 되기 때문에 작은 크기의 파일은 압축하지 않는다. 작은 크기의 파일도 압축할 수 있도록 기본 설정을 10으로 변경한다.
{% endhint %}

{% tabs %}
{% tab title="application,yml" %}
```yaml
server:
  compression:
    enabled: true
    min-response-size: 10
```
{% endtab %}
{% endtabs %}

스프링부트 설정 파일에 해당 설정 값을 작성하게 되면 아래와 같이 헤더 정보에 Transfer-Encoding이 chunked로 표시되는 것을 확인할 수 있다.

<pre data-overflow="wrap"><code>[Cache-Control:"no-cache, private", vary:"accept-encoding", Content-Type:"text/html;charset=UTF-8", Content-Language:"ko-KR", <a data-footnote-ref href="#user-content-fn-1">Transfer-Encoding:"chunked"</a>, Date:"Mon, 27 Jan 2025 14:44:58 GMT"]
</code></pre>



{% hint style="info" %}
_**E-Tag 와 If-None-Match 를 활용한 HTTP 조건부 요청 캐싱**_

* 필터를 활용하여 "/etag" 경로도 ETag를 적용해보자
{% endhint %}

{% tabs %}
{% tab title="config/WebMvcConfig" %}
```java
@Bean
public FilterRegistrationBean filterRegistrationBean(){
    FilterRegistrationBean registration = new FilterRegistrationBean();
    Filter etagHeaderFilter = new ShallowEtagHeaderFilter();
    registration.setFilter(etagHeaderFilter);
    registration.addUrlPatterns("/etag");
    return registration;
}
```
{% endtab %}
{% endtabs %}

`etag`로 접근하는 요청에 E-Tag 값을 넣어 ETag가 무엇인지 테스트를 통해 확인한다.

{% code overflow="wrap" %}
```
[ETag:""005d25486ae7138209a9b6ceb6edf3b11"", Content-Type:"text/html;charset=UTF-8", Content-Language:"ko-KR", Date:"Mon, 27 Jan 2025 14:57:32 GMT", content-length:"293"]
```
{% endcode %}



{% hint style="info" %}
**캐시 무효화**

컨텐츠가 변경될 때 URL을 변경하여 응답을 장기간 캐시하게 만드는 기술이다.

정적 리소스를 max-age: 1년으로 설정한 후 JS나 CSS 리소스에 변경사항이 생기면 캐시가 제거되도록 할 때 사용하는 방식이다.
{% endhint %}

{% tabs %}
{% tab title="config/WebMbcConfig" %}
```java
@Bean
public FilterRegistrationBean filterRegistrationBean(){
    FilterRegistrationBean registration = new FilterRegistrationBean();
    Filter etagHeaderFilter = new ShallowEtagHeaderFilter();
    registration.setFilter(etagHeaderFilter);
    registration.addUrlPatterns("/etag", PREFIX_STATIC_RESOURCES + "/" + version.getVersion() + "/*");
    return registration;
}

@Override
public void addResourceHandlers(final ResourceHandlerRegistry registry) {
    registry.addResourceHandler(PREFIX_STATIC_RESOURCES + "/" + version.getVersion() + "/js/**")
            .addResourceLocations("classpath:/js/") // NOTE: 핸들러를 통해 들어온 정적 파일을 어디에 매칭할 것인지
            .setCacheControl(CacheControl.maxAge(Duration.ofDays(365)).cachePublic());
}
```
{% endtab %}
{% endtabs %}

정적 파일을 서빙할 때 캐싱이 적용 되어 있는 "resources/{version}/js/index.js" 에 캐싱이 적용되어 있고 max-age는 1년이라고 가정 해보자.

만약, 1년이 지나지 않은 시점에서 변경사항이 일어나서 배포해야 한다면 어떻게 할 것인가?

버저닝을 날짜로 사용한다면 배포 시점이 달라질 때 마다 캐시가 무효화 되고 새롭게 E-Tag를 저장하여 다시 캐싱을 적용할 수 있다.

리소스 요청 예시: `/resources/20250128000572/js/index.js`

{% code overflow="wrap" %}
```
[Vary:"Origin", "Access-Control-Request-Method", "Access-Control-Request-Headers", Last-Modified:"Mon, 27 Jan 2025 14:44:55 GMT", Cache-Control:"max-age=31536000, public", Accept-Ranges:"bytes", ETag:""0adf06cf637aff7c06810711225d7eec6"", Date:"Mon, 27 Jan 2025 15:04:42 GMT"], status code: 304 NOT_MODIFIED
```
{% endcode %}



### Servlet

***

{% hint style="info" %}
_**스레드 공유 영역인 서블릿의 인스턴스 변수 범위 확인하기**_

* 인스턴스 변수를 사용하는 서블릿에서 한 번의 요청을 보낼 때 마다 카운트를 증가 시키면 몇이 나올까?
{% endhint %}

{% tabs %}
{% tab title="ServletTest.java" %}
{% code overflow="wrap" %}
```java
@Test
void testSharedCounter() throws Exception {
    // 톰캣 서버 시작
    final var tomcatStarter = TestHttpUtils.createTomcatStarter();
    tomcatStarter.start();

    // shared-counter 페이지를 4번 호출한다.
    final var PATH = "/shared-counter";
    TestHttpUtils.send(PATH);
    TestHttpUtils.send(PATH);
    TestHttpUtils.send(PATH);
    final var response = TestHttpUtils.send(PATH);

    // 톰캣 서버 종료
    tomcatStarter.stop();

    assertThat(response.statusCode()).isEqualTo(200);

    // expected를 0이 아닌 올바른 값으로 바꿔보자.
    // 예상한 결과가 나왔는가? 왜 이런 결과가 나왔을까?
    // NOTE: Servlet의 인스턴스 변수를 공용으로 사용하고 있기 때문에 increase 할 때 프로세스가 실행될 때 최초 초기화 된 값에서 여러 스레드가 호출할 때 마다 공유하기 때문이다.
    assertThat(Integer.parseInt(response.body())).isEqualTo(4);
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

4번 호출 한 서블릿의 카운터 변수는 결과적으로 "4"일 것이다. 그 이유는 쓰레드 간 해당 자원을 공유하고 있기 때문이다



{% hint style="info" %}
_**서블릿의 로컬 영역에서 카운터 증가 확인하기**_

* 로컬 변수를 사용하는 서블릿에서 한 번의 요청을 보낼 때 마다 카운트를 증가 시키면 몇이 나올까?
{% endhint %}

{% tabs %}
{% tab title="ServletTest.java" %}
```java
void testLocalCounter() throws Exception {
    // 톰캣 서버 시작
    final var tomcatStarter = TestHttpUtils.createTomcatStarter();
    tomcatStarter.start();

    // local-counter 페이지를 3번 호출한다.
    final var PATH = "/local-counter";
    TestHttpUtils.send(PATH);
    TestHttpUtils.send(PATH);
    final var response = TestHttpUtils.send(PATH);

    // 톰캣 서버 종료
    tomcatStarter.stop();

    assertThat(response.statusCode()).isEqualTo(200);

    // expected를 0이 아닌 올바른 값으로 바꿔보자.
    // 예상한 결과가 나왔는가? 왜 이런 결과가 나왔을까?
    // Local scope에서 변수를 초기화 하고 increase 하기 때문에 해당 변수는 몇 번을 실행하든 호출이 일어났다면 1이 나오게된다.
    assertThat(Integer.parseInt(response.body())).isEqualTo(1);
}
```
{% endtab %}
{% endtabs %}

당연히 로컬 변수는 공유하지 않기 때문에 몇 번을 호출하더라도 1이 나올것이다.





[^1]: compression enable
