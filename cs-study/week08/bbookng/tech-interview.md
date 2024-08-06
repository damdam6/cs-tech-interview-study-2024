#### 1. Array는 어떤 자료구조 인가요 ?
> Array 는 연관된 data를 **메모리상에 연속적이며 순차적**으로 **미리 할당된 크기**만큼 저장하는 자료구조입니다. 
- **미리 예상한 것보다 더 많은 수의 data를 저장하느라 Array의 size를 넘어서게 됐습니다. 이 때, 어떻게 해결할 수 있을까요 ?**
	
	> Dynamic Array 를 사용하여 resize를 하는 방식으로 유동적으로 size를 조절할 수 있습니다. Dynamic Array 는 자동으로 Resize를 해주긴 하지만 Dynamic Array 를 사용하지 않더라도 현재 Array 보다 더 큰 size의 Array를 선언하고 기존 값들을 복사하는 방법을 사용할 수 있습니다.  혹은 Linked List 자료구조를 사용한다면 동적으로 크기를 변경할 수 있습니다.
#### 2. Dynamic Array는 어떤 자료구조 인가요?
> Array의 경우 size가 고정되었기 때문에 선언시에 설정한 size보다 많은 갯수의 data가 추가되면 저장할 수 없습니다. 이에 반해 Dynamic Array는 저장공간이 가득 차게 되면 resize를 하여 유동적으로 size를 조절하여 데이터를 저장하는 자료구조입니다. 
- **Dynamic Array를 Linked List와 비교하여 장단점을 설명해주세요.** 
>		Dynamic Array는 `O(1)`이라는 시간복잡도를 가지는 빠른 인덱싱과 메모리에 순차적으로 저장되기 때문에 메모리 효율성이 장점이지만, 삽입/삭제 시 비용이 높고 크기를 초과했을 때 resizing을 하는 과정에서 메모리 재할당이 필요하다는 단점이 있습니다.
>		반면에 Linked List 는 삽입/삭제가 용이하고 크기 조절이 유연하여 메모리 재할당이 필요없다는 장점이 있지만 특정 위치에 접근하기 위해서는 `O(n)`이라는 시간복잡도로 인덱싱이 느리다는 점과 각 노드가 다음 노드에 대한 참조를 포함해야 하므로 추가적인 메모리를 사용한다는 단점이 있습니다. 

#### 3. Linked List에 대해서 설명해주세요.
> Linked List는 Node라는 구조체로 이루어져 있는데, Node는 데이터 값과 다음 Node의 address를 저장합니다. Linked List는 물리적인 **메모리상에서는 비연속적으로 저장**이 되지만 Linked List를 구성하는 각각의 **Node가 next Node의 address를 가리킴으로써 논리적인 연속성**을 가진 자료구조입니다. 
#### 4. Array와 Linked list를 비교해서 설명해주세요.
> Array는 메모리 상에서 연속적으로 데이터를 저장하는 자료구조입니다. Linked list는 메모리 상에서는 연속적이지 않지만, 각각의 원소가 다음 원소의 메모리 주소값을 저장해 놓음으로써 논리적 연속성을 유지합니다. 
> 그래서 각 operation의 시간복잡도가 다릅니다. 데이터 조회는 Array의 경우 `O(1)`, Linked list는 `O(n)`이며 삽입/삭제는 Array는 `O(n)`, Linked list의 경우 `O(1)`의 시간복잡도를 갖습니다.
> 따라서 얼마만큼의 데이터를 저장할지 미리 알고있고, 조회를 많이 한다면 Array를 사용하는 것이 좋지만 몇 개의 데이터를 저장할 지 불확실하고 삽입 및 삭제가 잦다면 Linked List를 사용하는 것이 좋습니다. 
- **어느 상황에 Linked List를 쓰는게 Array 보다 나을까요 ?** 
> 		삽입 및 삭제가 빈번한 상황에서 Linked List를 사용하는 것이 유리합니다. 예를 들어, 요소를 자주 추가하거나 삭제해야 하는 경우, 특히 중간이나 앞부분에서 작업이 많을 때 Linked List는 효율적입니다. 동적 메모리 할당을 통해 크기를 조절할 수 있으며, 초기 크기를 지정할 필요가 없어 메모리 관리가 유연합니다.
- **어느 상황에 Array를 쓰는게 Linked List 보다 나을까요 ?**
> 		데이터를 자주 조회해야 하며, 크기가 고정되어 있거나 예측 가능한 경우 Array를 사용하는 것이 더 좋습니다. 배열은 연속된 메모리 공간을 사용하므로, 인덱스를 통한 데이터 접근이 매우 빠르며, 캐시 효율성도 높습니다. 따라서, 정적인 데이터 집합이나 읽기 작업이 빈번한 경우 Array가 더 적합합니다.
- **Array와 Linked List의 memory allocation은 언제 일어나며, 메모리의 어느 영역을 할당 받나요 ?**
> 		Array는 일반적으로 프로그램이 시작될 때 스택(Stack) 또는 힙(Heap) 메모리 영역에서 연속된 공간을 할당받습니다. 고정 크기의 배열은 주로 스택에서 할당되며, 동적 배열은 힙에서 할당됩니다. Linked List는 각 노드가 필요할 때마다 동적으로 힙 메모리에서 개별적으로 할당됩니다. 이는 메모리의 불연속적인 할당을 의미하지만, 삽입/삭제 작업이 용이해집니다.
#### 5. Queue는 어떤 자료구조 인가요 ?
> queue는 선입선출의 자료구조입니다. 시간복잡도는 enqueue가 `O(1)`, dequeue가 `O(1)` 입니다. 
> 활용 예시로는 Cache 구현, 프로세스 관리, BFS 등이 있습니다. 
- **Array-Base 와 List-Base 의 경우 어떤 차이가 있나요 ?** 
> 		Array-Base Queue의 경우 고정된 크기의 배열을 사용하여 구현되며, 구현이 간단하고 접근 속도가 빠릅니다. 그러나 크기를 미리 정의해야 하며 비어있는 공간이 있을 시 메모리 낭비가 발생할 수 있습니다. 
> 		List-Base의 경우 Linked List를 사용하여 구현되며 필요한 만큼의 메모리만 사용합니다. 그러나 구현이 복잡하고 각 노드에 추가 메모리 오버헤드가 있습니다. 
>
#### 6. Stack은 어떤 자료구조인가요 ? 
> stack은 후입선출의 자료구조입니다. 시간복잡도는 push `O(1)`, pop `O(1` 입니다.
> 활용 예시로는 괄호 유효성 검사, 웹 브라우저 방문 기록 (뒤로 가기), DFS 등이 있습니다. 
#### 7. **Stack 두 개**를 이용하여 **Queue**를 구현해 보세요.
> queue의 **enqueue()** 를 구현할 때 **첫 번째 stack**을 사용하고, **dequeue()** 를 구현할 때 **두 번째 stack**을 사용하면 queue를 구현할 수 있습니다. 
> 편의상 enqueue()에 사용할 stack을 instack이라고 부르고, dequeue()에 사용할 stack을 outstack이라고 부르겠습니다. 두 개의 stack으로 queue를 구현하는 방법은 다음과 같습니다. 
> 1. enqueue() :: instack에 push()를 하여 데이터를 저장합니다. 
> 2. dequeue() :: 
> 	a. 만약 outstack이 비어있다면 instack.pop()을 하고 oustack.push()를 하여 instack에서 outstack으로 모든 데이터를 옮겨 넣습니다. 이 결과로 가장 먼저 들어간 데이터는 outstack의 top에 위치하게 됩니다. 
> 	b. outstack.pop()을 하면 가장 먼저 들어갔던 데이터가 가장 먼저 추출이 됩니다. (FIFO)

```python
class Queue(object):
	def __init__(self):
		self.instack = []
		self.outstack = []
		
	def enqueue(self, element):
		self.instack.append(element)

	def dequeue(self):
		if not self.outstack:
			while self.instack:
				self.outstack.append(self.instack.pop())
		return self.outstack.pop()
```

```java
public class Queue {
	Stack<Integer> inStack;
	Stack<Integer> outStack;

	public Queue() {
		inStack = new Stack<>();
		outStack = new Stack<>();
	}
	
	public void enQueue(int v) {
		inStack.push(v);
	}
	
	public int deQueue() {
		if (outStack.isEmpty()) {
			while (!inStack.isEmpty()) {
				outStack.push(inStack.pop());
			}
		}
		
		return outStack.pop();
	}
}
```
#### 8. **Queue 두 개**를 이용하여 Stack을 구현해 보세요. 
> 편의상 push()에 사용할 queue는 q1이라고 부르고 pop()에 사용할 queue를 q2라고 칭하겠습니다. 두 개의 queue를 이용하여 stack을 구현하는 방법은 다음과 같습니다. 
> 1. push() :: q1으로 enqueue()를 하여 데이터를 저장합니다. 
> 2. pop() ::
> 	a. q1에 저장된 데이터의 갯수가 1 이하로 남을 때까지 dequeue()를 한 후, 추출된 데이터를 q2에 enqueue() 합니다. 결과적으로 가장 최근에 들어온 데이터를 제외한 모든 데이터는 q2로 옮겨집니다. 
> 	b. q1에 남아 있는 하나의 데이터를 dequeue()해서 가장 최근에 저장된 데이터를 반환합니다. (LIFO)
> 	c. 다음에 진행될 pop()을 위와 같은 알고리즘으로 진행하기 위해 q1과 q2의 이름을 swap 합니다. 

```python
import queue

class Stack(object):
	def __init__(self):
		self.q1 = queue.Queue()
		self.q2 = queue.Queue()
	
	def push(self, element):
		self.q1.put(element)
		
	def pop(self):
		while self.q1.qsize() > 1:
			self.q2.put(self.q1.get())
		
		temp = self.q1
		self.q1 = self.q2
		self.q2 = temp
		
		return self.q2.get()
```

```java
import java.util.LinkedList;
import java.util.Queue;

public class Stack {
	private Queue<Integer> q1;
	private Queue<Integer> q2;
	
	public Stack() {
		q1 = new LinkedList<>();
		q2 = new LinkedList<>();
	}
	
	public void push(int v) {
		q1.offer(v);
	}
	
	public int pop() {
		while (q1.size() > 1) {
			q2.offer(q1.poll());
		}
		
		Queue<Integer> temp = q1;
		q1 = q2;
		q2 = temp;
		
		return q2.poll();
	}
}
```
#### 9. Queue와 Priority queue를 비교하여 설명해 주세요.
> Queue 자료구조는 시간 순서상 먼저 집어 넣은 데이터가 먼저 나오는 **선입선출** 구조로 저장하는 형식입니다. 이와 다르게 우선순위 큐는 들어간 순서에 상관없이 우선순위가 높은 데이터가 먼저 나옵니다. 
> Queue의 operation 시간복잡도는 `enqueue` `O(1)`, `dequeue` `O(1)` 이고, Priority queue는 `push` `O(logn)`, `pop` `O(logn)` 입니다. 
#### 10. BST는 어떤 자료구조 인가요 ?
> 이진탐색트리(Binary Search Tree; BST는 정렬된 Tree입니다. 어느 node를 선택하든 해당 node의 left subtree에는 그 node의 값보다 작은 값들을 지닌 node들로만 이루어져 있고, node의 right subtree에는 그 node의 값보다 큰 값들을 지닌 node들로만 이루어져 있는 binary tree입니다. 
> 검색과 저장, 삭제의 시간복잡도는 모두 `O(logn)`이고, worst case는 한쪽으로 치우친 tree가 됐을 때 `O(n)`입니다. 
- **이진트리(Binary Tree)는 어떤 자료구조 인가요 ?**
> 		이진트리는 각 노드가 최대 두 개의 자식을 가지는 트리 자료구조입니다.  각 노드는 왼쪽 자식과 오른쪽 자식을 가질 수 있으며, 이진트리에는 특별한 정렬 규칙이 없기 때문에 모든 종류의 데이터 구조를 포함할 수 있습니다. 
- **BST의 worst case 시간복잡도는 `O(n)`입니다. 어떠한 경우에 worst case가 발생하나요 ?**
>		트리의 균형이 한쪽으로 치우쳐진 경우에 발생합니다. 
- **해결방법은 무엇인가요?**
> 		해결 방법은 트리를 균형 있게 유지하는 것입니다. 이를 위해 Balanced Binay Tree 를 사용할 수 있는데, 대표적으로 AVL 트리와 Red-Black Tree가 있습니다. 
> 		이러한 트리들은 삽입과 삭제 시 자동으로 트리를 균형 상태로 유지하여, 최악의 경우에도 시간복잡도가 `O(logn)`이 되도록 보장합니다. 
#### 11. Hash table은 어떤 자료구조 인가요?
> hash table은 **효율적인 탐색(빠른 탐색)** 을 위한 자료구조로써 `key-value` 쌍의 데이터를 입력받습니다. hash function `h`에 key값을 입력으로 넣어 얻은 해시값 `h(k)`를 위치로 지정하여 `key-value` 데이터 쌍을 저장합니다. 저장, 삭제, 검색의 시간복잡도는 모두 `O(1)` 입니다.
- **좋은 hash function 의 조건은 뭘까요 ?**
	
	> 좋은 hash function 의 조건은 hash collision을 최소화하고 해시 값을 균등하게 분포시키며, 계산이 빠르고 효율적이어야 합니다. 
#### 12. Hash table에서 collision이 발생하면 어떻게 되나요 ? 해결방법엔 무엇이 있을까요 ?
> collision이 발생할 경우 대표적으로 2가지 방법으로 해결합니다. 
> 첫 번째, open addressing 방식은 collision이 발생하면 미리 정한 규칙에 따라 hash table의 비어있는 slot을 찾습니다. 빈 slot을 찾는 방법에 따라 크게 Linear Probing, Quadratic Probing, Double Hashing으로 나뉩니다. 
> 두 번째, seperete chaining 방식은 linked list를 이용합니다. 만약에 collision이 발생하면 linked list에 노드(slot)를 추가하여 데이터를 저장합니다. 
- **worst case에 시간복잡도는 `O(n)`이라고 했는데 어떤 상황인가요 ?**
	
	> 해시 테이블의 모든 키가 동일한 해시 값을 가질 때 발생합니다. 이 경우, 모든 요소가 하나의 슬롯에 저장되어 선형 탐색이 필요하게 됩니다. 
- **이중해싱(double hashing)이 무엇인지 설명해 주세요.**
	
	> 이중해싱은 충돌을 해결하기 위한 open addressing 방식 중 하나입니다. 첫 번째 해시함수 `h1(k)` 로 충돌이 발생한 경우, 두 번째 해시 함수 `h2(k)`를 사용하여 이동할 간격을 결정합니다. 이동할 간격은 `h2(k)`에 결정되므로, 충돌이 발생해도 다른 슬롯을 탐색하게 되어 충돌을 줄일 수 있습니다. 
	> 이중 해싱은 두 해시 함수를 사용함으로써 더 균등하게 슬롯을 탐색할 수 있게 합니다. 