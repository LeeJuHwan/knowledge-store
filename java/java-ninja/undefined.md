---
description: 비교연산자(==)와 equals 메서드의 차이
---

# 동일성과 동등성

{% hint style="info" %}
동일성과 동등성 정의

* 동일성: 물리적인 객체 참조 값이 완전히 같은가?
* 동등성: 물리적인 객체 참조 값이 다르더라도 논리적 기준으로 서로 같은가?
{% endhint %}



### 동일성

{% tabs %}
{% tab title="EqualsExample.java" %}
```java
public class EqualsExample {

    private String dataKey;

    public EqualsExample(String dataKey) {
        this.dataKey = dataKey;
    }
}

```
{% endtab %}

{% tab title="EqualsMain.java" %}
```java
public class EqualsMain {

    public static void main(String[] args) {
        EqualsExample equalsExample1 = new EqualsExample("1");
        EqualsExample equalsExample2 = new EqualsExample("1");

        System.out.println(equalsExample1 == equalsExample2); // false
        System.out.println(equalsExample1.equals(equalsExample2)); // false
        
        System.out.println("equalsExample1.hashCode() = " + equalsExample1.hashCode()); // 1791741888
        System.out.println("equalsExample2.hashCode() = " + equalsExample2.hashCode()); // 883049899
        
        String a = "1";
        String b = "1";
        
        System.out.println(a == b); // true
        System.out.println("a.hashCode() = " + a.hashCode()); // 49
        System.out.println("b.hashCode() = " + b.hashCode()); // 49

    }
}

```
{% endtab %}
{% endtabs %}

위 `EqualsExample` 을 인스턴스화 하여 두 개의 객체간 비교를 해보면 동일성이 다른것을 알 수 있다.

동일성은 위의 정의 내용대로 `hashCode()` 값이 서로 같은지를 비교하는데, 두 객체의 해시코드 값을 출력해보면 두 객체의 해시코드 값이 다르다.

반대로, 해시코드의 값이 같다면 동일성의 결과로 `true` 가 반환된다.

**이렇게 객체의 참조 주소 값을 서로 비교하는 경우 "동일성" 비교라고 한다.**



### 동등성

{% tabs %}
{% tab title="EqualsExample2.java" %}
```java
public class EqualsExample2 {

    public String dataKey;

    public EqualsExample2(String dataKey) {
        this.dataKey = dataKey;
    }

    @Override
    public boolean equals(Object o) {
        if (o == null || getClass() != o.getClass()) return false;
        EqualsExample2 that = (EqualsExample2) o;
        return Objects.equals(dataKey, that.dataKey);
    }
}

```
{% endtab %}

{% tab title="EqualsMain.java" %}
```java
public class EqualsMain {

    public static void main(String[] args) {
        var equalsExample3 = new EqualsExample2("1");
        var equalsExample4 = new EqualsExample2("1");

        System.out.println(equalsExample3 == equalsExample4); // false
        System.out.println(equalsExample3.equals(equalsExample4)); // true

        System.out.println("equalsExample3.hashCode() = " + equalsExample3.hashCode()); // 1854731462
        System.out.println("equalsExample4.hashCode() = " + equalsExample4.hashCode()); // 317574433
    }
}
```
{% endtab %}
{% endtabs %}

`EqualsExample2` 클래스에서 `equals` 메서드를 오버라이딩하고, 그 외엔 동일성에서 다뤘던 내용과 같다.

그리고 동일한 코드로 객체만 변경 하여 검증을 해보면 동일성은 `false`, 동등성은 `true` 가 반환되었다.

앞서 `equals` 메서드를 우리가 선언한 "논리적 기준(dataKey)이 같다면 서로 같다" 라고 재정의 하였기 때문에 두 객체의 해시코드 값이 서로 다르더라도 동등성은 `true` 가 나오게된다.



#### Object 에서 구현한 equals 메서드 내용

<figure><img src="../../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

실제 `Object`에서도 `equals` 메서드를 구현해두기로 동일성 비교를 사용하는 것을 알 수 있다.

그렇다면, `equals` 메서드를 오버라이딩 하지 않으면 우린 평소에 `==` 비교로 값을 반환 받는다고 생각하면 된다.

이로인해, 객체간 비교를 할 때 서로가 갖고 있는 논리적 기준이 같은지 확인하기 위해서는 equals 메서드를 오버라이딩 해야한다.



#### 해시코드 도 재정의 해야한다.

위 예시에서는 `equals` 메서드를 통해서 객체간의 논리적 기준이 같을 때 서로 같은 객체로 확인하는 방식에 대해 알아보았는데, 사실 IDE 에서 이 메서드를 직접 작성하지 않고 편하게 클릭 한 번에 모두 재정의 되도록 지원한다.

이 때, `equals()` & `hashCode()` 를 같이 재정의하는데 이 예시에서 `hashMap` 을 다룰 일이 없었기 때문에 해시코드가 중요하게 사용되지 않아 재정의 하지 않았을 뿐 사실 두 메서드 모두 재정의 해야한다.



그 이유는 [hashcode.md](hashcode.md "mention") 페이지에서 다루었으니, 해당 페이지에서 해시코드란 무엇이며 왜 `equals()` 와 `hashCode()` 가 재정의 되어야하는지에 대해 설명한다.
