---
description: 간단한 토이 프로젝트를 레이어드 아키텍처로 구성하여 테스트 코드 작성 해보기
---

# 스프링 레이어드 아키텍처 테스트하기



<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>



### Project settings

{% hint style="success" %}
#### Core

* [x] Java 21
* [x] Spring Boot 3.4.2



#### Dependency

* [x] spring boot web
* [x] spring data JPA
* [x] H2 Database
* [x] Lombok
{% endhint %}



#### Resources Configuration

***

{% tabs %}
{% tab title="application.yml" %}
```yaml
spring:
  profiles:
    default: local

  datasource:
    url: jdbc:h2:mem:~/cafeKioskApplication
    driver-class-name: org.h2.Driver
    username: sa
    password:

  jpa:
    hibernate:
      ddl-auto: none

---

spring:
  config:
    activate:
      on-profile: local

  jpa:
    hibernate:
      ddl-auto: create
    show-sql: true
    properties:
      hibernate:
        format_sql: true
    defer-datasource-initialization: true # (spring boot 2.5 ~) Hibernate 초기화 이후 data.sql 실행

  h2:
    console:
      enabled: true

---

spring:
  config:
    activate:
      on-profile: test


  jpa:
    hibernate:
      ddl-auto: create
    show-sql: true
    properties:
      hibernate:
        format_sql: true

  sql:
    init:
      mode: never
```
{% endtab %}

{% tab title="data.sql" %}
```
insert into product(product_number, type, selling_status, name, price)
values ('001', 'HANDMADE', 'SELLING', '아메리카노', 4000),
       ('002', 'HANDMADE', 'HOLD', '카페라떼', 4500),
       ('003', 'BAKERY', 'STOP_SELLING', '크루아상', 3500);
```
{% endtab %}
{% endtabs %}



#### Structures

***

```
```

