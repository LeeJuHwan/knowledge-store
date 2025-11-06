# Insertion Sort

{% hint style="info" %}
#### 삽입 정렬이란?

수열의 왼쪽부터 순서대로 정렬하는 알고리즘이며, 진행에 따라 왼쪽에는 숫자가 점차 정렬되고, 오른쪽에는 아직 확인되지 않은 숫자가 남는다.

오른쪽의 미탐색 영역에서 숫자를 하나씩 꺼내어 정렬이 끝난 영역의 적절한 위치에 삽입해 나가며 정렬을 완성하는 알고리즘이다.



💡 카드 게임을 할 때 손에 카드를 쥔 채 정렬하는 방법을 모방했다.
{% endhint %}



### 동작 방식

{% stepper %}
{% step %}
<figure><img src="../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

삽입 정렬은 첫 번째 요소가 간단하게 정렬되기 때문에 두 번째 요소(인덱스 = 1)에서 시작한다.
{% endstep %}

{% step %}
<figure><img src="../../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>

현재 요소(키)를 선택하고 왼쪽의 정렬된 부분과 비교한다.
{% endstep %}

{% step %}
<figure><img src="../../.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>

키보다 큰 요소는 오른쪽으로 한 위치 이동하여 키를 삽입할 공간을 만든다.
{% endstep %}
{% endstepper %}

### 구현

{% tabs %}
{% tab title="InsertionSort.java" %}
```java
public class InsertionSort {

    public static void main(String[] args) {
        InsertionSort insertionSort = new InsertionSort();
        int[] A = {5, 3, 4, 7, 2, 8, 6, 9, 1};
        insertionSort.sort(A);
        insertionSort.printArray(A);

    }

    public void sort(int[] A) {
        for (int i = 1; i < A.length; i++) {
            int key = A[i];
            int target = i - 1;

            while (target >= 0 && A[target] > key) {
                A[target + 1] = A[target];
                target--;
            }

            A[target + 1] = key;
        }
    }

    public void printArray(int[] A) {
        for (int num : A) {
            System.out.print(num + " ");
        }
    }

}
```
{% endtab %}
{% endtabs %}



#### 풀이

주어진 배열에서 1번째 인덱스를 기준 "키"로 설정하고, 비교하고자 하는 값은 키의 좌측 값으로 설정한다.

비교하고자 하는 값이 기준 키 보다 크다면 오른쪽으로 한 칸 밀고, 한 칸 더 좌측에 있는 값과 비교한다.

이 과정을 반복하다, 비교 대상이 키보다 작은 경우가 나오면, 키 값을 해당하는 위치에 삽입하고 다음 키를 기준으로 다시 반복한다.



### 선택 가이드

#### 시간 복잡도

* **BestCase(O(n))**: 정렬 대상이 거의 정렬이 완성된 상태는 요소를 오른쪽으로 미는 행위가 발생하지 않기 때문에 배열 요소의 숫자만큼만 비교하는 반복 횟수를 갖는다.
* **WorstCase(O(n**<sup>**2**</sup>**)):** 큰 수부터 정렬 되어있는 경우(역순 정렬) 모든 요소를 오른쪽으로 밀고, 비교하고 하는 행위를 거쳐야 하기 때문에 n<sup>2</sup> 만큼의 반복 횟수를 갖는다.
* **AverageCase(O(n**<sup>**2**</sup>**))**: 정렬 되어있지 않은 배열에서 평균적으로 절반정도 비교하고, 인덱스를 이동하는 행위를 거치기 때문에 n<sup>2</sup> 만큼의 반복 횟수를 갖는다.



#### 장점

1. 이미 메모리에 저장된 배열의 인덱스 위치를 변경하는 in-place 정렬 방법으로, 추가적인 메모리 공간이 필요하지 않고 배열에 직접 접근하기 때문에 O(1) 로 조회가 가능하다.
2. 안정 정렬 방식으로, 이미 정렬된 배열에서 다른 기준으로 재정렬 할 경우 흐트러지지 않는다.
3. 알고리즘 설명에 나와있듯이 기준 키와 비교하고자 하는 값의 크기 비교, 인덱스 이동 과정만 있기 때문에 구현이 단순하다.



#### 단점

1. 배열의 크기가 클 경우 데이터 비교와 인덱스 이동이 굉장히 많이 발생하는 비효율적인 알고리즘이다.



{% hint style="success" %}
#### 어떤 상황에 적용하면 좋을까?

배열의 크기가 작거나 대부분 정렬되어 있는 경우 O(n) 으로 단순하게 구현하여 효율적인 정렬이 가능하다.
{% endhint %}



**참고 자료**

***

{% embed url="https://www.tuanh.net/blog/java/-insertion-sort-in-java-for-optimal-performance" %}

{% embed url="https://gmlwjd9405.github.io/2018/05/06/algorithm-insertion-sort.html" %}

