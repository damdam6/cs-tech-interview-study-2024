# Bubble Sort

- Selection Sort와 유사한 알고리즘으로 **서로 인접한 두 원소의 대소를 비교하고, 조건에 맞지 않다면 자리를 교환하며 정렬하는 알고리즘** 이다.
- 정렬 과정에서 원소의 이동이 마치 거품이 수면 위로 올라오는 것 같다고 해서 거품(Bubble) 이라는 이름이 붙음 ...
- 비교정렬
- 제자리 정렬
- 안정 정렬

### 💡 정렬 과정 (round)
1. 앞에서부터 현재 원소와 바로 다음의 원소를 비교한다. 
2. 현재 원소가 다음 원소보다 크면 원소를 교환한다.
3. 다음 원소로 이동하여 해당 원소와 그 다음 원소를 비교한다. 

- 이 때, 각 라운드를 진행할 때마다 뒤에서부터 한 개씩 정렬되기 때문에, 라운드가 진행될 때마다 한 번씩 줄면서 비교하게 된다.  
- 총 라운드는 **배열 크기 - 1번 진행**되고, 각 라운드 별 비교 횟수는 **배열 크기 - 라운드 횟수** 만큼 비교한다. 

![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdfQeBw%2FbtqT1848T64%2FH5D9U4z8dhELfRTkfk2k30%2Fimg.png)

### 💡 구현 코드
```java
package sortingAlgorithm;

public class BubbleSort {
    static int[] arr = new int[10];
    
    public static void main(String[] args) {
        arr = new int[]{4, 3, 6, 5, 7, 8, 12, 2, 1,10};
        bubbleSort(arr);

        for (int i = 0; i < arr.length; i++) {
            System.out.print(arr[i] + " ");
        }
    }
    
    private static void bubbleSort(int[] arr) {
        int size = arr.length;

        // 배열 size - 1 만큼 진행
        for (int i = 1; i < size; i++) {
            for (int j = 0; j < size - i; j++) {
                if (arr[j] > arr[j + 1]) {
                    swap(arr, j, j + 1);
                }
            }
        }
    }

    private static void swap(int[] arr, int i, int j) {
        int tmp = arr[i];
        arr[i] = arr[j];
        arr[j] = tmp;
    }
}
```

### 💡 시간복잡도
> 시간복잡도를 계산하면, `(n-1) + (n-2) + (n-3) + .... + 2 + 1 => n(n-1)/2`이므로, **O(n^2)** 이다. 
> 또한, Bubble Sort는 정렬이 돼있던 안돼있던, 2개의 원소를 비교하기 때문에 최선, 평균, 최악의 경우 모두 시간복잡도가 **O(n^2)** 으로 동일하다. 

### 💡 장단점
#### ✔ 장점
1. 구현이 매우 간단하고, 소스코드가 직관적이다.
2. 정렬하고자 하는 배열 안에서 교환하는 방식이므로, 다른 메모리 공간을 필요로 하지 않는다. (제자리 정렬)
3. 안정 정렬이다.
#### ✔ 단점
1. 시간복잡도가 최악, 최선, 평균, 모두 `O(n^2)`으로, 굉장히 비효율적이다.
2. 정렬되어 있지 않은 원소가 정렬됐을 때의 자리로 가기 위해서, 교환 연산 (swap)이 많이 일어나게 된다. 