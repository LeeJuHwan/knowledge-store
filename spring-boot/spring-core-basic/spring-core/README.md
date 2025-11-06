# 스프링 핵심 원리 이해

### Project settings

{% hint style="success" %}
#### Core

* [x] Java 21
* [x] Spring Boot 3.4.4
{% endhint %}

{% tabs %}
{% tab title="build.gradle" %}
```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.4.4'
    id 'io.spring.dependency-management' version '1.1.7'
}

group = 'hello'
version = '0.0.1-SNAPSHOT'

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

tasks.named('test') {
    useJUnitPlatform()
}
```
{% endtab %}
{% endtabs %}



{% hint style="info" %}
#### 비즈니스 요구사항과 설계

> **회원**
>
> * 회원가입 및 조회
> * 일반, VIP 등급
> * 데이터 저장 시 자체 DB 구축 또는 외부 시스템 연동 (미확정 요구사항)



> **주문과 할인 정책**
>
> * 상품 주문
> * 회원 등급에 따른 할인 정책
> * VIP 등급은 모든 상품에 대해 1,000 원 할인 (추후 변경 가능)
{% endhint %}

