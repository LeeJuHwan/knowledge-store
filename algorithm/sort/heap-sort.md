# Heap Sort

{% hint style="info" %}
**힙 정렬이란?**

정렬 명칭에도 나와있듯 힙 자료구조를 이용한 정렬 방법이며 선택 정렬의 최적화 버전이다.

* Heap: 최솟값 또는 최댓값을 빠르게 찾아내기 위해 완전이진트리 형태로 만들어진 자료구조

<img src="../../.gitbook/assets/image (5) (2).png" alt="" data-size="original">

최소힙으로 오름차순으로 구현하게 되는 경우, 형제노드간 우선순위를 고려할 수 없어 최대힙 자료구조로 재구성하고 최대힙의 루트노드를 뒤로 배치시켜 다시 최대힙으로 재구성하는 과정을 반복하며 정렬 하는 방식이다.
{% endhint %}

### 동작 방식

{% stepper %}
{% step %}
**힙 자료구조를 이용하기 위해 최초 입력 받은 배열을 힙에 저장한다**

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
**힙에 저장된 정렬되지 않은 상태를 최대힙으로 만들기 위해 내림차순으로 재구성한다**

재구성 과정은 가장 작은 서브트리부터 최대 힙을 만족하도록 순차적으로 진행하는 것으로, 힙에서 상위 노드에 최대값이 오도록 만드는 과정이다.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
**루트노드를 배열의 맨 뒷쪽으로 배치한다**

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
**뒤로 배치된 원소를 제외하고 최대힙으로 재구성한다**

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
**3번과 4번 과정을 반복한다**

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
{% endstep %}
{% endstepper %}

### 구현

{% tabs %}
{% tab title="HeapSort.java" %}
```java
public class HeapSort {

    public static void main(String[] args) {
        HeapSort heapSort = new HeapSort();
        int[] A = {9, 4, 3, 8, 10, 2, 5};
        heapSort.sort(A);
        heapSort.printArray(A);
    }

    public void printArray(int[] A) {
        for (int num : A) {
            System.out.print(num + " ");
        }
    }

    public void sort(int[] A) {
        int n = A.length;

        // 힙 구성
        for (int i = n / 2 - 1; i >= 0; i--) {
            makeHeap(A, n, i);
        }

        // 힙에서 요소 하나씩 추출
        for (int i = n - 1; i > 0; i--) {
            // 현재 루트(최대값)를 끝으로 이동
            int temp = A[0];
            A[0] = A[i];
            A[i] = temp;

            // 감소된 힙에 대해 힙 속성 재구성
            makeHeap(A, i, 0);
        }
    }

    private void makeHeap(int[] array, int n, int i) {
        int root = i; // 루트
        int leftChild = 2 * i + 1; // 왼쪽 자식
        int rightChild = 2 * i + 2; // 오른쪽 자식

        // 왼쪽 자식이 루트보다 크면
        if (leftChild < n && array[leftChild] > array[root]) {
            root = leftChild;
        }

        // 오른쪽 자식이 현재 가장 큰 값보다 크면
        if (rightChild < n && array[rightChild] > array[root]) {
            root = rightChild;
        }

        // 가장 큰 값이 루트가 아니면 교환
        if (root != i) {
            int swap = array[i];
            array[i] = array[root];
            array[root] = swap;

            // 재귀적으로 힙 속성 재구성
            makeHeap(array, n, root);
        }
    }
```
{% endtab %}
{% endtabs %}

맨 처음부터 힙을 생성할 때, 배열에 있는 원소들을 내림차순으로 정렬하는 최대 힙으로 구성한다.

다음, 최대힙의 특성을 활용하여 루트노드에 있는 값을 배열의 끝으로 이동한뒤, 힙을 최대 힙으로 재구성한다.

이 과정을 반복하여 배열에 있는 요소가 순차적으로 정렬이된다.

### 선택 가이드

#### 시간 복잡도

* 힙에서 요소를 추출하고 재구성하는 과정을 n-1번 반복하므로 O(n log n)이 걸린다.
* 데이터의 초기 상태와 관계없이 항상 O(n log n)의 성능을 보장한다.

#### 장점

* 최악의 경우에도 O(n log n) 을 보장하며, Quick Sort 의 최악의 경우인 O(n<sup>2</sup>) 을 피하기에 적합하다.

#### 단점

* Heap Sort 의 시간 복잡도는 O(n log n) 으로 최악인 경우도 마찬가지 이지만, 평균적인 Quick Sort 에 비해 속도가 느리다.
  * 이는 캐시 메모리로 인한 지역 참조성 때문인데, 힙 정렬 과정에서 루트 노드를 배열에 맨 뒤로 배치하고 최대 힙으로 재구성하는 과정은 캐시가 예측하기 어렵기에 캐시 미스가 발생한다.
  * 반대로, Quick Sort 는 연속적인 메모리 접근 덕분에 캐시가 다음 행동을 예측하여 캐시 히트를 자주 일으켜 물리 메모리에 있는 데이터에 접근하지 않기 때문에 속도가 더 빠르다.
* 재구성 과정에서 같은 값을 가진 요소들의 상대적인 순서가 바뀔 수 있는 불안정 정렬이다.

**참고 자료**

{% embed url="https://www.geeksforgeeks.org/dsa/heap-sort/" %}

{% embed url="https://st-lab.tistory.com/225" %}

{% embed url="https://medium.com/pocs/locality%EC%9D%98-%EA%B4%80%EC%A0%90%EC%97%90%EC%84%9C-quick-sort%EA%B0%80-merge-sort%EB%B3%B4%EB%8B%A4-%EB%B9%A0%EB%A5%B8-%EC%9D%B4%EC%9C%A0-824798181693" %}

{% embed url="https://www.baeldung.com/cs/quicksort-vs-heapsort" %}
