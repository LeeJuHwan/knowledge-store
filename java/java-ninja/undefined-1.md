# 자바에서 정렬하는 방법

### Arrays.sort()

#### 원시타입을 정렬하는 Dual-Pivot QuickSort

***

{% hint style="info" %}
#### Dual-Pivot QuickSort 란?

QuickSort 에서 기준이 되는 Pivot 을 두 개로 나누어, 파티션을 총 3개로 분할하며 정렬하는 방식이다.

시간 복잡도 : O(n log n) 최악의 경우 O(n^2)
{% endhint %}

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

`pivot` 의 기준점을 배열의 시작과 끝으로 지정하고, 두 개의 피벗의 값과 비교하며 배열을 총 3개의 파티션으로 나누는 것이 특징이다.

* left side: `left pivot` 보다 값이 작은 파티션
* center: `left pivot` 보다 크거나 같고, `right pivot` 보다는 작은 파티션
* right side: `right pivot` 보다 큰 파티션



```java
    private int[] sort(int[] A, int start, int end) {
        if (start < end) {
            // 두 pivot을 기준으로 분할
            int[] pivots = partition(A, start, end);
            
            // 3개 영역을 재귀적으로 정렬
            sort(A, start, pivots[0] - 1);      // 첫 번째 영역: < pivot1
            sort(A, pivots[0] + 1, pivots[1] - 1); // 두 번째 영역: pivot1 ~ pivot2
            sort(A, pivots[1] + 1, end);        // 세 번째 영역: > pivot2
        }
        return A;
    }
```

위 기준에 맞춰 재귀를 호출할 때 기존 `QuickSort` 는 왼쪽과 오른쪽 파티션에 대해서만 재귀를 호출했지만, `Dual-Pivot QuickSort` 는 세개의 파티션에 대해 재귀를 호출하게 된다.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>



> #### 자바 7부터 왜 dual-pivot quick sort를 사용하게 되었을까?

우선, 일반적인 `QuickSort` 는 파티션이 한쪽으로 치우쳐지는 경우 `O(n^2)` 시간 복잡도의 알고리즘이다. 한 개의 `pivot` 을 기준으로 파티셔닝 하다보니 발생할 수 있는 가능성이 현저히 낮지가 않다.

그렇기 때문에 파티셔닝이 한쪽으로 치우쳐지는 것을 방지하기 위해 파티션을 3개로 분할하여 각 파티션 별 숫자가 균형잡히도록 유도하고, `O(n^2)` 상황을 최대한 방지하고자 하는 목적으로 자바7부터 `dual-pivot quick sort`가 쓰인것으로 보인다.



하지만 그럼에도 불구하고 최악은 `O(n^2)` 인 것은 변함 없지만, 기존 `one-pivot quick sort` 보다 평균적으로 비교하는 횟수가 줄어들기 때문에 속도의 이점도 분명 존재한다.



> #### 원시타입은 왜 dual-pivot quick sort를 사용하게 되었을까?

원시 타입에서 배열은 데이터가 메모리에 연속된 공간에 저장되기 때문에 메모리 접근이 효율적이며 연산 속도가 빠른 장점이 있다.

반면, 객체는 생성시 heap 에서 관리 되기 때문에 메모리 저장 위치가 분산되고, 객체간 비교를 위해 compareTo() 메서드를  호출해야하기 때문에 원시 타입에서 데이터를 바로 연산하는 것 보다 추가적인 오버헤드가 발생한다.



{% tabs %}
{% tab title="Arrays.java - DualPivotQuickSort" %}
```java
public static void sort(int[] a, int fromIndex, int toIndex) {
    rangeCheck(a.length, fromIndex, toIndex);
    DualPivotQuicksort.sort(a, 0, fromIndex, toIndex);
}
```
{% endtab %}

{% tab title="Arrays.java -TimSort" %}
```java
public static void sort(Object[] a) {
    if (LegacyMergeSort.userRequested)
        legacyMergeSort(a);
    else
        ComparableTimSort.sort(a, 0, a.length, null, 0, 0);
}
```
{% endtab %}
{% endtabs %}





#### 참조타입을 정렬하는 TimSort

***



### Collections.sort() <a href="#id-2.-20collections.sort-1" id="id-2.-20collections.sort-1"></a>



### Comparator를 이용한 커스텀 정렬



### Comparable 인터페이스 구현



### Stream API를 이용한 정렬



### **참고 자료**

{% embed url="https://www.youtube.com/watch?v=XYVbjQXkmiI&t=33s" %}

{% embed url="https://kyr-db.tistory.com/737" %}

{% embed url="https://cladren123.tistory.com/249" %}





