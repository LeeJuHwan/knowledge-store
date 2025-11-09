# Counting Sort

{% hint style="info" %}
**카운팅 정렬이란?**

보통의 정렬 알고리즘은 배열의 값을 서로 비교하는 알고리즘인데, 카운팅 정렬은 비교 알고리즘이 아닌 누적합을 구성하여 정렬하는 알고리즘으로 배열의 크기가 작을 때 특히 유용하다.
{% endhint %}

### 동작 원리

***

{% stepper %}
{% step %}
**주어진 배열에서 가장 큰 값을 찾는다**

<figure><img src="../../.gitbook/assets/image (2) (2).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
**누적합을 저장하기 위해 배열에서 가장 큰 값에 1을 더한 크기의 배열을 생성한다**

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
**배열의 요소를 카운트하여 누적합 배열에 저장한다**

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
**배열의 왼쪽 요소부터 값을 더하며 증가하는 누적합을 구성한다**

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
**정렬되지 않은 원본 배열의 요소를 꺼내어 자신의 값에 해당하는 누적합 배열의 인덱스를 조회하고, 누적합 배열 인덱스에 해당하는 값에서 -1 을 한 좌표에 정렬 결과를 담을 배열에 저장한다**

<figure><img src="../../.gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
**5번 과정을 반복하여 배열의 값을 모두 정렬 시킨다**

<figure><img src="../../.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>
{% endstep %}
{% endstepper %}

### 구현

{% tabs %}
{% tab title="CountingSort.java" %}
```jsx
package sort;

public class CountingSort {

    public static void main(String[] args) {
        CountingSort countingSort = new CountingSort();
        int[] A = {2, 5, 3, 0, 2, 3, 0, 3};
        int[] sorted = countingSort.sort(A);
        countingSort.printArray(sorted);
    }

    public void printArray(int[] A) {
        for (int num : A) {
            System.out.print(num + " ");
        }
    }

    public int[] sort(int[] A) {
        int n = A.length;

        // 최대값 찾기
        int k = getMax(A);
        int[] count = new int[k + 1];
        int[] output = new int[n];

        // 각 요소의 빈도 계산
        for (int i = 0; i < n; i++) {
            count[A[i]]++;
        }

        // 누적 합 계산
        for (int i = 1; i <= k; i++) {
            count[i] += count[i - 1];
        }

        // 출력 배열에 정렬된 요소 배치
        for (int i = n - 1; i >= 0; i--) {
            output[count[A[i]] - 1] = A[i];
            count[A[i]]--;
        }

        return output;
    }

    private int getMax(int[] A) {
        int max = A[0];
        for (int num : A) {
            if (num > max) {
                max = num;
            }
        }
        return max;
    }

}
```
{% endtab %}
{% endtabs %}

#### 풀이

주어진 배열에서 최대 값을 찾는다.

최대 값에서 1을 더하여, 누적합을 저장할 배열의 크기를 지정하여 할당한다.

반환할 결과 배열을 생성해도 되고, 원본 배열에 복사해도 되기 때문에 구현 방법에 맡기지만, 위 구현에서는 결과를 반환하기 위해 결과 배열도 생성한다.

누적합을 저장할 배열에 각 요소마다 몇 번씩 등장하는지 갯 수를 센 뒤, 왼쪽의 값을 쭉 더해나가는 누적합을 완성시킨다.

배열의 뒤에서부터 값을 하나씩 순회하며, 조회된 값을 누적합 배열의 인덱스로 접근하고 그 접근 결과를 -1 하여 반환할 배열에 배치한다.

위 과정을 반복하여 뒤에서부터 0번째 인덱스까지 모두 순회하면 정렬이 완료된다.

### 선택 가이드

#### 시간 복잡도

* 모든 데이터에서 최대 값 검색 N 회, 요소 갯 수 카운트 N 회, 누적합 N -1 회, 결과 배열로 이동 N 회의 흐름을 따르며 O(n) 알고리즘이다.

#### 장점

* 요소가 등장한 횟수를 세기 때문에 안정 정렬 알고리즘이다.
* 값을 비교하는 알고리즘이 아닌 계산 하는 알고리즘은 비교 알고리즘보다 훨씬 빠른 속도를 자랑한다.

#### 단점

* 배열의 요소 중 큰 값이 클수록 배열의 크기가 커지기 때문에 비효율적이다.
* 배열 요소를 중간에 저장하기 위해 추가적인 메모리 공간을 필요로한다.

**참고 자료**

***

{% embed url="https://www.geeksforgeeks.org/dsa/counting-sort/" %}
