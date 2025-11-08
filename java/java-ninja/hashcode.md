# hashCode가 뭘까요?

{% hint style="info" %}
#### **해시란?**

해시 함수라는 알고리즘을 사용해 임의의 데이터를 고정된 길이의 값으로 반환 하는 것이다.

이러한 방식은 데이터를 빠르게 저장하고 검색할 수 있는 해시 테이블이라는 자료구조에서 핵심적인 역할을 한다.

해시 테이블은 해시 함수로 반환 된 키를 사용하여 값을 저장하고 조회하기 때문에 충돌이 없는 이상적인 경우 평균적으로 O(1)의 매우 빠른 시간 복잡도를 가진다.
{% endhint %}

{% hint style="info" %}
#### **해시코드란?**

해시코드는 객체의 속성 값을 기준으로 객체를 식별하기 위한 정수 값이다.
{% endhint %}

## 자바는 왜 해시를 사용할까?

기본적인 배열 자료구조는 데이터를 인덱스 기반으로 조회하는 데 O(1) 이지만, 데이터 전체를 탐색해야 할 때 O(N) 이 소요된다.

해시를 이용한 자료구조를 사용하면 Key: Value 에서 키의 접근은 인덱스 접근으로 배열에서 O(1) 시간복잡도를 그대로 나타낼 수 있게 되어 데이터의 조회, 저장, 삭제 작업이 O(1) 로 효율적이다.

즉, 자바에서는 대량의 데이터를 효과적으로 다루기 위해 해시를 사용한다.



## 왜 `equals()` 메서드를 재정의할 때`hashCode()` 도 같이 재정의 되어야할까?

해시코드를 사용하는 컬렉션인 `HashSet`, `HashMap` 에서 데이터를 저장하기 위한 인덱스를 계산할 때 해시코드를 사용한다.

그리고, 이 해시를 사용하는 컬렉션에서 내가 사용하고자 하는 객체를 키로 설정하는 경우 꼭 재정의 되어야한다.

### `equals` 만 재정의하고 `hashCode` 를 재정의 하지 않는다면?

```java
A a1 = A("a"); // hashCode = 1234
A a2 = A("a"); // hahsCode = 1235
```

위 두 객체는 논리적으로 같지만, 해시코드 값이 다르기 때문에 시스템 내에선 서로 다른 객체로 인식하게 된다.

여기서 `HashMap` 구조에 키 값을 A 객체로 사용하게 되는 경우 문제가 발생한다.

a1 을 HashMap 의 키로 저장하여 `hashMap.put(a1, "a")` 이런 코드를 작성했다고 가정하자.

* `hashCode` 에 의해 반환 된 해시 값과 `putVal` 내부 구현 내용에 따라 `and` 연산을 거쳐 배열에 1번 인덱스에 저장된다고 가정한다.

HashMap 에서 서로 논리적으로 같은 a2 객체를 기준으로 조회한다면 어떨까?

* `hashCode` 값이 1235 이기 때문에 내부 구현 내용에 따라 2번 인덱스가 조회 됐다고 가정한다.

```
hashMap = [null, a1, null, null ...]
```

위 코드처럼 2번에는 값이 존재하지 않기 때문에 null 을 반환하게 된다.

즉, 해시 컬렉션이 **논리적으로 같은 객체"를 "물리적으로 같은 인덱스**에서 찾도록 보장하기 위해, `equals()`의 판단 기준이 같다면 `hashCode()`의 반환 값도 반드시 같도록 재정의해야 한다.

## 자바에서 해시 충돌이 발생하면 어떻게 해결하는가?

### 해시 충돌은 왜 발생하는가?

`hashCode` 의 연산 결과 값은 해시 인덱스의 분포도를 최대한 분산 시키는데 있지만, 결국 int 자료형 크기 내에서만 유한한 것이다.

결국 완전하게 고유하지 않을 수 있다는 여지가 존재한다.

그렇기 때문에 자바에서는 동일한 해시 인덱스가 반환 되었을 때 충돌이 발생할 것이라는 걸 인지하고, 그 충돌을 처리하는 동작 과정을 많이 적용하였다.

### `HashMap` 에 값을 할당하는 과정

`HashMap` 을 생성할 때 아무런 인자값을 넣지 않으면 기본 16개를 담을 수 있는 배열을 생성한다.

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

`hash()` 함수에서 위 연산을 통해 `hashCode` 값을 넓게 분포 시킨다.

* `hashCode` 반환 값을 16비트 만큼 오른쪽으로 쉬프트
* `hashCode` 값과 오른쪽으로 쉬프트 한 값에 대한 XOR 연산

```
해쉬코드:       10101010 11110000 00001111 11001100
오른쪽 쉬프트:  00000000 00000000 10101010 11110000
XOR:            10101010 11110000 10100101 00111100
```

해시로 인해 배열의 인덱스 한 곳에 몰리는 현상을 방지하기 위해 해시맵을 저장할 배열의 크기인 `(n - 1)` 과 `hash` 알고리즘의 결과 값을 `and` 연산하여 배열의 저장될 인덱스를 결정한다.

### 해시 충돌을 해결하는 과정

{% stepper %}
{% step %}
만약, 배열의 인덱스에 값이 존재한다면 `equals` 메서드를 통해 같은 객체일 경우 덮어쓰고, 그렇지 않은 경우 새로운 노드 객체를 만들어 `LinkedList` 를 구성하여 리스트의 마지막에 연결한다.
{% endstep %}

{% step %}
한 버킷에 7개의 노드가 있는 상태에서 8번째 노드가 추가될 때 `HashMap`은 전체 배열의 크기를 확인하여 만약, 배열 크기가 64보다 작으면 트리 변환 대신 배열 전체를 2배로 `resize`하여 충돌을 줄인다.
{% endstep %}

{% step %}
배열 크기가 64 이상이면서 `LinkedList` 의 길이가 8이상이면, 해당 버킷을 `Red-Black Tree` 구조로 변환한다.

`LinkedList`의 길이가 계속 길어지면 검색 성능이 O(N)에 가까워져 해시 테이블의 장점(O(1))이 사라지기 때문에 `Red-Black Tree`로 변환하여 성능을 O(log N)으로 향상시킨다.
{% endstep %}
{% endstepper %}

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

`HashMap` 에서 새로운 값을 넣는 경우 생성자에서 어떠한 값을 입력하지 않으면 기본 16개의 요소를 담을 수 있는 배열을 선언한다

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

배열의 크기가 일정 숫자를 초과하면 새로운 배열(기본 값 \*\* 2)에 할당한다.

```java
static final int MAXIMUM_CAPACITY = 1 << 30;
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    ...
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    ...
}
```

배열의 크기가 64가 넘으면서 `LinkedList` 의 사이즈가 8을 넘으면 트리노드 자료구조로 변환한다.

만약, 배열의 크기가 64가 넘지 않으면 배열의 확장을 먼저 시도한다.

```java
/**
 * Replaces all linked nodes in bin at index for given hash unless
 * table is too small, in which case resizes instead.
 */
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;
        do {
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}

```



**참고 자료**

***

{% embed url="https://www.geeksforgeeks.org/dsa/what-is-hashing/" %}

{% embed url="https://www.geeksforgeeks.org/java/hashing-in-java/" %}

* HashMap.java  소스코드
