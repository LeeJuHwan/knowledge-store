---
description: 비교연산자(==)와 equals 메서드의 차이
---

# 동일성과 동등성

> "동일성" 과 "동등성" 의 차이를 묻다.
>
> * 동일성: 물리적인 객체 참조 값이 완전히 같은가?
> * 동등성: 물리적인 객체 참조 값이 다르더라도 논리적 기준으로 서로 같은가?



실제 Object 클래스가 구현하고 있는 equals 메서드 내부를 살펴보면, 실제 == 을 사용하는 것을 볼 수 있다.

{% tabs %}
{% tab title="String.java" %}
```java
    /**
     * Compares this string to the specified object.  The result is {@code
     * true} if and only if the argument is not {@code null} and is a {@code
     * String} object that represents the same sequence of characters as this
     * object.
     *
     * <p>For finer-grained String comparison, refer to
     * {@link java.text.Collator}.
     *
     * @param  anObject
     *         The object to compare this {@code String} against
     *
     * @return  {@code true} if the given object represents a {@code String}
     *          equivalent to this string, {@code false} otherwise
     *
     * @see  #compareTo(String)
     * @see  #equalsIgnoreCase(String)
     */
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        return (anObject instanceof String aString)
                && (!COMPACT_STRINGS || this.coder == aString.coder)
                && StringLatin1.equals(value, aString.value);
    }

```
{% endtab %}
{% endtabs %}

자바 Object의 equals 는 동등성을 따질 때 논리적으로 어떤 값을 기준으로 할지 모르기 때문에 기본적으로 동일성을 제공한다.

\
이 논리적 기준을 정의하는 것은 개발자가 직접 정의하는 eqauls 메서드 오버라이딩이 필요하다.
