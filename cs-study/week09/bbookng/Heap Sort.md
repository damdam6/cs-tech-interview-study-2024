# Heap Sort

- 완전 이진 트리를 기본으로 하는 힙(Heap) 자료구조를 기본으로 한 정렬 방식
	> 완전 이진 트리란 ?
	> : 삽입할 때 왼쪽부터 차례대로 추가하는 이진 트리 
- 불안정 정렬

### 💡 정렬 과정
1. 최대 힙을 구성
2. 현재 힙 루트는 가장 큰 값이 존재함. 루트의 값을 마지막 요소와 바꾼 후, 힙의 사이즈를 하나 줄임.
3. 힙의 사이즈가 1보다 크면 위 과정 반복

![img](https://t1.daumcdn.net/cfile/tistory/999896445AD4953023)
- 루트를 마지막 노드로 대체(11 → 4), 다시 최대 힙 구성

![img](https://t1.daumcdn.net/cfile/tistory/99E1AD445AD4953015)
- 이와 같은 방식으로 최대 값을 하나씩 뽑아내면서 정렬
### 💡 구현 코드
```java
package sortingAlgorithm;

public class HeapSort {
    static int[] arr = new int[10];

    public static void main(String[] args) {
        arr = new int[]{4, 3, 6, 5, 7, 8, 12, 2, 1, 10};
        heapSort(arr);

        for (int i = 0; i < arr.length; i++) {
            System.out.print(arr[i] + " ");
        }
    }

    private static void heapSort(int[] arr) {
        int size = arr.length;

        /*
         * 부모노드와 heaify과정에서 음수가 발생하면 잘못 된 참조가 발생하기 때문에
         * 원소가 1개이거나 0개일 경우는 정렬 할 필요가 없으므로 바로 함수를 종료한다.
         */
        if (size <= 1) {
            return;
        }

        // 가장 마지막 노드의 부모 노드 인덱스
        int parentIdx = getParent(size - 1);

        // max heap 만들기
        for (int i = parentIdx; i >= 0; i--) {

            // 부모노드 (i값)을 1씩 줄이면서 heap 조건을 만족시키도록 재구성.
            heapify(arr, i, size - 1);
        }

        // 정렬 과정
        for (int i = size - 1; i > 0 ; i--) {
            /*
             *  root인 0번째 인덱스와 i번째 인덱스의 값을 교환해준 뒤
             *  0 ~ (i-1) 까지의 부분트리에 대해 max heap을 만족하도록
             *  재구성한다.
             */
            swap(arr, 0, i);
            heapify(arr, 0, i - 1);
        }
    }

    private static void heapify(int[] arr, int parentIdx, int lastIdx) {
        // 현재 트리에서 부모 노드의 자식 노드 인덱스를 각각 구해준다.
        // 현재 부모 인덱스를 가장 큰 값으로 갖고 있다고 가정.
        int leftChildIdx = 2 * parentIdx + 1;
        int rightChildIdx = 2 * parentIdx + 2;
        int largestIdx = parentIdx;

        /*
         *  left child node와 비교
         *
         *  자식노드 인덱스가 끝의 원소 인덱스를 넘어가지 않으면서
         *  현재 가장 큰 인덱스보다 왼쪽 자식노드의 값이 더 클경우
         *  가장 큰 인덱스를 가리키는 largestIdx를 왼쪽 자식노드인덱스로 바꿔준다.
         *
         */
        if (leftChildIdx <= lastIdx && arr[largestIdx] < arr[leftChildIdx]) {
            largestIdx = leftChildIdx;
        }

        /*
         *  left right node와 비교
         *
         *  자식노드 인덱스가 끝의 원소 인덱스를 넘어가지 않으면서
         *  현재 가장 큰 인덱스보다 오른쪽 자식노드의 값이 더 클경우
         *  가장 큰 인덱스를 가리키는 largestIdx를 오른쪽 자식노드인덱스로 바꿔준다.
         *
         */
        if (rightChildIdx <= lastIdx && arr[largestIdx] < arr[rightChildIdx]) {
            largestIdx = rightChildIdx;
        }

        /*
         * largestIdx 와 부모노드가 같지 않다는 것은
         * 위 자식노드 비교 과정에서 현재 부모노드보다 큰 노드가 존재한다는 뜻이다.
         * 그럴 경우 해당 자식 노드와 부모노드를 교환해주고,
         * 교환 된 자식노드를 부모노드로 삼은 서브트리를 검사하도록 재귀 호출 한다.
         */

        if (parentIdx != largestIdx) {
            swap(arr, largestIdx, parentIdx);
            heapify(arr, largestIdx, lastIdx);
        }
    }

    private static int getParent(int child) {
        return (child - 1) / 2;
    }

    private static void swap(int[] arr, int i, int j) {
        int tmp = arr[i];
        arr[i] = arr[j];
        arr[j] = tmp;
    }
}

```

- **재귀를 하지 않는 `heapify`**
	- 성능 측면에서 개선 가능. 
	- 재귀 호출을 할 경우 정렬해야 할 원소가 많아지고, 매 재귀마다 부모 노드와 자식 노드가 교환된다면 최악의 경우 트리의 높이 만큼 재귀 호출이 발생. -> StackOverFlow가 발생할 수 있다. 
```java
private static void heapify_not_recursion(int[] arr, int parentIdx, int lastIdx) {

        int leftChildIdx;
        int rightChildIdx;
        int largestIdx;

        /*
         * 현재 부모 인덱스의 자식 노드 인덱스가
         * 마지막 인덱스를 넘지 않을 때 까지 반복한다.
         *
         * 이 때 왼쪽 자식 노드를 기준으로 해야 한다.
         * 오른쪽 자식 노드를 기준으로 범위를 검사하게 되면
         * 마지막 부모 인덱스가 왼쪽 자식만 갖고 있을 경우
         * 왼쪽 자식노드와는 비교 및 교환을 할 수 없기 때문이다.
         */
        while((parentIdx * 2) + 1 <= lastIdx) {
            leftChildIdx = (parentIdx * 2) + 1;
            rightChildIdx = (parentIdx * 2) + 2;
            largestIdx = parentIdx;

            /*
             * left child node와 비교
             * (범위는 while문에서 검사했으므로 별도 검사 필요 없음)
             */
            if (arr[leftChildIdx] > arr[largestIdx]) {
                largestIdx = leftChildIdx;
            }

            /*
             * right child node와 비교
             * right child node는 범위를 검사해주어야한다.
             */
            if (rightChildIdx <= lastIdx && arr[rightChildIdx] > arr[largestIdx]) {
                largestIdx = rightChildIdx;
            }

            /*
             * 교환이 발생했을 경우 두 원소를 교체 한 후
             * 교환이 된 자식노드를 부모 노드가 되도록 교체한다.
             */
            if (largestIdx != parentIdx) {
                swap(arr, parentIdx, largestIdx);
                parentIdx = largestIdx;
            }
            else {
                return;
            }
        }
    }
```

### 💡 특징
- Quick Sort와 Merge Sort의 성능이 좋기 때문에 Heap Sort의 사용 빈도는 높지 않음. 
- 가장 크거나 가장 작은 값을 구할 때 유용함.
	- 최소 힙 or 최대 힙의 루트 값이기 때문에 한 번의 힙 구성을 통해 구하는 것이 가능
- 최대 k 만큼 떨어진 요소들을 정렬할 때 유용함. 

### 💡 장단점
#### ✔ 장점
1. 최악의 경우에도 `O(nlogn)` 으로 유지가 된다. 
2. 힙의 특징상 부분 정렬을 할 때 효과가 좋다. 
#### ✔ 단점
1. 일반적인 `O(nlogn)` 알고리즘에 비해 성능은 약간 떨어진다. 
2. 한 번 최대 힙을 만들면서 불안정 정렬 상태에서 최댓값만 갖고 정렬을 하기 때문에 안정 정렬이 아니다. 