---
description: 제네릭이 무엇인지, 왜 제네릭을 사용하는지 그리고 자바는 제네릭을 어떻게 사용하고 있는지 알아보기
---

# 제네릭의 기본 이해

{% hint style="info" %}
#### 제네릭이란?

제네릭은 사전적 의미 자체적으로도 "일반적인" 이라는 뜻을 갖고 있으며, 자바에서는 클래스 내부에서 사용할 데이터 타입을 외부에서 직접 지정하는 방식이다.
{% endhint %}

제네릭은 사용할 타입을 미리 결정하지 않는 것이 중요한데, 클래스 내부에서 사용하는 타입을 클래스를 정의 하는 시점이 아닌 실제 사용하는 생성 시점에 타입이 결정되는 매커니즘이다.



#### 제네릭 명명 관례

타입 매개변수는 변수명 처럼 아무렇게 작성해도 상관이 없다. 하지만, 일반적으로 대문자를 사용하고 용도에 맞도록 영문자의 앞 첫글자를 사용하는 관례를 따른다.

<table><thead><tr><th valign="middle">keyword</th><th>description</th></tr></thead><tbody><tr><td valign="middle">E</td><td>Element</td></tr><tr><td valign="middle">K</td><td>Key</td></tr><tr><td valign="middle">N</td><td>Number</td></tr><tr><td valign="middle">T</td><td>Type</td></tr><tr><td valign="middle">V</td><td>Value</td></tr><tr><td valign="middle">S, U, V</td><td>2nd, 3rd, 4th type</td></tr></tbody></table>



### 제네릭은 왜 필요할까?

<sub>다양한 데이터 타입을 사용하여 데이터를 저장하고 조회할 수 있는 기능이 필요하다고 가정한다.</sub>

{% tabs %}
{% tab title="IntegerBox" %}
```java
public class IntegerBox {

    private Integer value;

    public void set(Integer value) {
        this.value = value;
    }

    public Integer get() {
        return value;
    }

}
```
{% endtab %}

{% tab title="StringBox" %}
```java
public class StringBox {

    private String value;

    public void set(String value) {
        this.value = value;
    }

    public String get() {
        return value;
    }

}
```
{% endtab %}

{% tab title="BoxMain" %}
```java
public class BoxMain {

    public static void main(String[] args) {
        IntegerBox integerBox = new IntegerBox();
        integerBox.set(10);
        Integer integer = integerBox.get();
        System.out.println("integer = " + integer);

        StringBox stringBox = new StringBox();
        stringBox.set("hello");
        String string = stringBox.get();
        System.out.println("string = " + string);
    }

}
```
{% endtab %}
{% endtabs %}

데이터 타입에 매칭되는 객체가 하나씩 생겨났고, 메인 로직에서 원하는 타입을 담고 싶을 때 마다 새로운 객체를 만들어내야 한다.

만약, 여기서 `Double`, `Boolean` 타입도 저장하고 조회할 수 있는 기능이 필요하다면 어떨까?



#### **제네릭을 사용하지 않고 `Object` 의 다형성 특성을 활용하여 해결 해보기**

위 예시에서 제네릭이라는 기법을 모르는 상태로 이 문제를 해결하고자 할 때 쉽게 사용할 수 있는 다형성의 특성을 살리면 된다.

모든 객체는 `Object` 의 하위 타입이라는 것을 알고 있으니, `Object` 로 저장하고 조회하면 된다.

{% tabs %}
{% tab title="ObjectBox" %}
```java
public class ObjectBox {

    private Object value;

    public void set(Object value) {
        this.value = value;
    }

    public Object get() {
        return value;
    }

}
```
{% endtab %}

{% tab title="BoxMain" %}
```java
public class BoxMain {

    public static void main(String[] args) {
        ObjectBox integerBox = new ObjectBox();
        integerBox.set(10);
        Integer integer = (Integer) integerBox.get();
        System.out.println("integer = " + integer);

        ObjectBox stringBox = new ObjectBox();
        stringBox.set("hello");
        String string = (String) stringBox.get();
        System.out.println("string = " + string);

    }
}
```
{% endtab %}
{% endtabs %}

`Object` 를 이용해서 문제를 해결해보니 데이터 타입에 맞는 새로운 객체를 만들지 않아도 되어 재사용이 가능한 이점이 있다.

하지만, `Object` 타입은 모든 객체의 상위 타입이기 때문에 숫자 타입을 조회하고 싶었으나 의도치 않게 문자 타입이 들어가거나 하는 등 의도하지 않은 타입으로 인해 캐스팅이 불가능하여 예외가 발생할 수 있다.

`Object` 로 해결하는 방법은 재사용을 통해 여러 객체가 생성 되는 것을 보완 하였지만 타입의 안정성을 지키지 못하였다.



#### 제네릭 적용 하여 문제 해결 하기

{% tabs %}
{% tab title="GenericBox" %}
```java
public class GenericBox<T> {

    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }

}
```
{% endtab %}

{% tab title="BoxMain" %}
```java
public class BoxMain3 {

    public static void main(String[] args) {
        GenericBox<Integer> integerBox = new GenericBox<>(); // 생성 시점에 T 타입이 Integer 로 결정
        integerBox.set(10);
        Integer integer = integerBox.get();
        System.out.println("integer = " + integer);

        GenericBox<String> stringBox = new GenericBox<>();
        stringBox.set("hello");
        String string = stringBox.get();
        System.out.println("string = " + string);
    }

}
```
{% endtab %}
{% endtabs %}

`Object` 로 풀었던 방식과 비슷하지만, `Generic` 클래스를 구성하게 되어 객체의 타입을 클래스 내부에서 지정하지 않고 외부에서 정의하게 되어 데이터를 저장하고, 조회할 때 같은 타입을 바라보게 되었다.

이로써 제네릭을 사용하지 않을 때의 문제점인 **코드 재사용성** 과 객체를 저장할 때와 반환할 때 타입이 동일한 **타입 안정성** 을 모두 해결할 수 있다.



#### 제네릭 사용시 주의할 점

{% stepper %}
{% step %}
제네릭 타입을 지정하지 않는 경우 Object 로 타입 추론된다.

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
제네릭 타입은 참조형 객체만 사용할 수 있다. -> 원시 타입은 사용할 수 없다.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>
{% endstep %}
{% endstepper %}



### 자바에서는 제네릭을 어떻게 사용하고 있을까?

자바에서는 이런 제네릭을 굉장히 많이 쓰고있는데, 그 중 개발을 하다보면 자주 마주치는 컬렉션인 `ArrayList` 를 살펴보자.

```java
ArrayList<String> stringArray = new ArrayList<>();
stringArray.add("hello");

for (String s : stringArray) {
    System.out.println("s = " + s);
}
```

대충 배열에 값을 담기 위해 `ArrayList` 객체를 생성하게 되는데 이 때 마찬가지로 제네릭을 사용하는 문법이 등장한다.

{% tabs %}
{% tab title="HashMap" %}
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
{
    ...
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
}
```
{% endtab %}

{% tab title="ArrayList" %}
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    ...
    
    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return {@code true} (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        modCount++;
        add(e, elementData, size);
        return true;
    }
}
```
{% endtab %}
{% endtabs %}

그 외에도 `HashMap` 도 동일하게 이런 제네릭을 사용하여 사용자가 자유롭게 다양한 타입을 이용할 수 있는 컬렉션을 구성한 것을 알 수 있다.



**참고 자료**

***

{% embed url="https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EC%A4%91%EA%B8%89-2/dashboard" %}

{% embed url="https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%A0%9C%EB%84%A4%EB%A6%ADGenerics-%EA%B0%9C%EB%85%90-%EB%AC%B8%EB%B2%95-%EC%A0%95%EB%B3%B5%ED%95%98%EA%B8%B0" %}
