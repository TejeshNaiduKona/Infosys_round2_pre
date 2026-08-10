# Infosys Offline Coding Exam — DSA Guide (Basic → Advanced)
Python templates + patterns to clear hidden test cases

---

## 0. How Infosys OA hidden tests actually trip people up
Before topics — this is what separates "sample tests pass, hidden tests fail":
- **Empty input** (`n=0`, empty string/array)
- **Single element**
- **All same elements** (e.g. all zeros, all duplicates)
- **Negative numbers** (sum/product/min problems)
- **Very large n** (10^5–10^6) → brute force O(n²) TLEs, need O(n log n) or O(n)
- **Integer overflow-style large sums** (Python is fine, but watch for wrong dtype assumptions if using numpy)
- **Off-by-one on ranges** — always test with n=1 and n=2 manually
- **Exact I/O format** — Infosys often gives strict input parsing (space/newline separated). Read input with `sys.stdin` for speed, and match output format EXACTLY (no extra spaces/newlines, correct capitalization like "YES"/"Yes")

Fast I/O template:
```python
import sys
input = sys.stdin.readline
data = sys.stdin.read().split()
```

---

## 1. ARRAYS

### Basics
- **Traversal / Search**: linear scan O(n); binary search O(log n) — array must be sorted
- **Max/Min**: single pass, track running best
- **Frequency count**: `collections.Counter(arr)` — O(n)

### Binary Search (must-know, appears constantly)
```python
def binary_search(arr, target):
    lo, hi = 0, len(arr) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1
```
Variant — **binary search on answer** (used in "minimize max / maximize min" problems, e.g. allocate books, painter's partition, ship packages):
```python
def can_do(x):
    # feasibility check for candidate answer x
    ...

lo, hi = min_possible, max_possible
while lo < hi:
    mid = (lo + hi) // 2
    if can_do(mid):
        hi = mid
    else:
        lo = mid + 1
# lo is the answer
```

### Prefix Sum
```python
prefix = [0] * (n + 1)
for i in range(n):
    prefix[i + 1] = prefix[i] + arr[i]
# sum of arr[l..r] inclusive = prefix[r+1] - prefix[l]
```
Use for: range sum queries, subarray sum == K, equilibrium index.

### Sliding Window
**Fixed size k:**
```python
window_sum = sum(arr[:k])
best = window_sum
for i in range(k, n):
    window_sum += arr[i] - arr[i - k]
    best = max(best, window_sum)
```
**Variable size (longest subarray meeting a condition):**
```python
left = 0
window_val = 0
best = 0
for right in range(n):
    window_val += arr[right]
    while window_val > target:   # shrink condition
        window_val -= arr[left]
        left += 1
    best = max(best, right - left + 1)
```

### Subarray Problems (core Infosys favorites)
- **Kadane's Algorithm** — max subarray sum:
```python
def kadane(arr):
    best = cur = arr[0]
    for x in arr[1:]:
        cur = max(x, cur + x)
        best = max(best, cur)
    return best
```
- **Count subarrays with given sum** (works with negatives, uses prefix sum + hashmap):
```python
def count_subarrays_sum_k(arr, k):
    from collections import defaultdict
    freq = defaultdict(int)
    freq[0] = 1
    s = 0
    count = 0
    for x in arr:
        s += x
        count += freq[s - k]
        freq[s] += 1
    return count
```
- **Longest subarray with sum K** (same idea, store first index of each prefix sum):
```python
def longest_subarray_sum_k(arr, k):
    first_seen = {0: -1}
    s = 0
    best = 0
    for i, x in enumerate(arr):
        s += x
        if (s - k) in first_seen:
            best = max(best, i - first_seen[s - k])
        if s not in first_seen:
            first_seen[s] = i
    return best
```

### Two Pointers
- Pair sum in sorted array, container with most water, 3-sum (sort + two pointers), Dutch national flag (0/1/2 sort), remove duplicates in-place.
```python
def two_sum_sorted(arr, target):
    l, r = 0, len(arr) - 1
    while l < r:
        s = arr[l] + arr[r]
        if s == target:
            return (l, r)
        elif s < target:
            l += 1
        else:
            r -= 1
    return None
```

### Sorting — know complexities, not just `.sort()`
| Algorithm | Time | Space | Notes |
|---|---|---|---|
| Bubble/Selection/Insertion | O(n²) | O(1) | only for small n or "implement sort" questions |
| Merge Sort | O(n log n) | O(n) | stable, good for linked lists / counting inversions |
| Quick Sort | O(n log n) avg, O(n²) worst | O(log n) | in-place |
| Counting Sort | O(n+k) | O(k) | when range of values is small |

**Cyclic sort** (when values are in range 1..n — used for "find missing/duplicate number"):
```python
def cyclic_sort(arr):
    i = 0
    while i < len(arr):
        correct = arr[i] - 1
        if arr[i] != arr[correct]:
            arr[i], arr[correct] = arr[correct], arr[i]
        else:
            i += 1
    return arr
```

### Matrix
- Row/column traversal, spiral traversal, rotate matrix in-place (transpose + reverse rows), search in row-wise/column-wise sorted matrix (start top-right corner).

---

## 2. STRINGS
- Reverse, palindrome check (two pointers)
- Anagram check → sort both or Counter comparison, O(n log n) or O(n)
- Character frequency → `Counter`
- **Longest substring without repeating characters** (sliding window + set/dict):
```python
def longest_unique_substr(s):
    seen = {}
    left = best = 0
    for right, ch in enumerate(s):
        if ch in seen and seen[ch] >= left:
            left = seen[ch] + 1
        seen[ch] = right
        best = max(best, right - left + 1)
    return best
```
- **String matching**: KMP (O(n+m)) for pattern search if brute force is too slow; know it exists even if you use `in` operator when allowed.
- Valid parentheses → stack.
- String to integer parsing edge cases (leading zeros, signs, overflow-style limits if asked to simulate 32-bit int).

---

## 3. LINKED LIST
```python
class Node:
    def __init__(self, val):
        self.val = val
        self.next = None
```
- Reverse a linked list (iterative, O(1) space):
```python
def reverse(head):
    prev = None
    while head:
        nxt = head.next
        head.next = prev
        prev = head
        head = nxt
    return prev
```
- **Detect cycle** — Floyd's slow/fast pointer:
```python
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    return False
```
- Find middle node (slow/fast pointer), merge two sorted lists, remove Nth node from end (two pointers with gap N).

---

## 4. STACK & QUEUE
- Valid parentheses, next greater element (monotonic stack), min stack (auxiliary stack), evaluate postfix/infix expressions.
- **Next Greater Element** (monotonic stack pattern — common in Infosys):
```python
def next_greater(arr):
    res = [-1] * len(arr)
    stack = []  # indices
    for i, x in enumerate(arr):
        while stack and arr[stack[-1]] < x:
            res[stack.pop()] = x
        stack.append(i)
    return res
```
- Queue via two stacks, circular queue, deque for sliding window maximum.

---

## 5. RECURSION & BACKTRACKING
- Base case first, always. Trace with n=1/n=2 to validate.
- Factorial, Fibonacci (then optimize with memoization — this is the DP bridge).
- **Backtracking template:**
```python
def backtrack(path, choices):
    if is_solution(path):
        results.append(path[:])
        return
    for choice in choices:
        if not valid(choice, path):
            continue
        path.append(choice)
        backtrack(path, next_choices)
        path.pop()  # undo
```
- Common problems: permutations, subsets/power set, N-Queens, Sudoku solver, combination sum.

---

## 6. TREES
```python
class TreeNode:
    def __init__(self, val):
        self.val = val
        self.left = self.right = None
```
- Traversals: inorder/preorder/postorder (recursive + iterative with stack), level order (BFS with queue).
- **BST operations**: search/insert O(log n) avg, O(n) worst (skewed).
- Height/depth, diameter, check balanced, check if BST (pass min/max bounds down), LCA (lowest common ancestor).
```python
def lca(root, p, q):
    if not root or root.val == p or root.val == q:
        return root
    left = lca(root.left, p, q)
    right = lca(root.right, p, q)
    if left and right:
        return root
    return left or right
```
- Level order / BFS:
```python
from collections import deque
def level_order(root):
    if not root:
        return []
    result, q = [], deque([root])
    while q:
        level = []
        for _ in range(len(q)):
            node = q.popleft()
            level.append(node.val)
            if node.left: q.append(node.left)
            if node.right: q.append(node.right)
        result.append(level)
    return result
```

---

## 7. GRAPHS
- Represent as adjacency list: `graph = defaultdict(list)`
- **BFS** (shortest path, unweighted):
```python
def bfs(graph, start):
    visited = {start}
    q = deque([start])
    order = []
    while q:
        node = q.popleft()
        order.append(node)
        for nbr in graph[node]:
            if nbr not in visited:
                visited.add(nbr)
                q.append(nbr)
    return order
```
- **DFS** (recursive or with explicit stack) — connected components, cycle detection.
- Topological sort (Kahn's BFS algorithm using in-degree, or DFS-based).
- Dijkstra's (shortest path, weighted, non-negative) using heapq.
- Union-Find / Disjoint Set — for connected components, cycle detection in undirected graphs, Kruskal's MST.
```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    def union(self, a, b):
        ra, rb = self.find(a), self.find(b)
        if ra == rb:
            return False
        self.parent[ra] = rb
        return True
```

---

## 8. DYNAMIC PROGRAMMING (biggest score differentiator)
**Recognize DP**: "count ways", "min/max cost", "can you reach/partition", overlapping subproblems + optimal substructure.

**Approach**: recursion → add memo (top-down) → convert to tabulation (bottom-up) → optimize space.

- **Fibonacci-style (1D DP)**:
```python
def climb_stairs(n):
    if n <= 2:
        return n
    dp = [0] * (n + 1)
    dp[1], dp[2] = 1, 2
    for i in range(3, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]
```
- **0/1 Knapsack** (classic template — appears in many disguises):
```python
def knapsack(weights, values, W):
    n = len(weights)
    dp = [[0]*(W+1) for _ in range(n+1)]
    for i in range(1, n+1):
        for w in range(W+1):
            dp[i][w] = dp[i-1][w]
            if weights[i-1] <= w:
                dp[i][w] = max(dp[i][w], dp[i-1][w-weights[i-1]] + values[i-1])
    return dp[n][W]
```
- **Longest Common Subsequence (2D DP)**:
```python
def lcs(a, b):
    m, n = len(a), len(b)
    dp = [[0]*(n+1) for _ in range(m+1)]
    for i in range(1, m+1):
        for j in range(1, n+1):
            if a[i-1] == b[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[m][n]
```
- Also know: coin change (min coins / count ways), longest increasing subsequence (O(n log n) with binary search), edit distance, matrix path sum/min path, house robber.

---

## 9. GREEDY
- Activity selection (sort by end time), fractional knapsack, job sequencing with deadlines, minimum platforms/meeting rooms (sort start/end, sweep).
- Rule of thumb: if a locally optimal choice provably leads to a globally optimal one (prove via exchange argument or just pattern-match to known greedy problems) — otherwise it's DP.

---

## 10. HASHING
- `dict`/`Counter`/`set` for O(1) avg lookup — turns O(n²) brute force into O(n).
- Two Sum, group anagrams, first non-repeating character, subarray sum = K (see Arrays section), detect duplicates.

---

## 11. BIT MANIPULATION (occasionally shows up)
- `x & 1` → odd/even, `x >> 1` → divide by 2, `x & (x-1)` → clears lowest set bit (useful for counting set bits / power-of-2 check).
- XOR trick: find the single non-repeating number in an array where every other appears twice → `reduce(xor, arr)`.

---

## 12. PATTERN CHEAT SHEET — map problem phrasing to technique

| Phrase in question | Likely technique |
|---|---|
| "contiguous subarray with max/min sum" | Kadane's |
| "subarray/substring with sum/condition == K" | Prefix sum + hashmap, or sliding window |
| "longest substring without repeating" | Sliding window |
| "find pair/triplet with sum" | Two pointers (sort first) or hashing |
| "kth largest/smallest" | Heap, or quickselect |
| "minimum number of X to achieve Y" | DP or Greedy |
| "number of ways to..." | DP (counting) |
| "can you partition/reach target" | DP (boolean) |
| "next greater/smaller element" | Monotonic stack |
| "shortest path unweighted" | BFS |
| "shortest path weighted" | Dijkstra |
| "connected components / cycle" | DFS/BFS or Union-Find |
| "all permutations/combinations/subsets" | Backtracking |
| "sorted array, find element/boundary" | Binary search |
| "minimize the maximum" / "maximize the minimum" | Binary search on answer |
| "missing number / duplicate in range 1..n" | Cyclic sort or XOR |

---

## 13. FINAL CHECKLIST BEFORE SUBMITTING EACH SOLUTION
1. Handle `n == 0` and `n == 1` explicitly if not covered naturally.
2. Re-read output format — exact case, spacing, single vs multi-line.
3. State time complexity to yourself — if O(n²) and n can be 10^5+, rewrite.
4. Test with a case containing negative numbers if the domain allows them.
5. Test with all-duplicate input.
6. For recursion, confirm base case triggers correctly for smallest input.

---

*Built from the visible outline of your linked doc (Array topics: searching, sorting, prefix sum, sliding window, subarray problems, Kadane's) plus the standard Infosys OA syllabus. Send me the rest of the doc's contents (or share access) and I'll extend this with any additional topics it lists.*
