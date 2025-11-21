# 1주차: 기본 자료구조 - 배열, 리스트, 스택, 큐, 덱

## 🎯 학습 목표
- **심층 이해**: 자료구조의 메모리 레벨 동작 원리와 성능 특성을 깊이 이해합니다.
- **구현 능력**: 파이썬 클래스로 각 자료구조를 직접 구현하고 최적화합니다.
- **활용 능력**: 실제 시스템에서 각 자료구조가 어떻게 사용되는지 파악합니다.

## 📚 핵심 개념 심층 분석

### 1. 배열 (Array) & 동적 배열 (Dynamic Array)

#### 🧠 심층 동작 원리: 메모리와 캐시
- **메모리 구조**: 배열은 **연속된 메모리 블록**을 사용합니다. 이는 물리적 메모리(RAM) 상에서 데이터가 나란히 붙어있음을 의미합니다.
- **Locality of Reference (참조 지역성)**: CPU는 메모리에서 데이터를 읽을 때 캐시 라인(Cache Line) 단위로 읽어옵니다. 배열은 연속되어 있으므로, 인덱스 `i`를 읽을 때 `i+1`, `i+2`도 함께 캐시에 로드될 확률이 높습니다. 이를 **공간 지역성(Spatial Locality)**이라 하며, 이로 인해 연결 리스트보다 실제 접근 속도가 훨씬 빠릅니다.
- **Amortized Analysis (분할 상환 분석)**: 동적 배열의 `append` 연산은 평소에는 O(1)이지만, 꽉 찼을 때(Resize)는 O(n)이 걸립니다. 하지만 Resize는 드물게 발생하므로, 전체 연산 비용을 평균내면 **Amortized O(1)**이 됩니다.

#### 📊 복잡도 분석
| 연산 | 평균 시간 복잡도 | 최악 시간 복잡도 | 설명 |
|:---:|:---:|:---:|:---|
| 접근 (Get) | O(1) | O(1) | 인덱스 주소 계산: `base_addr + index * size` |
| 검색 (Search) | O(n) | O(n) | 값을 찾기 위해 순차 탐색 필요 |
| 삽입 (Append) | O(1)* | O(n) | *Amortized. Resize 발생 시 O(n) |
| 삽입 (Insert) | O(n) | O(n) | 중간에 끼워넣으려면 뒤의 요소들을 밀어야 함 (Shift) |
| 삭제 (Delete) | O(n) | O(n) | 중간 요소를 지우면 뒤의 요소들을 당겨와야 함 |

### 2. 연결 리스트 (Linked List)

#### 🧠 심층 동작 원리: 포인터와 유연성
- **메모리 구조**: 노드들이 메모리 여기저기에 흩어져 있고, 포인터(참조)로 연결됩니다.
- **Cache Miss**: 다음 노드를 찾기 위해 포인터를 따라갈 때, 해당 메모리가 캐시에 없을 확률이 높습니다(Cache Miss). 따라서 순차 접근 시 배열보다 느립니다.
- **오버헤드**: 각 노드마다 데이터 외에 포인터(주소)를 저장할 추가 메모리가 필요합니다. (32bit 시스템 4바이트, 64bit 시스템 8바이트 추가)

#### 💡 배열 vs 연결 리스트: 언제 무엇을 쓸까?
- **배열**: 데이터 개수가 정해져 있거나, 빈번한 인덱스 접근(Random Access)이 필요할 때. (예: 주식 가격 차트, 이미지 픽셀 데이터)
- **연결 리스트**: 데이터 개수가 예측 불가능하고, 중간 삽입/삭제가 빈번할 때. (예: 운영체제의 프로세스 리스트, 음악 플레이어의 재생 목록)

### 3. 스택 (Stack) & 큐 (Queue)

#### 🌍 Real-world Use Cases (실제 활용 사례)
- **스택 (LIFO)**:
    - **함수 호출 (Call Stack)**: 프로그래밍 언어에서 함수가 호출될 때 지역 변수와 리턴 주소를 저장하는 곳이 스택 구조입니다. 재귀 함수가 깊어지면 `Stack Overflow`가 발생하는 이유입니다.
    - **웹 브라우저 뒤로 가기**: 방문 기록을 스택에 쌓아두고 뒤로 가기를 누르면 `pop` 합니다.
    - **Undo (실행 취소)**: 텍스트 에디터의 작업 내역 저장.
- **큐 (FIFO)**:
    - **프로세스 스케줄링**: CPU를 기다리는 프로세스들이 줄을 서는(Ready Queue) 구조입니다.
    - **프린터 대기열**: 인쇄 요청이 들어온 순서대로 처리합니다.
    - **너비 우선 탐색 (BFS)**: 그래프 탐색 시 방문할 노드를 순서대로 저장합니다.

---

## 💻 파이썬 구현 가이드

### 1. 동적 배열 (Dynamic Array) 구현
```python
import ctypes

class DynamicArray:
    def __init__(self):
        self.n = 0 # 현재 요소 개수
        self.capacity = 1 # 현재 용량
        self.A = self._make_array(self.capacity) # 실제 배열 (ctypes 사용)

    def __len__(self):
        return self.n

    def __getitem__(self, k):
        if not 0 <= k < self.n:
            raise IndexError('invalid index')
        return self.A[k]

    def append(self, obj):
        if self.n == self.capacity:
            self._resize(2 * self.capacity) # 용량 2배 확장 (Doubling)
        self.A[self.n] = obj
        self.n += 1

    def _resize(self, c):
        """새로운 더 큰 배열을 만들고 기존 데이터를 복사"""
        B = self._make_array(c)
        for k in range(self.n):
            B[k] = self.A[k]
        self.A = B
        self.capacity = c

    def _make_array(self, c):
        return (c * ctypes.py_object)()

# 테스트
arr = DynamicArray()
arr.append(1)
print(len(arr)) # 1
print(arr[0])   # 1
```

### 2. 단일 연결 리스트 (Singly Linked List) 구현
```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class SinglyLinkedList:
    def __init__(self):
        self.head = None
        self.size = 0

    def append(self, data):
        new_node = Node(data)
        if not self.head:
            self.head = new_node
        else:
            current = self.head
            while current.next: # 마지막 노드 찾기 (O(n)) -> tail 포인터로 최적화 가능
                current = current.next
            current.next = new_node
        self.size += 1

    def delete(self, data):
        # 구현 과제: data를 가진 첫 번째 노드를 삭제하는 로직 구현
        # 1. head가 삭제 대상일 때
        # 2. 중간/끝 노드가 삭제 대상일 때 (이전 노드의 next를 변경)
        pass

    def __str__(self):
        # 리스트 내용을 문자열로 반환 (예: 1 -> 2 -> 3)
        pass
```

### 3. 스택 (Stack) 구현
```python
class Stack:
    def __init__(self):
        self.items = []

    def push(self, item):
        self.items.append(item)

    def pop(self):
        if not self.is_empty():
            return self.items.pop()
        raise IndexError("pop from empty stack")

    def peek(self):
        if not self.is_empty():
            return self.items[-1]
        
    def is_empty(self):
        return len(self.items) == 0
```

### 4. 큐 (Queue) 구현 (Node 기반)
```python
class Queue:
    def __init__(self):
        self.front = None # 삭제 위치 (Dequeue)
        self.rear = None  # 삽입 위치 (Enqueue)
        self.size = 0

    def enqueue(self, item):
        new_node = Node(item)
        if not self.rear:
            self.front = self.rear = new_node
        else:
            self.rear.next = new_node
            self.rear = new_node
        self.size += 1

    def dequeue(self):
        # 구현 과제: front 노드를 반환하고 삭제하는 로직 구현
        # front를 다음 노드로 이동
        # 빈 큐일 때 처리 주의
        pass
```

### 5. 덱 (Deque) 구현 (Doubly Linked List 기반)
```python
class DNode:
    def __init__(self, data):
        self.data = data
        self.prev = None
        self.next = None

class Deque:
    def __init__(self):
        self.front = None
        self.rear = None
    
    def add_front(self, item):
        # 구현 과제: 새로운 노드를 앞에 추가
        pass
        
    def add_rear(self, item):
        # 구현 과제: 새로운 노드를 뒤에 추가
        pass
        
    def remove_front(self):
        # 구현 과제: 앞쪽 노드 삭제 및 반환
        pass
        
    def remove_rear(self):
        # 구현 과제: 뒤쪽 노드 삭제 및 반환
        pass
```

## ✅ 이번 주 과제
1. `SinglyLinkedList`의 `delete`, `find` 메서드 완성하기
2. `Queue`의 `dequeue` 메서드 완성하기
3. `Deque`의 4가지 핵심 연산 완성하기
4. (심화) `DynamicArray`에서 요소 삭제 시 용량을 줄이는(Shrink) 로직 추가해보기 (예: 사용량이 25% 미만일 때 용량을 절반으로 줄임)
