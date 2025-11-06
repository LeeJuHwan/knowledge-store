# Merge Sort

{% hint style="info" %}
#### 병합 정렬이란?

존 폰 노이만이 1945년에 고안한 알고리즘으로, 분할 정복의 진수를 보여주는 알고리즘이다.

최선과 최악 모두 O(n log n)인 사실상 항상 동일한 시간복잡도를 나타내는 알고리즘이며, 대부분의 경우 퀵 정렬보다는 느리지만 일정한 실행 속도뿐만 아니라 무엇보다도 안전 정렬이라는 점에서 여전히 많이 쓰이고 있다.

\
병합 정렬은 분할 정복의 특징 답게 주어진 배열에서 두 부분으로 분할하고, 다시 네 부분으로 분할 등의 방식으로 각각 더이상 쪼갤 수 없을 때까지 계속 분할한 후, 분할이 끝나면 정렬하면서 정복해 나간다.
{% endhint %}



### 동작 방식

{% stepper %}
{% step %}
#### **주어진 배열을 반으로 분할한다**

<figure><img src="../../.gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
#### **분할 된 배열을 다시 네 등분으로 분할한다**

<figure><img src="../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
#### **최종적으로 더이상 분할할 수 없을 때 까지 분할을 완료한다**

<figure><img src="../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
#### **크기가 1인 부분 배열들을 서로 합치면서 재귀적으로 호출하며 정렬을 시작한다**

<figure><img src="../../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
<figure><img src="../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
<figure><img src="../../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
<figure><img src="../../.gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
<figure><img src="../../.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
<figure><img src="../../.gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
<figure><img src="../../.gitbook/assets/image (86).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
<figure><img src="../../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>
{% endstep %}
{% endstepper %}



### 구현

{% tabs %}
{% tab title="MergeSort.java" %}
```java
package sort;

public class MergeSort {

    public static void main(String[] args) {
        MergeSort mergeSort = new MergeSort();
        int[] A = {6, 4, 3, 7, 5, 1, 2};
        mergeSort.sort(A);
        mergeSort.printArray(A);
    }

    public void sort(int[] A) {
        sort(A, 0, A.length - 1);
    }

    private void sort(int[] A, int left, int right) {
        if (left < right) {
            int mid = (left + right) / 2;
            sort(A, left, mid); // 왼쪽 절반 정렬
            sort(A, mid + 1, right); // 오른쪽 절반 정렬
            merge(A, left, mid, right);
        }
    }

    public void printArray(int[] A) {
        for (int num : A) {
            System.out.print(num + " ");
        }
    }

    private void merge(int[] A, int left, int mid, int right) {
        int n1 = mid - left + 1; // 왼쪽 부분 배열 크기
        int n2 = right - mid;    // 오른쪽 부분 배열 크기

        int[] leftArray = new int[n1]; // 왼쪽 부분 배열
        int[] rightArray = new int[n2]; // 오른쪽 부분 배열

        // 왼쪽 배열에 n1 크기만큼 원본 데이터 복사
        for (int i = 0; i < n1; i++) {
            leftArray[i] = A[left + i];
        }

        // 오른쪽 배열에 n2 크기만큼 원본 데이터 복사
        for (int j = 0; j < n2; j++) {
            rightArray[j] = A[mid + 1 + j];
        }

        int i = 0; // 왼쪽 배열 인덱스
        int j = 0; // 오른쪽 배열 인덱스

        int k = left; // 병합 될 배열의 인덱스 시작점
        while (i < n1 && j < n2) {
            if (leftArray[i] <= rightArray[j]) {
                A[k] = leftArray[i];
                i++;
            } else {
                A[k] = rightArray[j];
                j++;
            }
            k++;
        }

        while (i < n1) {
            A[k] = leftArray[i];
            i++;
            k++;
        }

        while (j < n2) {
            A[k] = rightArray[j];
            j++;
            k++;
        }
    }

}
```
{% endtab %}
{% endtabs %}



#### 풀이

정렬할 때, 배열과 배열의 시작, 배열의 끝을 인자로 넘기며 메서드를 호출한다.

이 때, 배열의 크기가 1인 경우는 정렬할 필요가 없기 때문에 최소 2개인 경우에만 정렬이 가능하도록 조건 검사를 진행한다.

다음, 정렬 메서드를 호출하는데 재귀의 특성 덕분에 배열의 크기가 맨 처음엔 7개인 상태에서 중간 값을 구하여 호출하면 초기엔 `sort(array, 0, 3)` 을 호출하게 된다.

재귀로 인해 다시 정렬 메서드가 호출될 때엔 `(0 + 3) / 2 = 1` 로 중간 값을 구하여 `sort(array, 0, 1)` 을 다시 재귀로 호출하여 결국 더이상 분할이 불가능한 상태에서 병합을 진행한다.

병합 메서드 내부에서는 왼쪽 배열과 오른쪽 배열을 할당할 수 있는 메모리 공간을 확보하고, 전체 배열을 순회 하며 미리 확보해둔 메모리 공간에 전체 배열의 값을 할당한다.

* 이 때, 더이상 분할이 불가능한 상태에서 진행 되기 때문에 맨 처음으로 `leftArray = [6], rightArray = [4]` 가 할당되게 된다.

미리 메모리 공간을 확보해둔 배열의 크기만큼 반복하며, 서로 값을 비교하고 더 작은 수에 해당하는 배열인 경우 전체 배열에 값을 덮어쓴다.

그리고, 왼쪽 배열이든 오른쪽 배열이든 값이 더 컸던 경우는 작은 값을 할당했던 전체 배열의 바로 오른쪽에 위치한다.

이 과정을 계속 반복하여, 더이상 분할되지 않던 배열의 크기인 1개 부터, 2개, 4개 .. n 개까지 반복한다.



### 선택 가이드

#### 시간 복잡도

* **BestCase(O(n log n))**: 정렬할 때 항상 분할하여 정복하기 때문에 O(n log n) 의 반복횟수를 갖는다.
* **WorstCase(O(n log n))**: 최악의 상황 또한 항상 분할하여 정복하기 때문에 O(n log n) 의 반복횟수를 갖는다.
* **AverageCase(O(n log n)): 항상 O(n log n) 을 나타내는 정렬 알고리즘이다.**



#### 장점

* 시간복잡도가 최악과 최선 두 가지 경우 모두 일정하게 O(n log n) 시간복잡도를 갖는 알고리즘이다.
* 안전 정렬 알고리즘으로, 정렬된 배열에서 재정렬시 흐트러지지 않는다.



#### 단점

* 하위 배열을 보관할 메모리 공간을 추가로 확보해야 한다.
* 보조 배열에서 원본배열로 복사하는 과정은 매우 많은 시간을 소비하기 때문에 데이터가 많을경우 상대적으로 시간이 많이 소요된다.



{% hint style="success" %}
#### 언제 병합 정렬을 사용하는게 좋을까?

배열의 요소들이 대부분 정렬이 되어있는 경우 퀵 정렬보다 빠른 성능으로 알려져있기 때문에, 대부분 정렬되어 있는 경우에 적합하다.
{% endhint %}
