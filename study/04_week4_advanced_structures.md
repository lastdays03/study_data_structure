# 4주차: 해시 테이블, 정렬 알고리즘, 고급 자료구조

## 🎯 학습 목표
- **심층 이해**: 해시 충돌의 원리와 해결법, 정렬 알고리즘의 안정성과 효율성을 이해합니다.
- **구현 능력**: 해시 테이블과 고급 정렬 알고리즘을 직접 구현합니다.
- **활용 능력**: 상황에 맞는 적절한 정렬 알고리즘과 자료구조를 선택합니다.

## 📚 핵심 개념 심층 분석

### 1. 해시 테이블 (Hash Table)

#### 🕊️ 비둘기집 원리와 충돌(Collision)
- **해시 함수**: 임의의 길이의 데이터를 고정된 길이의 인덱스로 매핑합니다.
- **충돌 불가피성**: 입력 가능한 데이터의 수는 무한하지만, 해시 테이블의 크기(버킷 수)는 유한합니다. 비둘기집 원리에 의해 서로 다른 두 데이터가 같은 인덱스를 가리키는 **충돌**은 반드시 발생합니다.
- **Load Factor (적재율)**: `데이터 개수 / 테이블 크기`. 적재율이 높아지면 충돌 확률이 급격히 증가하여 성능이 떨어집니다. 보통 0.7 이상이 되면 테이블 크기를 늘리고 재해싱(Rehashing)을 합니다.

### 2. 정렬 알고리즘의 안정성 (Stability)

#### ⚖️ Stable vs Unstable
- **안정 정렬 (Stable Sort)**: 값이 같은 요소들의 **상대적인 순서가 유지**되는 정렬입니다.
    - 예: (이름, 점수) 쌍을 점수 순으로 정렬할 때, 점수가 같으면 원래 목록에 있던 이름 순서가 유지됩니다.
    - 종류: 병합 정렬, 버블 정렬, 삽입 정렬.
- **불안정 정렬 (Unstable Sort)**: 상대적인 순서가 뒤바뀔 수 있습니다.
    - 종류: 퀵 정렬, 힙 정렬, 선택 정렬.
- **활용**: 엑셀에서 '이름 순 정렬' 후 '부서 순 정렬'을 했을 때, 같은 부서 내에서 이름 순이 유지되려면 안정 정렬이어야 합니다.

### 3. 퀵 정렬 (Quick Sort)

#### ⚡ 피벗(Pivot)의 중요성
- **평균 O(n log n)**: 피벗이 데이터를 절반씩 잘 나누면 빠릅니다.
- **최악 O(n^2)**: 이미 정렬된 배열에서 피벗을 항상 첫 번째 요소로 잡으면, 분할이 1:N-1로 일어나 비효율적입니다.
- **해결책**: 피벗을 랜덤하게 잡거나, `Median of Three`(첫값, 중간값, 끝값 중 중간값)를 피벗으로 사용하는 방법을 씁니다.

---

## 💻 파이썬 구현 가이드

### 1. 해시 테이블 (Chaining 방식)
```python
class HashTable:
    def __init__(self, size=10):
        self.size = size
        self.table = [[] for _ in range(self.size)] # 각 버킷은 리스트(체이닝)

    def _hash(self, key):
        return hash(key) % self.size

    def put(self, key, value):
        idx = self._hash(key)
        bucket = self.table[idx]
        
        # 이미 키가 존재하면 업데이트
        for i, (k, v) in enumerate(bucket):
            if k == key:
                bucket[i] = (key, value)
                return
        
        # 없으면 추가
        bucket.append((key, value))

    def get(self, key):
        idx = self._hash(key)
        bucket = self.table[idx]
        
        for k, v in bucket:
            if k == key:
                return v
        return None # Key not found

    def remove(self, key):
        # 구현 과제: 키를 찾아 삭제
        pass
```

### 2. 병합 정렬 (Merge Sort) - 안정 정렬
```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    
    return merge(left, right)

def merge(left, right):
    result = []
    i = j = 0
    
    while i < len(left) and j < len(right):
        if left[i] < right[j]: # 작거나 같을 때 왼쪽 먼저 -> 안정성 유지
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
            
    result.extend(left[i:])
    result.extend(right[j:])
    return result
```

### 3. 퀵 정렬 (Quick Sort) - 불안정 정렬
```python
def quick_sort(arr):
    if len(arr) <= 1:
        return arr
    
    pivot = arr[len(arr) // 2]
    lesser_arr, equal_arr, greater_arr = [], [], []
    
    for num in arr:
        if num < pivot:
            lesser_arr.append(num)
        elif num > pivot:
            greater_arr.append(num)
        else:
            equal_arr.append(num)
            
    return quick_sort(lesser_arr) + equal_arr + quick_sort(greater_arr)
```

### 4. 트라이 (Trie) - 문자열 검색
```python
class TrieNode:
    def __init__(self):
        self.children = {} # {char: TrieNode}
        self.is_end_of_word = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_end_of_word = True

    def search(self, word):
        # 구현 과제
        pass

    def starts_with(self, prefix):
        # 구현 과제
        pass
```

## ✅ 이번 주 과제
1. `HashTable`의 `remove` 메서드 구현하기
2. `Quick Sort`를 In-place 방식(추가 리스트 없이 인덱스 조작만으로)으로 구현해보기
3. `Trie`의 `search`와 `starts_with` 메서드 완성하기
