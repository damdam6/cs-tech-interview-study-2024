# Merge Sort

- **병합정렬**, **합병 정렬**이라 부르며, **분할 정복** 방법을 통해 구현
- 요소를 쪼갠 후, 다시 합병 시키면서 정렬해 나가는 방식으로 쪼개는 방식은 Quick Sort와 유사함. 
- **안정 정렬**
- **비교 정렬**
- **제자리 정렬 X**  ( 입력 받은 배열 외에 다른 메모리가 필요하다.)
	- 합병할 때, 합병한 결과를 담을 배열이 필요하다. 

### 💡 정렬 과정
1. 주어진 리스트를 절반으로 분할하여 부분 리스트로 나눈다. 
2. 해당 부분리스트의 길이가 1이 아니라면 1번 과정을 되풀이한다. 
3. 인접한 부분리스트끼리 정렬하여 합친다. 

![img](https://img1.daumcdn.net/thumb/R1920x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F07jQt%2Fbtq1lao22zT%2FKkr0QfF1VGxi3bfGYp2r61%2Fimg.png)

### 💡 구현 코드
```java
package sortingAlgorithm;

public class MergeSort {
    static int[] arr = new int[10];
    static int[] sorted;

    public static void main(String[] args) {
        arr = new int[]{4, 3, 6, 5, 7, 8, 12, 2, 1, 10};
        sorted = new int[arr.length];

        mergeSort(arr, 0, arr.length - 1);
        sorted = null;

        for (int i = 0; i < arr.length; i++) {
            System.out.print(arr[i] + " ");
        }
    }

    private static void mergeSort(int[] arr, int left, int right) {
        // 더이상 쪼갤 수 없으므로 return
        if (left == right) return;

        int mid = (left + right) / 2;

        mergeSort(arr, left, mid); // left ~ mid
        mergeSort(arr, mid + 1, right); // mid + 1 ~ right

        // 병합
        merge(arr, left, mid, right);
    }

    private static void merge(int[] arr, int left, int mid, int right) {
        int l = left; // 왼쪽 부분리스트 시작점
        int r = mid + 1; // 오른쪽 부분리스트 시작점
        int idx = left; // sorted 배열을 채워넣기 위한 인덱스

        while (l <= mid && r <= right) {
            if (arr[l] <= arr[r]) {
                sorted[idx] = arr[l];
                idx ++;
                l ++;
            } else {
                sorted[idx] = arr[r];
                idx ++;
                r ++;
            }
        }

        if (l > mid) {
            while (r <= right) {
                sorted[idx] = arr[r];
                idx ++;
                r ++;
            }
        } else {
            while (l <= mid) {
                sorted[idx] = arr[l];
                idx ++;
                l ++;
            }
        }

        for (int i = left; i <= right; i++) {
            arr[i] = sorted[i];
        }
    }
}

```

### 💡 장단점
#### ✔ 장점
1. 항상 두 부분리스트로 쪼개어 들어가기 때문에 최악의 경우에도 `O(nlogn)` 으로 유지가 된다. 
2. 안정정렬이다. 
#### ✔ 단점
1. 정렬과정에서 추가적인 보조 배열 공간을 사용하기 때문에 메모리 사용량이 많다. 
2. 보조 배열에서 원본 배열로 복사하는 과정은 매우 많은 시간을 소비하기 때문에 데이터가 많을 경우 상대적으로 시간이 많이 소요된다. 

---
### 💡 LinkedList 를 이용한 MergeSort
```java
package sortingAlgorithm;

public class MergeSort_LinkedList {
    static class Node {
        int value;
        Node next;

        public Node(int value) {
            this.value = value;
            this.next = null;
        }
    }

    public static class MergeSortLinkedList {
        Node head;

        // 새로운 노드 추가
        public void add(int value) {
            Node newNode = new Node(value);
            newNode.next = head;
            head = newNode;
        }

        public void printList(Node node) {
            while (node != null) {
                System.out.print(node.value + " ");
                node = node.next;
            }
            System.out.println();
        }

        public Node mergeSort(Node head) {
            if (head == null || head.next == null) {
                return head;
            }

            // 분할 과정
            Node mid = getMid(head);
            Node nextOfMid = mid.next;

            // mid의 next를 Null로 설정하여 끝을 마킹함으로써 리스트를 두 부분으로 나눔.
            mid.next = null;

            // head부터 mid까지
            Node left = mergeSort(head);
            // mid + 1 부터 끝까지
            Node right = mergeSort(nextOfMid);

            Node sortedList = sortedMerge(left, right);
            return sortedList;
        }

        public Node sortedMerge(Node a, Node b) {
            Node result;

            if (a == null) {
                return b;
            }

            if (b == null) {
                return a;
            }

            // 정렬
            if (a.value <= b.value) {
                result = a;
                result.next = sortedMerge(a.next, b);
            } else {
                result = b;
                result.next = sortedMerge(a, b.next);
            }
            return result;
        }

        // 투포인터 (토끼와 거북이 알고리즘)
        public Node getMid(Node head) {
            if (head == null) {
                return head; // 빈 리스트의 경우 중간 노드는 없음.
            }

            Node slow = head, fast = head;

            while (fast.next != null && fast.next.next != null) {
                slow = slow.next;
                fast = fast.next.next;
            }
            return slow;
        }
    }

    public static void main(String args[]) {
        MergeSortLinkedList list = new MergeSortLinkedList();
        list.add(15);
        list.add(10);
        list.add(5);
        list.add(20);
        list.add(3);
        list.add(7);

        System.out.print("Original List: ");
        list.printList(list.head);

        list.head = list.mergeSort(list.head);

        System.out.print("Sorted List: ");
        list.printList(list.head);
    }
}

```