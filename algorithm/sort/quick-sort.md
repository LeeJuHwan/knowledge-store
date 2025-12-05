# Quick Sort

{% hint style="info" %}
**퀵정렬이란?**

영국의 컴퓨터과학자 [토니 호어](https://ko.wikipedia.org/wiki/%ED%86%A0%EB%8B%88_%ED%98%B8%EC%96%B4)가 1959년에 고안한 알고리즘으로, 피벗을 기준으로 좌우를 나누는 특징 때문에 파티션 교환 정렬이라고도 불린다.
{% endhint %}

퀵 정렬은 병합 정렬과 마찬가지로 분할 정복 알고리즘이며, 피벗이라는 개념을 통해 피벗보다 작은 왼쪽, 크면 오른쪽과 같은 방식으로 파티셔닝하면서 쪼개나간다.

여러가지 변형 버전이 존재하나, [니코 로무토](https://iq.opengenus.org/nico-lomuto/)가 구현한 가장 간단한 파티션 구성을 통해 어떻게 동작하는 살펴보자.

* 로무토 파티션은 항상 맨 오른쪽의 피벗을 선택하는 단순한 방식으로, 토니 호어가 고안한 최초의 퀵 정렬 알고리즘보다 훨씬 더 간결하고 이해하기 쉬운 장점이 있다.

<figure><img src="../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

로무토 파티션 방식의 수도코드를 기준으로 살펴보면, 최초 파티션을 한 번 분할하여 왼쪽과 오른쪽 파티션을 각각 재귀로 호출하는 방식이 보인다.

반복해서 언급하는 로무토 파티션은 토니 호어가 고안한 방식과는 차이가 있고, 또 훨씬 단순한 방식이다.

> 토니 호어의 퀵 정렬 알고리즘 동작 원리

{% embed url="https://upload.wikimedia.org/wikipedia/commons/9/9c/Quicksort-example.gif" %}

다시, 니코 로무토가 고안한 방식으로 돌아와서 이 코드를 자바로 작성해보자.

{% tabs %}
{% tab title="QuicSort.java" %}
```java
package sort;

public class QuickSort {

    public static void main(String[] args) {
        QuickSort quickSort = new QuickSort();
        int[] A = {3, 6, 8, 10, 1, 2, 1};
        int[] sortedA = quickSort.sort(A);
        quickSort.printArray(sortedA);
    }

    public int[] sort(int[] A) {
        return sort(A, 0, A.length - 1);
    }

    public void printArray(int[] A) {
        for (int num : A) {
            System.out.print(num + " ");
        }
    }

    private int[] sort(int[] A, int start, int end) {
        if (start < end) {
            int p = partition(A, start, end);
            sort(A, start, p - 1);
            sort(A, p + 1, end);
        }
        return A;
    }

    private int partition(int[] A, int low, int high) {
        int pivot = A[high];
        int left = low;

        for (int right = low; right < high; right++) {
            if (A[right] < pivot) {
                swap(A, left, right);
                left++;
            }
        }

        swap(A, left, high);
        return left;
    }

    private void swap(int[] A, int left, int right) {
        int temp = A[left];
        A[left] = A[right];
        A[right] = temp;
    }
}
```
{% endtab %}

{% tab title="도식화" %}
<figure><img src="../../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>
{% endtab %}
{% endtabs %}

#### 풀이

피벗은 항상 맨 오른쪽 값을 기준으로 하며, 이를 기준으로 2개의 포인터가 이동해서 오른쪽 포인터의 값이 피벗보다 작다면 서로 스왑하는 형태로 진행된다.

{% stepper %}
{% step %}
오른쪽 `right` 포인터가 이동하면서 피벗의 값이 오른쪽 값 보다 더 클 때, 왼쪽과 오른쪽을 스왑한다. (도식화 5번)
{% endstep %}

{% step %}
스왑 이후에는 왼쪽 `left` 포인터가 한칸 전진하고, 다시 오른쪽 값과 비교하며 모든 배열을 순회한다.
{% endstep %}

{% step %}
오른쪽 포인터가 끝에 도달하게 되면 `pivot` 을 기준으로 작은 파티션보다 앞에 위치하기 위해 `left` 와 `pivot` 의 위치를 스왑한 후 `left` 를 반환한다. (도식화 7번)
{% endstep %}
{% endstepper %}

이렇게 계속 분할하면서 정복을 진행하여 코드 기준으로 `low < high` 를 만족하지 않을 때 까지 즉, 서로 위치가 역전될 때까지 계속 재귀로 반복하며 정렬을 완료한다.

#### 요약

퀵 정렬은 평균적으로 `O(n log n)` 을 나타내지만, 최악의 경우 O(n^2) 이 된다.

{% hint style="warning" %}
**O(n^2) 은 어떤 상황일까?**

만약, 이미 정렬된 배열이 입력값으로 들어왔다고 가정해보자.

이 경우 피벗은 계속 오른쪽에 위치하게 되므로 파티셔닝이 전혀 이뤄지지 않는다.

배열의 크기가 동일한 n 번을 순회하여 결국 전체 비교를 하기 때문에 버블 정렬과 다를 바 없는 최악의 성능을 보이게 한다.
{% endhint %}

항상 일정하게 `O(n log n)` 성능을 보이는 병합 정렬과 달리, 퀵 정렬은 이처럼 입력된 배열에 따라 성능의 편차가 심한 편이다.

그렇기 때문에 단순한 방식이 아니라 알고리즘을 개선해 좀 더 최적화하여 사용하며, 실무에서는 퀵 정렬이 불안정 정렬에다 고르지 않은 성능 탓에 퀵 정렬보다 병합 정렬이 활발히 쓰이고 있다.

자바의 `Arrays.sort()` 또한 병합 정렬을 개선해서 만든 `Timsort` 를 기본적으로 채택하여 사용하고 있으며, 원시 자료형처럼 안정성이 중요하지 않은 경우에만 제한적으로 퀵 정렬(`Dual-Pivot Quick Sort`) 을 활용한다.

{% hint style="info" %}
**용어 설명**

1. Timsort: 파이썬 개발자가 만든 파이썬의 기본 정렬 알고리즘으로, 우수한 성능으로 인해 다른 언어에 적극 채택된 알고리즘
2. Dual-Pivot Quick Sort: 2개의 피벗을 이용하는 퀵 정렬 개선 버전
{% endhint %}

#### 참고 자료

***

{% embed url="https://en.wikipedia.org/wiki/Quicksort" %}

{% embed url="https://www.youtube.com/watch?v=7h1s2SojIRw" %}

{% embed url="https://www.youtube.com/watch?v=aY0yYfztKMY&t=117s" %}
