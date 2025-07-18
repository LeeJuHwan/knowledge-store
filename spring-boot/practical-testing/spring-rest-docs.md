---
description: 테스트 코드를 통한 API 문서 자동화 도구
---

# Spring REST Docs

{% hint style="info" %}
#### REST Docs  vs Swagger

> **REST Docs**
>
> <mark style="color:orange;">**장점**</mark>
>
> * 테스트를 통과해야 문서가 만들어진다. -> 즉, 신뢰도가 높다
> * 프로덕션 코드에 비침투적이다. -> 문서 내부에서 프로덕션 코드를 실행시킬 수 없다
>
> <mark style="color:orange;">**단점**</mark>
>
> * 코드 양이 많다.
> * 설정이 까다롭다.



> **Swagger**
>
> <mark style="color:orange;">**장점**</mark>
>
> * 적용이 쉽다.
> * 문서에서 바로 API 호출을 수행 해볼 수 있다.
> * UI 가 상대적으로 깔끔하다.
>
> <mark style="color:orange;">**단점**</mark>
>
> * 프로덕션 코드에 침투적이다.
> * 테스트와 무관하기 때문에 실제로 동작하는 것과 코드의 신뢰도가 떨어질 수 있다.



#### 문서 선택 가이드

* 문서를 보며 바로바로 API 를 호출해야 하는 상황이다 -> _<mark style="color:green;">**Swagger**</mark>_
* 테스트가 더 중요하며 프로덕션 코드를 건드리지 않는 선에서 문서를 작성하고 싶다 -> _<mark style="color:green;">**REST Docs**</mark>_
{% endhint %}

#### REST Docs 를 만들기 위한 테스트 코드 작성 전 알아보기

***

REST Docs 를 사용하기 위해서 테스트 코드를 작성해야 한다.&#x20;

하지만, Controller 테스트를 구성 했던 코드에 Docs 를 위한 내용을 추가하여 공통으로 관리하게 되면 프로덕트가 커졌을 때 관리하기 힘들다는 단점이 명백하다.

이러한 이유로 기능 테스트 코드는 기능 테스트끼리 관리하고, 문서를 위한 코드는 따로 작성하여 서로 분리하는 것을 지향한다.

{% embed url="https://www.inflearn.com/community/questions/1584470/rest-docs-%EB%AC%B8%EC%84%9C%EC%9A%A9-%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%BD%94%EB%93%9C%EB%A5%BC-%EB%94%B0%EB%A1%9C-%EC%9E%91%EC%84%B1%ED%95%B4%EC%95%BC-%EB%90%98%EB%82%98%EC%9A%94" %}



컨트롤러 계층에서 단위 테스트를 돕는 도구 중 자주 선택되는 것은 MockMvc 와 RestAssured 이다.

MockMvc 는 springboot-starter-test 의존성에 포함되어 있어 별도의 의존성을 추가하지 않고 바로 사용할 수 있으며 SpringBootTest 어노테이션 없이 WebMvcTest 어노테이션으로 컨트롤러 계층만 테스트가 가능하다.

RestAssured 는 rest-assured:rest-assured 의존성을 별도로 추가해야 하며 컨트롤러 테스트를 하기 위해 SpringBootTest 어노테이션을 사용해야한다.



> 어떤 도구가 내게 더 어울릴까?

**RestAssured**&#x20;

* 가독성이 중요한가?
* 컨트롤러를 테스트 하는 과정에서 Bean 을 많이 사용하여 영향을 받는가?

\
**MockMvc**

* 전체 테스트를 자주 실행하여 빌드 속도에 영향을 받는가?
* 의존성을 별도로 추가하지 않고 바로 테스트가 가능해야 하는가?
* 컨트롤러 계층만 단위 테스트가 필요한가?&#x20;

{% embed url="https://tecoble.techcourse.co.kr/post/2020-08-19-rest-assured-vs-mock-mvc/" %}

{% embed url="https://techblog.woowahan.com/2597/" %}



### 설정

<details>

<summary>build.gradle 파일 보기</summary>

```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.4.2'
    id 'io.spring.dependency-management' version '1.1.7'
    id "org.asciidoctor.jvm.convert" version "3.3.2"
}

group = 'sample'
version = '0.0.1-SNAPSHOT'

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
    asciidoctorExt
}

repositories {
    mavenCentral()
}

dependencies {
    // Spring boot
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-validation'

    // test
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'

    // lombok
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    // h2
    runtimeOnly 'com.h2database:h2'

    // RestDocs
    asciidoctorExt 'org.springframework.restdocs:spring-restdocs-asciidoctor'
    testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'
}

tasks.named('test') {
    useJUnitPlatform()
}

ext { // 전역 변수
    snippetsDir = file('build/generated-snippets')
}

test {
    outputs.dir snippetsDir
}

asciidoctor {
    inputs.dir snippetsDir
    configurations 'asciidoctorExt'

    sources { // 특정 파일만 html로 만든다.
        include("**/index.adoc")
    }
    baseDirFollowsSourceFile() // 다른 adoc 파일을 include 할 때 경로를 baseDir로 맞춘다.
    dependsOn test
}

bootJar {
    dependsOn asciidoctor
    from("${asciidoctor.outputDir}") {
        into 'static/docs'
    }
}
```

</details>

<details>

<summary>REST Docs Test Support 추상 클래스 구성하기</summary>

```java
package sample.cafekiosk.spring.docs;


import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.documentationConfiguration;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.restdocs.RestDocumentationContextProvider;
import org.springframework.restdocs.RestDocumentationExtension;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;

@ExtendWith(RestDocumentationExtension.class)
public abstract class RestDocsSupport {

    protected MockMvc mockMvc;
    protected ObjectMapper objectMapper = new ObjectMapper();

    @BeforeEach
    void setUp(RestDocumentationContextProvider provider) {
//        this.mockMvc = MockMvcBuilders.webAppContextSetup(context)
        this.mockMvc = MockMvcBuilders.standaloneSetup(initController())
                .apply(documentationConfiguration(provider))
                .build();
    }

    protected abstract Object initController();
}
```

</details>

<details>

<summary>REST Docs Index 페이지 구성하기</summary>

* `cafekiosk/src/docs/asciidoc/index.adoc`

```asciidoc
ifndef::snippets[]
:snippets: ../../build/generated-snippets
endif::[]
= CafeKiosk REST API 문서
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toclevels: 2
:sectlinks:

[[Product-API]]
== Product API

[[product-create]]
=== 신규 상품 등록

==== HTTP Request
include::{snippets}/product-create/http-request.adoc[]
include::{snippets}/product-create/request-fields.adoc[]

==== HTTP Response
include::{snippets}/product-create/http-response.adoc[]
include::{snippets}/product-create/response-fields.adoc[]
```

</details>

<details>

<summary>공통 요청, 응답 정의하기</summary>

```asciidoc
==== Request Fields
|===
|Path|Type|Optional|Description

{{#fields}}

|{{#tableCellContent}}`+{{path}}+`{{/tableCellContent}}
|{{#tableCellContent}}`+{{type}}+`{{/tableCellContent}}
|{{#tableCellContent}}{{#optional}}true{{/optional}}{{/tableCellContent}}
|{{#tableCellContent}}{{description}}{{/tableCellContent}}

{{/fields}}

|===
```

```asciidoc
==== Response Fields
|===
|Path|Type|Optional|Description

{{#fields}}

|{{#tableCellContent}}`+{{path}}+`{{/tableCellContent}}
|{{#tableCellContent}}`+{{type}}+`{{/tableCellContent}}
|{{#tableCellContent}}{{#optional}}true{{/optional}}{{/tableCellContent}}
|{{#tableCellContent}}{{description}}{{/tableCellContent}}

{{/fields}}

|===
```

</details>



<figure><img src="../../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>



### 문서화를 위한 테스트 코드 작성

{% tabs %}
{% tab title="ProductControllerTest" %}
```java
package sample.cafekiosk.spring.api.controller.product;

import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import java.util.List;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.http.MediaType;
import sample.cafekiosk.spring.ControllerTestSupport;
import sample.cafekiosk.spring.api.controller.product.dto.request.ProductCreateRequest;
import sample.cafekiosk.spring.api.service.product.response.ProductResponse;
import sample.cafekiosk.spring.domain.product.ProductSellingStatus;
import sample.cafekiosk.spring.domain.product.ProductType;

class ProductControllerTest extends ControllerTestSupport {

    @DisplayName("신규 상품을 등록한다.")
    @Test
    void createProduct() throws Exception {
        // given
        ProductCreateRequest request = ProductCreateRequest.builder()
                .type(ProductType.HANDMADE)
                .sellingStatus(ProductSellingStatus.SELLING)
                .name("아메리카노")
                .price(4000)
                .build();

        // when // then
        mockMvc.perform(
                        post("/api/v1/products/new")
                                .content(objectMapper.writeValueAsString(request))
                                .contentType(MediaType.APPLICATION_JSON)
                )
                .andDo(print())
                .andExpect(status().isOk());
    }
    ...
}
```
{% endtab %}

{% tab title="ProductControllerDocsTest" %}
```java
package sample.cafekiosk.spring.docs.product;

import static org.mockito.BDDMockito.any;
import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.mock;
import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.document;
import static org.springframework.restdocs.operation.preprocess.Preprocessors.preprocessRequest;
import static org.springframework.restdocs.operation.preprocess.Preprocessors.preprocessResponse;
import static org.springframework.restdocs.operation.preprocess.Preprocessors.prettyPrint;
import static org.springframework.restdocs.payload.PayloadDocumentation.fieldWithPath;
import static org.springframework.restdocs.payload.PayloadDocumentation.requestFields;
import static org.springframework.restdocs.payload.PayloadDocumentation.responseFields;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.http.MediaType;
import org.springframework.restdocs.payload.JsonFieldType;
import sample.cafekiosk.spring.api.controller.product.ProductController;
import sample.cafekiosk.spring.api.controller.product.dto.request.ProductCreateRequest;
import sample.cafekiosk.spring.api.service.product.ProductService;
import sample.cafekiosk.spring.api.service.product.request.ProductCreateServiceRequest;
import sample.cafekiosk.spring.api.service.product.response.ProductResponse;
import sample.cafekiosk.spring.docs.RestDocsSupport;
import sample.cafekiosk.spring.domain.product.ProductSellingStatus;
import sample.cafekiosk.spring.domain.product.ProductType;

class ProductControllerDocsTest extends RestDocsSupport {

    private final ProductService productService = mock(ProductService.class);

    @Override
    protected Object initController() {
        return new ProductController(productService);
    }

    @DisplayName("신규 상품 등록 API")
    @Test
    void createProduct() throws Exception {
        ProductCreateRequest request = ProductCreateRequest.builder()
                .type(ProductType.HANDMADE)
                .sellingStatus(ProductSellingStatus.SELLING)
                .name("아메리카노")
                .price(4000)
                .build();

        given(productService.createProduct(any(ProductCreateServiceRequest.class)))
                .willReturn(ProductResponse.builder()
                        .id(1L)
                        .productNumber("001")
                        .type(ProductType.HANDMADE)
                        .sellingStatus(ProductSellingStatus.SELLING)
                        .name("아메리카노")
                        .price(4000)
                        .build()
                );

        mockMvc.perform(
                        post("/api/v1/products/new")
                                .content(objectMapper.writeValueAsString(request))
                                .contentType(MediaType.APPLICATION_JSON)
                )
                .andDo(print())
                .andExpect(status().isOk())
                .andDo(document(
                        "product-create",
                        preprocessRequest(prettyPrint()),
                        preprocessResponse(prettyPrint()),
                        requestFields(
                                fieldWithPath("type").type(JsonFieldType.STRING)
                                        .description("상품 타입"),
                                fieldWithPath("sellingStatus").type(JsonFieldType.STRING)
                                        .optional()
                                        .description("상품 판매 상태"),
                                fieldWithPath("name").type(JsonFieldType.STRING)
                                        .description("상품 이름"),
                                fieldWithPath("price").type(JsonFieldType.NUMBER)
                                        .description("상품 가격")
                        ),
                        responseFields(
                                fieldWithPath("code").type(JsonFieldType.NUMBER)
                                        .description("HTTP 상태 코드"),
                                fieldWithPath("status").type(JsonFieldType.STRING)
                                        .description("상태"),
                                fieldWithPath("message").type(JsonFieldType.STRING)
                                        .description("메시지"),
                                fieldWithPath("data").type(JsonFieldType.OBJECT)
                                        .description("응답 데이터"),
                                fieldWithPath("data.id").type(JsonFieldType.NUMBER)
                                        .description("상품 ID"),
                                fieldWithPath("data.productNumber").type(JsonFieldType.STRING)
                                        .description("상품 번호"),
                                fieldWithPath("data.type").type(JsonFieldType.STRING)
                                        .description("상품 타입"),
                                fieldWithPath("data.sellingStatus").type(JsonFieldType.STRING)
                                        .description("상품 판매 상태"),
                                fieldWithPath("data.name").type(JsonFieldType.STRING)
                                        .description("상품 이름"),
                                fieldWithPath("data.price").type(JsonFieldType.NUMBER)
                                        .description("상품 가격")
                        )
                ));
    }
}
```
{% endtab %}
{% endtabs %}

ProductControllerTest 의 createProduct 메서드를 API 문서화를 하는 과정을 진행하며 어떻게 적용할지 mockMvc 를 기준으로 정리하였다.

> **문서 내 API 항목 URL Slug 설정**

```java
document("product-create", ...
```

이렇게 작성하면 실제 렌더링된 화면에서 API 항목을 클릭 했을 때 docs/../product-create 로 설정된다.



> **요청 및 응답 JSON 가독성**

```java
preprocessRequest(prettyPrint()),
preprocessResponse(prettyPrint()),
```

이 코드를 덧붙이지 않으면 긴 JSON 형식이 한 줄로 표기되어 읽기 힘든 문서가 된다.



> **요청 값 정의**

{% tabs %}
{% tab title="POST - request body" %}
```java
requestFields(
    fieldWithPath("type").type(JsonFieldType.STRING)
            .description("상품 타입"),
    fieldWithPath("sellingStatus").type(JsonFieldType.STRING)
            .optional()
            .description("상품 판매 상태"),
    fieldWithPath("name").type(JsonFieldType.STRING)
            .description("상품 이름"),
    fieldWithPath("price").type(JsonFieldType.NUMBER)
            .description("상품 가격")
),
```
{% endtab %}

{% tab title="GET - parameter" %}
```java
queryParameters(
    parameterWithName("page").description("The page to retrieve"),
    parameterWithName("per_page").description("Entries per page")
)
```
{% endtab %}
{% endtabs %}



> **응답 값 정의**

```java
responseFields(
        fieldWithPath("code").type(JsonFieldType.NUMBER)
                .description("HTTP 상태 코드"),
        fieldWithPath("status").type(JsonFieldType.STRING)
                .description("상태"),
        fieldWithPath("message").type(JsonFieldType.STRING)
                .description("메시지"),
        fieldWithPath("data").type(JsonFieldType.OBJECT)
                .description("응답 데이터"),
        fieldWithPath("data.id").type(JsonFieldType.NUMBER)
                .description("상품 ID"),
        fieldWithPath("data.productNumber").type(JsonFieldType.STRING)
                .description("상품 번호"),
        ...
)
```

응답 값을 정의할 때 JSON 의 하위 계층을 표시하기 위해서 fieldWithPath 메서드 시그니처를 "a.b" 로 명시해서 하위 계층을 표시할 수 있다.

