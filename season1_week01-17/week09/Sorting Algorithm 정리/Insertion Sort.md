# Insertion Sort

### ✨ 삽입 정렬 (Insertion Sort)
- 정렬된 데이터 범위에 정렬되지 않은 데이터를 적절한 위치에 삽입하여 정렬하는 방식.
- 평균 시간복잡도는 `O(n^2)`으로 느린 편이지만 구현 난이도가 낮음.

- 특정 데이터를 리스트의 앞에서부터 **이미 정렬된 서브 리스트**의 값들과 비교하여 자신의 위치에 삽입하는 방식. 
- 이 때 이미 서브 리스트는 정렬이 되어있기 때문에 서브 리스트 안에서도 자신이 삽입되어야 할 위치가 정해져 있음. 그 위치에 데이터를 삽입하는 것이 **삽입 정렬**.

### 💡 과정
1. 현재 타겟이 되는 숫자와 이전 위에 있는 원소들(서브 리스트)을 비교한다. 
	- index 1부터 시작하며, index 0 을 첫 번째 서브리스트로 봄.
2. 타겟이 되는 숫자가 이전 위치에 있던 원소보다 작다면 위치를 서로 교환한다. 
3. 마지막 index까지 위 과정을 반복한다. 

![img](https://blog.kakaocdn.net/dn/bxvpd6/btqOuH69gZU/s5NmD45Lo0HaI80sK9QXt1/img.gif)

```java
package sortingAlgorithm;

public class InsertionSorting {
    static int[] arr = new int[10];
    public static void main(String[] args) {
        arr = new int[]{4, 3, 6, 5, 7, 8, 12, 2, 1, 10};
        insertionSort(arr);

        for (int i = 0; i < arr.length; i++) {
            System.out.print(arr[i] + " ");
        }
    }

    private static void insertionSort(int[] arr) {
        for (int i = 1; i < arr.length; i++) {
            int targetNumber = arr[i];
            int j = i-1;

            // 위치를 찾을 때 까지 subList 원소들이랑 뒤에서부터 (targetNumber 바로 전부터) 비교하며 swap 시켜줌
            while (j >= 0 && targetNumber < arr[j]) {
                // 한 칸씩 뒤로 미룸
                arr[j+1] = arr[j];
                // 그 다음 요소로 index 바꿔줌
                j --;
            }

            // while 문을 빠져나오고 나면 마지막 비교한 원소가 targetNumber 보다 작아서 탈출했기 때문에
            // 작은 원소 + 1의 index 에 targetNumber를 넣어 정렬해줌
            arr[j+1] = targetNumber;
        }
    }
}

```

### 💡 시간복잡도
> 최악의 경우(역으로 정렬되어 있을 경우) Selection Sort와 마찬가지로, `(n-1) + (n-2) + .... + 2 + 1 => n(n-1)/2` 즉, **O(n^2)** 이다.
>
> 하지만, 모두 정렬이 되어있는 경우(Optimal)한 경우, 한번씩 밖에 비교를 안하므로 **O(n)** 의 시간복잡도를 가지게 된다. 또한, 이미 정렬되어 있는 배열에 자료를 하나씩 삽입/제거하는 경우에는, 현실적으로 최고의 정렬 알고리즘이 되는데, 탐색을 제외한 오버헤드가 매우 적기 때문이다.
>
> 최선의 경우는 **O(n)** 의 시간복잡도를 갖고, 평균과 최악의 경우 **O(n^2)** 의 시간복잡도를 갖게 된다.

### 💡 장단점
#### ✔ 장점
1. 알고리즘이 단순하다.
2. 대부분의 원소가 이미 정렬되어 있는 경우, 매우 효율적일 수 있다. 
3. 정렬하고자 하는 배열 안에서 교환하는 방식이므로, 다른 메모리 공간을 필요로 하지 않는다. (제자리 정렬)
4. 안정 정렬이다. 
5. Selection Sort나 Bubble Sort와 같은 `O(n^2)` 알고리즘에 비해 상대적으로 빠르다. 
#### ✔ 단점
1. 평균과 최악의 시간복잡도가 `O(n^2)` 으로 비효율적이다.
2. Bubble Sort와 Selection Sort와 마찬가지로, 배열의 길이가 길어질수록 비효율적이다. 




- **Team Sort** 
	- Merge Sort + Insertion Sort 
---
[Ref](https://st-lab.tistory.com/179)