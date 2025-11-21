# 3주차: 그래프 구조 - 인접 리스트/행렬, DFS/BFS, 최단 경로

## 🎯 학습 목표
- **심층 이해**: 그래프의 표현 방식에 따른 성능 차이와 탐색 알고리즘의 원리를 깨닫습니다.
- **구현 능력**: 복잡한 연결 관계를 코드로 모델링하고 효율적으로 탐색합니다.
- **활용 능력**: 네비게이션, 소셜 네트워크 등 현실 세계의 문제를 그래프로 해결합니다.

## 📚 핵심 개념 심층 분석

### 1. 그래프 표현: 행렬 vs 리스트

#### 📊 공간 복잡도와 선택 기준
- **인접 행렬 (Adjacency Matrix)**: `V x V` 크기의 2차원 배열.
    - 공간 복잡도: **O(V^2)**. 정점이 10,000개면 1억 개의 칸이 필요(약 400MB).
    - 장점: 두 정점 연결 여부 확인이 **O(1)**.
    - 단점: 간선이 적은 희소 그래프(Sparse Graph)에서는 메모리 낭비가 심함.
- **인접 리스트 (Adjacency List)**: 각 정점에 연결된 이웃만 리스트로 저장.
    - 공간 복잡도: **O(V + E)**.
    - 장점: 메모리를 효율적으로 사용. 대부분의 실제 그래프(도로망, 친구 관계)는 희소 그래프이므로 인접 리스트가 선호됨.

### 2. DFS vs BFS: 탐색의 철학

#### 🔍 미로 찾기 vs 물결 퍼짐
- **DFS (깊이 우선 탐색)**: **미로 찾기**와 같습니다. 갈 수 있는 데까지 막다른 길(Leaf)이 나올 때까지 깊게 들어갑니다. 막히면 되돌아옵니다(Backtracking).
    - 구현: **스택** 또는 **재귀** 사용.
    - 활용: 모든 경로 탐색, 사이클 탐지, 위상 정렬.
- **BFS (너비 우선 탐색)**: 호수에 돌을 던졌을 때 **물결이 퍼져나가는 것**과 같습니다. 시작점에서 가까운 곳부터 층층이(Level-by-Level) 방문합니다.
    - 구현: **큐** 사용.
    - 활용: **가중치 없는 그래프의 최단 경로**, 가까운 이웃 찾기.

### 3. 다익스트라 (Dijkstra) 알고리즘

#### 🧠 그리디(Greedy)의 정당성
- **원리**: "현재 갈 수 있는 가장 가까운 정점을 확정하면, 그 정점까지의 최단 거리는 변하지 않는다"는 그리디적 가정을 기반으로 합니다.
- **왜 음수 간선이 있으면 안 되나?**: 다익스트라는 한 번 방문 확정된 노드는 다시 보지 않습니다. 하지만 음수 간선이 있다면, 멀리 돌아가는 경로가 나중에 음수 가중치로 인해 더 짧아질 수 있는 가능성이 생기는데, 다익스트라는 이를 감지하지 못합니다. (음수 간선이 있을 땐 벨만-포드 알고리즘 사용)

---

## 💻 파이썬 구현 가이드

### 1. 그래프 구현 (인접 리스트)
```python
class Graph:
    def __init__(self):
        self.graph = {} # {node: {neighbor: weight}}

    def add_edge(self, u, v, weight=1, directed=False):
        if u not in self.graph:
            self.graph[u] = {}
        self.graph[u][v] = weight
        
        if not directed:
            if v not in self.graph:
                self.graph[v] = {}
            self.graph[v][u] = weight

    def print_graph(self):
        for node in self.graph:
            print(f"{node} -> {self.graph[node]}")

# 테스트
g = Graph()
g.add_edge('A', 'B', 5)
g.add_edge('A', 'C', 1)
g.print_graph()
```

### 2. DFS (깊이 우선 탐색)
```python
def dfs_recursive(graph, node, visited=None):
    if visited is None:
        visited = set()
    
    visited.add(node)
    print(node, end=' ')
    
    for neighbor in graph.get(node, {}):
        if neighbor not in visited:
            dfs_recursive(graph, neighbor, visited)

def dfs_stack(graph, start_node):
    visited = set()
    stack = [start_node]
    
    while stack:
        node = stack.pop()
        if node not in visited:
            visited.add(node)
            print(node, end=' ')
            # 스택이므로 역순으로 넣어야 순서대로 방문 가능
            for neighbor in reversed(list(graph.get(node, {}))):
                if neighbor not in visited:
                    stack.append(neighbor)
```

### 3. BFS (너비 우선 탐색)
```python
from collections import deque

def bfs(graph, start_node):
    visited = set()
    queue = deque([start_node])
    visited.add(start_node)
    
    while queue:
        node = queue.popleft()
        print(node, end=' ')
        
        for neighbor in graph.get(node, {}):
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
```

### 4. 다익스트라 (최단 경로)
```python
import heapq

def dijkstra(graph, start):
    # 거리 테이블 초기화 (무한대)
    distances = {node: float('inf') for node in graph}
    distances[start] = 0
    
    # 우선순위 큐: (거리, 노드) -> 거리가 짧은 순서대로 나옴
    pq = [(0, start)]
    
    while pq:
        current_dist, current_node = heapq.heappop(pq)
        
        # 이미 처리된 노드라면 무시 (더 짧은 경로가 이미 발견됨)
        if distances[current_node] < current_dist:
            continue
            
        for neighbor, weight in graph[current_node].items():
            distance = current_dist + weight
            
            # 더 짧은 경로 발견 시 업데이트 (Relaxation)
            if distance < distances[neighbor]:
                distances[neighbor] = distance
                heapq.heappush(pq, (distance, neighbor))
                
    return distances
```

## ✅ 이번 주 과제
1. `Graph` 클래스에 `remove_edge` 메서드 추가하기
2. `bfs` 함수를 수정하여 시작 노드로부터 각 노드까지의 '최단 거리(간선 개수)'를 반환하도록 만들기
3. `dijkstra` 알고리즘을 직접 타이핑하고, 예제 그래프를 만들어 최단 경로가 맞는지 검증하기
