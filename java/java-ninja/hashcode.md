# hashCode가 뭘까요?

{% hint style="info" %}
해시코드란?

해시코드는 객체의 속성 값을 기준으로 객체를 식별하기 위한 정수 값이다.
{% endhint %}



> _**"왜****&#x20;****`equals()`****&#x20;****메서드를 재정의할 때****&#x20;****`hashCode()`****&#x20;****도 같이 재정의 되어야할까?"**_

해시코드를 사용하는 컬렉션인 `HashSet`, `HashMap` 에서 데이터를 저장하기 위한 인덱스를 계산할 때 해시코드를 사용한다.

그리고, 이 해시를 사용하는 컬렉션에서 내가 사용하고자 하는 객체를 키로 설정하는 경우 꼭 재정의 되어야한다.



#### Hash 컬렉션 구조 살펴보기

***

{% tabs %}
{% tab title="HashMap.java" %}
```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }


    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

```
{% endtab %}
{% endtabs %}

&#x20;`HashMap` 에서 새로운 값을 넣는 경우 생성자에서 어떠한 값을 입력하지 않으면 기본 16개의 요소를 담을 수 있는 배열을 선언한다.&#x20;

```java
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            ...
        else if (oldThr > 0) // initial capacity was placed in threshold
            ...
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }

```

위에 생성된 배열에서 어떤 인덱스에 값을 넣을지 결정하는데, 이 때 해시 값이 사용된다.

n 은 배열의 길이이고, i 는 배열의 길이 - 1 을 갖고 있으며 이 값을 이용하여 `hash` 와 `and` 연산을 한다.

* n = 16
* i = (16 - 1) = 15

**이제, AI 의 도움을 받아서 and 연산을 수행한 결과 값을 만들어내어 직접 확인해보자.**

* 32bit 기준
* `hash` 는 `int` 의 최대값 - 1 로 설정했다고 가정

```
  01111111111111111111111111111110  (hash = 2,147,483,646)
& 00000000000000000000000000001111  (n - 1 = 15)
------------------------------------
  00000000000000000000000000001110  (연산 결과 = 14)
```

그럼 새로운 값이 `HashMap` 의 내부 배열에 들어가는 초기 인덱스는 14번째이다.

그중, 만약 값이 같다면 `equals` 메서드를 호출하여 같은 객체인 경우 덮어쓰고, 다른 경우는 새로운 노드를 생성하여 `LinkedList` 로 구성하여 기존에 들어간 노드에 `next` 노드에 값을 할당한다.

```java
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
            
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
                ...
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }

            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
            
        if (++size > threshold)
            resize();

```

배열의 크기가 일정 숫자를 초과하면 새로운 배열(기본 값 \*\* 2)에 할당하고, 배열의 사이즈가 32가 넘으면서 `LinkedList` 의 사이즈가 8을 넘으면 트리노드 자료구조로 변환한다.

```java
static final float DEFAULT_LOAD_FACTOR = 0.75f;
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

final Node<K,V>[] resize() {
    newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    ...
    threshold = newThr;
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    ...
    if (++size > threshold)
        resize();
    }
```







&#x20; &#x20;
