# Quick Sort

- 분할 정복(divide and conquer) 방법을 통해 주어진 배열을 정렬한다.
	> **분할 정복**
	> : 문제를 작은 2개의 문제로 분리하고 각각을 해결한 다음, 결과를 모아서 원래의 문제를 해결하는 전략.
- **불안정 정렬** (= 동일한 값에 대해 순서가 바뀔 수도 있다.)
	- 피벗을 기준으로 반 나눠서 정렬하기 때문에, 같은 값이 서로 떨어져 있다면 순서가 바뀔 수 있음. 
- **비교 정렬**
- **제자리 정렬**
	- 입력받은 배열 외에 다른 메모리가 필요하지 않다. 
- Merge Sort와 달리 Quick Sort는 배열을 비균등하게 분할함. 

### 💡 정렬 과정 
1. 배열 가운데서 하나의 원소를 고른다. 이렇게 고른 원소를 **피벗(pivot)** 이라고 한다. 
2. 피벗 앞에는 피벗보다 값이 작은 모든 원소들이 오고, 피벗 뒤에는 피벗보다 값이 큰 모든 원소들이 오도록 피벗을 기준으로 배열을 둘로 나눈다. 
   이렇게 배열을 둘로 나누는 것을 **분할(Divide)** 이라고 한다. 분할을 마친 뒤에 피벗은 더 이상 움직이지 않는다. 
3. 분할된 두 개의 작은 배열에 대해 재귀(Recursion)적으로 이 과정을 반복한다. 

- 재귀 호출이 한 번 진행될 때마다 최소한 하나의 원소는 최종적으로 위치가 정해지므로, 이 알고리즘은 반드시 끝난다는 것을 보장할 수 있다. 

![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fchw02p%2Fbtq5beAEWDZ%2FtNBgPWfHLr4X4WQigxK4KK%2Fimg.png)
![img](https://img1.daumcdn.net/thumb/R1920x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FzLHMQ%2Fbtq4Y6CJyJa%2FM0kMrxkquMHf7hyGk4fm50%2Fimg.png)
![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FxfaJZ%2Fbtq4YkuCRS5%2F370EyT409RKehnkHMkOoEk%2Fimg.png)

### 💡 구현 코드
```java
package sortingAlgorithm;

public class QuickSort {

    static int[] arr = new int[10];

    public static void main(String[] args) {
        arr = new int[]{4, 3, 6, 5, 7, 8, 12, 2, 1, 10};
        quickSort(arr, 0, arr.length - 1);

        for (int i = 0; i < arr.length; i++) {
            System.out.print(arr[i] + " ");
        }
    }

    private static void quickSort(int[] arr, int left, int right) {
        if (left >= right) return;

        // partitioning 을 통해 서브 리스트를 만들고, 기준이 된 pivot 값을 반환 받아 다시 해당 pivot을 기준으로 partitioning.
        int pivot = partition(arr, left, right);

        // pivot 보다 작은 서브리스트 재귀적으로 정렬
        quickSort(arr, left, pivot - 1);
        quickSort(arr, pivot + 1, right);
    }

    private static int partition(int[] arr, int left, int right) {
        int pivot = arr[left];
        int i = left;
        int j = right;

        while (i < j) {
            // pivot 보다 큰 값을 찾을 때 까지
            while (arr[j] >= pivot && i < j) {
                j --;
            }

            // pivot 보다 작은 값을 찾을 때 까지
            while (arr[i] <= pivot && i < j) {
                i ++;
            }

            swap(arr, i, j);
        }

        swap(arr, left, i);

        return i;
    }

    private static void swap(int[] arr, int i, int j) {
        int tmp = arr[i];
        arr[i] = arr[j];
        arr[j] = tmp;
    }
}
```

### 💡 장단점
#### ✔ 장점
1. 특정 상태가 아닌 이상 평균 시간 복잡도는 `O(nlogn)` 이며, 다른 `O(nlogn)` 알고리즘에 비해 대체적으로 속도가 매우 빠르다.
	- 유사하게 `O(nlogn)` 정렬 알고리즘 중 분할정복 방식인 Merge Sort에 비해 2~3배 정도 빠르다.
2. 추가적인 별도의 메모리를 필요로하지 않으며 재귀 호출 스택 프레임에 의한 공간복잡도는 `logN` 으로 메모리를 적게 소비한다.  
#### ✔ 단점
1. **특정 조건** 하에 성능이 급격하게 떨어진다.
2. 재귀를 사용하기 때문에 재귀를 사용하지 못하는 환경일 경우 그 구현이 매우 복잡해진다. 

### 💡시간복잡도

#### 1. 최선의 경우: O(NlogN)
> 우선, 배열이 잘 이등분 되는 경우를 생각해봅시다. 예를 들어, 배열 `[8, 4, 7, 3, 5, 2, 6, 1]`을 정렬한다고 해볼게요.
>
> 1. 첫 번째 단계에서 4를 피벗(pivot)으로 선택하고 나머지 숫자들을 비교하여, 피벗보다 작은 숫자 `[3, 2, 1]`과 큰 숫자 `[8, 7, 6, 5]`으로 나누어집니다. 이때 비교 연산은 총 7번 이루어집니다.
>
> 2. 이제 두 개의 하위 배열 `[3, 2, 1]`과 `[8, 7, 6, 5]`에 대해 다시 퀵 정렬을 수행합니다. 이 두 배열은 각각 크기가 4이므로, 다음 단계에서도 비교 연산은 각 배열당 3번씩 총 6번 이루어집니다.
>
> 3. 이 과정을 계속 반복하면, 매번 배열이 반씩 줄어들면서 비교 연산이 진행됩니다.
>
> 최종적으로, 각 단계에서 비교 연산의 총 횟수는 배열의 크기(N)에 비례하고, 단계의 깊이는 logN이 됩니다. 그래서 전체 시간복잡도는 O(NlogN)이 됩니다.

#### 2. 최악의 경우: O(N^2)
> 이번엔 배열이 이미 정렬되어 있는 경우를 생각해볼게요. 예를 들어, `[1, 2, 3, 4, 5, 6, 7, 8]` 같은 배열을 정렬한다고 해보죠.
>
> 1. 첫 번째 단계에서 1을 피벗으로 선택하고 나머지 숫자들을 비교합니다. 이때 비교 연산은 총 7번 이루어지고, 피벗보다 작은 값이 없으므로 한쪽 배열은 빈 배열이 되고, 나머지 배열은 `[2, 3, 4, 5, 6, 7, 8]`이 됩니다.
>
>    2. 다음 단계에서 2를 피벗으로 선택하고, 같은 방식으로 배열을 나누게 되면 또다시 한쪽 배열만 남게 됩니다. 이 과정이 계속 반복됩니다.
>
>       이렇게 되면, 매 단계에서 N, N-1, N-2, ..., 1번의 비교 연산이 필요하게 되어 총 비교 연산 횟수가 `N + (N-1) + (N-2) + ... + 1`이 됩니다. 이는 대략 O(N^2)에 해당하는 시간이 소요되죠.
#### ✔ 요약
> - **최선의 경우**: 배열이 매번 반으로 나뉘는 경우, 깊이는 logN, 각 단계에서 N번의 연산이 필요해서 전체 시간복잡도는 O(NlogN)입니다.
> - **최악의 경우**: 배열이 이미 정렬되어 있어 매번 한쪽 배열만 나뉘는 경우, 깊이는 N, 각 단계에서도 N번의 연산이 필요해서 전체 시간복잡도는 O(N^2)입니다.