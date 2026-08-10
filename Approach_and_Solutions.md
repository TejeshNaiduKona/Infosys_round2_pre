# DSA Master — Approaches & Solutions

Companion to `Problem_Set_By_Pattern.md`. Each section: approach reasoning + full code for the ★ flagship problem, then quick approach notes for the sibling problems (solve those yourself first).

---

## 1. Kadane's — LC 53: Maximum Subarray
**Approach**: At each index, decide: extend the previous subarray, or start fresh here. If `cur_sum` goes negative, it can only drag future sums down, so reset it to the current element.
```python
def maxSubArray(nums):
    best = cur = nums[0]
    for x in nums[1:]:
        cur = max(x, cur + x)
        best = max(best, cur)
    return best
```
- *152 Max Product Subarray*: track both running max AND min (a negative number can flip min→max), reset both at negatives.
- *918 Circular Subarray*: answer is `max(Kadane(nums), total_sum - min_subarray_sum)` — handles wraparound.

---

## 2. Prefix Sum + Hashmap — LC 560: Subarray Sum Equals K
**Approach**: Running prefix sum `s`. If `s - k` was seen before at some count, that many subarrays ending here sum to k. Store frequency of each prefix sum in a hashmap.
```python
from collections import defaultdict
def subarraySum(nums, k):
    freq = defaultdict(int)
    freq[0] = 1
    s = count = 0
    for x in nums:
        s += x
        count += freq[s - k]
        freq[s] += 1
    return count
```
- *523 divisible by k*: use `s % k` as the hashmap key instead of raw sum.
- *325 max length sum = k*: store first index of each prefix sum, not frequency.

---

## 3. Sliding Window — LC 3: Longest Substring Without Repeating Characters
**Approach**: Expand `right` each step; if the char is already in the window (and its last position is inside `[left, right]`), jump `left` past it. Track max window width.
```python
def lengthOfLongestSubstring(s):
    seen = {}
    left = best = 0
    for right, ch in enumerate(s):
        if ch in seen and seen[ch] >= left:
            left = seen[ch] + 1
        seen[ch] = right
        best = max(best, right - left + 1)
    return best
```
- *209 min size subarray sum ≥ target*: shrink window while sum ≥ target, track min width.
- *76 min window substring*: expand until window has all target chars, then shrink; use two counters.
- *438 anagrams*: fixed-size window + character count comparison.

---

## 4. Two Pointers — LC 15: 3Sum
**Approach**: Sort array. Fix one element, then use two pointers on the rest (sorted) to find pairs summing to `-fixed`. Skip duplicates at every level to avoid repeat triplets.
```python
def threeSum(nums):
    nums.sort()
    res = []
    for i in range(len(nums) - 2):
        if i > 0 and nums[i] == nums[i-1]:
            continue
        l, r = i + 1, len(nums) - 1
        while l < r:
            s = nums[i] + nums[l] + nums[r]
            if s < 0:
                l += 1
            elif s > 0:
                r -= 1
            else:
                res.append([nums[i], nums[l], nums[r]])
                while l < r and nums[l] == nums[l+1]: l += 1
                while l < r and nums[r] == nums[r-1]: r -= 1
                l += 1; r -= 1
    return res
```
- *11 container with most water*: move the pointer at the shorter line inward (moving the taller one can't help).
- *75 sort colors*: three pointers (low/mid/high), one pass, no counting needed.

---

## 5. Binary Search — LC 704: Binary Search
```python
def search(nums, target):
    lo, hi = 0, len(nums) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1
```
- *33 rotated sorted array*: at each mid, figure out which half is properly sorted, then check if target lies in that half's range.
- *74 search 2D matrix*: treat the matrix as one flattened sorted array using index math `mid // cols, mid % cols`.

---

## 6. Binary Search on Answer — LC 875: Koko Eating Bananas
**Approach**: The answer (min eating speed) is monotonic — if speed `x` finishes in time, any speed `> x` also finishes in time. Binary search over possible speeds, using a feasibility check as the test.
```python
import math
def minEatingSpeed(piles, h):
    lo, hi = 1, max(piles)
    while lo < hi:
        mid = (lo + hi) // 2
        hours = sum(math.ceil(p / mid) for p in piles)
        if hours <= h:
            hi = mid
        else:
            lo = mid + 1
    return lo
```
- *1011 ship capacity*: same shape — binary search capacity, feasibility = can you ship all packages in D days at that capacity.
- *Allocate pages / painter's partition*: binary search the "max pages one person gets" / "max time one painter takes."

---

## 7. Cyclic Sort / XOR — LC 268: Missing Number
**Approach (XOR)**: XOR all indices 0..n and all array values together — everything present cancels, leaving the missing number.
```python
def missingNumber(nums):
    n = len(nums)
    result = n
    for i, x in enumerate(nums):
        result ^= i ^ x
    return result
```
- *448 disappeared numbers*: cyclic sort — place each value at `index = value - 1`, then scan for mismatches.
- *287 duplicate number*: treat array as a linked list (`nums[i]` points to `nums[nums[i]]`) and run Floyd's cycle detection.
- *41 first missing positive*: cyclic sort, then first index `i` where `nums[i] != i+1` gives the answer.

---

## 8. Sorting-based — LC 56: Merge Intervals
**Approach**: Sort intervals by start time. Walk through; if current interval overlaps the last merged one (`start <= last_end`), merge by extending the end; otherwise start a new group.
```python
def merge(intervals):
    intervals.sort(key=lambda x: x[0])
    merged = [intervals[0]]
    for start, end in intervals[1:]:
        if start <= merged[-1][1]:
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])
    return merged
```
- *Count Inversions*: modify merge sort — during the merge step, count how many elements from the right half jump ahead of left half elements.

---

## 9. Hashing — LC 1: Two Sum
```python
def twoSum(nums, target):
    seen = {}
    for i, x in enumerate(nums):
        if target - x in seen:
            return [seen[target - x], i]
        seen[x] = i
```
- *49 group anagrams*: key = `''.join(sorted(word))`, group words under that key.
- *128 longest consecutive sequence*: put all numbers in a set; only start counting a sequence from numbers whose `x-1` is NOT in the set (O(n) total, not O(n log n)).

---

## 10. Stack (Monotonic) — LC 20: Valid Parentheses
```python
def isValid(s):
    stack = []
    pairs = {')': '(', ']': '[', '}': '{'}
    for ch in s:
        if ch in pairs:
            if not stack or stack.pop() != pairs[ch]:
                return False
        else:
            stack.append(ch)
    return not stack
```
- *496/739 next greater element / daily temperatures*: monotonic decreasing stack of indices; pop and resolve when a bigger element is found.
- *84 largest rectangle in histogram*: monotonic increasing stack of indices; when a smaller bar appears, pop and compute area with popped bar as height.

---

## 11. Deque — LC 239: Sliding Window Maximum
**Approach**: Maintain a deque of indices with decreasing values. Front is always the max of the current window. Pop from back while smaller than incoming element; pop from front if it's out of window range.
```python
from collections import deque
def maxSlidingWindow(nums, k):
    dq = deque()
    res = []
    for i, x in enumerate(nums):
        while dq and nums[dq[-1]] < x:
            dq.pop()
        dq.append(i)
        if dq[0] <= i - k:
            dq.popleft()
        if i >= k - 1:
            res.append(nums[dq[0]])
    return res
```

---

## 12. Linked List — LC 206: Reverse Linked List
```python
def reverseList(head):
    prev = None
    while head:
        nxt = head.next
        head.next = prev
        prev = head
        head = nxt
    return prev
```
- *141/142 cycle detection*: Floyd's slow/fast pointer; for cycle start (142), after they meet, reset one pointer to head and move both one step at a time — they meet again exactly at the cycle start (provable with the math of the two segment lengths).
- *19 remove Nth from end*: two pointers with a gap of N; when the fast one hits the end, slow is right before the node to remove.

---

## 13. Backtracking — LC 46: Permutations
```python
def permute(nums):
    res = []
    def backtrack(path, remaining):
        if not remaining:
            res.append(path[:])
            return
        for i in range(len(remaining)):
            backtrack(path + [remaining[i]], remaining[:i] + remaining[i+1:])
    backtrack([], nums)
    return res
```
- *78 subsets*: at each element, branch into "include" and "exclude" — 2ⁿ leaves.
- *39 combination sum*: same as subsets but allow reusing the same element (don't advance index on include).
- *51 N-Queens*: place one queen per row, backtrack when a column/diagonal conflict is detected.

---

## 14. Trees — LC 102: Binary Tree Level Order Traversal
```python
from collections import deque
def levelOrder(root):
    if not root:
        return []
    res, q = [], deque([root])
    while q:
        level = []
        for _ in range(len(q)):
            node = q.popleft()
            level.append(node.val)
            if node.left: q.append(node.left)
            if node.right: q.append(node.right)
        res.append(level)
    return res
```
- *543 diameter*: recursive height function that also updates a global "max diameter seen so far" using left_height + right_height at every node.
- *98 validate BST*: recursive, pass down `(low, high)` bounds, check `low < node.val < high`.
- *236 LCA*: post-order recursion — if both left and right subtree calls return non-null, current node is the LCA.

---

## 15. Graphs BFS/DFS — LC 200: Number of Islands
```python
def numIslands(grid):
    if not grid: return 0
    rows, cols = len(grid), len(grid[0])
    def dfs(r, c):
        if r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] != '1':
            return
        grid[r][c] = '0'  # mark visited
        for dr, dc in [(-1,0),(1,0),(0,-1),(0,1)]:
            dfs(r+dr, c+dc)
    count = 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1':
                count += 1
                dfs(r, c)
    return count
```
- *994 rotting oranges*: multi-source BFS — start with ALL initially-rotten cells in the queue at once, track minutes via BFS layers.
- *133 clone graph*: DFS/BFS with a hashmap `original_node -> cloned_node` to avoid infinite loops and duplicate clones.

---

## 16. Graphs Shortest Path — LC 743: Network Delay Time (Dijkstra)
```python
import heapq
from collections import defaultdict
def networkDelayTime(times, n, k):
    graph = defaultdict(list)
    for u, v, w in times:
        graph[u].append((v, w))
    dist = {}
    heap = [(0, k)]
    while heap:
        d, node = heapq.heappop(heap)
        if node in dist:
            continue
        dist[node] = d
        for nbr, w in graph[node]:
            if nbr not in dist:
                heapq.heappush(heap, (d + w, nbr))
    return max(dist.values()) if len(dist) == n else -1
```

---

## 17. Union-Find / Topo Sort — LC 207: Course Schedule
**Approach**: Build a graph of prerequisites, compute in-degree of each course, repeatedly process courses with in-degree 0 (Kahn's algorithm). If all courses get processed, no cycle exists — order is possible.
```python
from collections import deque, defaultdict
def canFinish(numCourses, prerequisites):
    graph = defaultdict(list)
    indegree = [0] * numCourses
    for course, pre in prerequisites:
        graph[pre].append(course)
        indegree[course] += 1
    q = deque([c for c in range(numCourses) if indegree[c] == 0])
    visited = 0
    while q:
        node = q.popleft()
        visited += 1
        for nxt in graph[node]:
            indegree[nxt] -= 1
            if indegree[nxt] == 0:
                q.append(nxt)
    return visited == numCourses
```
- *547 provinces / 684 redundant connection*: Union-Find — union each edge's endpoints; a redundant connection is the edge where both endpoints already share a root.

---

## 18. DP 1D — LC 70: Climbing Stairs
```python
def climbStairs(n):
    if n <= 2: return n
    a, b = 1, 2
    for _ in range(3, n + 1):
        a, b = b, a + b
    return b
```
- *198 house robber*: `dp[i] = max(dp[i-1], dp[i-2] + nums[i])` — either skip this house or rob it (and add best from two houses back).
- *322 coin change*: `dp[amount] = min(dp[amount - coin] + 1)` over all coins — bottom-up, `dp[0] = 0`.
- *300 LIS*: `dp[i]` = length of longest increasing subsequence ending at i; O(n²) baseline, O(n log n) with binary search + patience sorting.

---

## 19. DP 2D — LC 1143: Longest Common Subsequence
```python
def longestCommonSubsequence(text1, text2):
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[m][n]
```
- *72 edit distance*: same table shape, but three operations (insert/delete/replace) instead of match/skip.
- *416 partition equal subset sum*: 0/1 knapsack where "weight" = "value" = number, target = totalSum/2, answer = is target reachable.

---

## 20. Greedy — LC 435: Non-overlapping Intervals
**Approach**: Sort by end time. Greedily keep an interval if it starts after the last kept interval's end; otherwise it must be removed (it overlaps).
```python
def eraseOverlapIntervals(intervals):
    intervals.sort(key=lambda x: x[1])
    count = 0
    last_end = float('-inf')
    for start, end in intervals:
        if start >= last_end:
            last_end = end
        else:
            count += 1
    return count
```
- *253 meeting rooms II*: sort starts and ends separately, sweep with two pointers, track max concurrent overlap.
- *Fractional knapsack*: sort by value/weight ratio descending, take as much of each item as possible until capacity fills.

---

## 21. Bit Manipulation — LC 136: Single Number
```python
def singleNumber(nums):
    result = 0
    for x in nums:
        result ^= x
    return result
```
- *191 number of 1 bits*: loop `n & 1`, then `n >>= 1`, or use `n & (n-1)` to strip lowest set bit each iteration (faster).
- *231 power of two*: `n > 0 and (n & (n-1)) == 0`.

---

## 22. String — LC 5: Longest Palindromic Substring
**Approach**: Expand around every possible center (odd-length centers at each index, even-length centers between indices). Track the widest valid palindrome found.
```python
def longestPalindrome(s):
    def expand(l, r):
        while l >= 0 and r < len(s) and s[l] == s[r]:
            l -= 1; r += 1
        return s[l+1:r]
    result = ""
    for i in range(len(s)):
        odd = expand(i, i)
        even = expand(i, i + 1)
        result = max(result, odd, even, key=len)
    return result
```

---

## 23. Matrix — LC 54: Spiral Matrix
```python
def spiralOrder(matrix):
    res = []
    while matrix:
        res += matrix.pop(0)                 # top row
        if matrix and matrix[0]:
            for row in matrix:
                res.append(row.pop())         # right column
        if matrix:
            res += matrix.pop()[::-1]         # bottom row reversed
        if matrix and matrix[0]:
            for row in matrix[::-1]:
                res.append(row.pop(0))        # left column
    return res
```

---

## Study rhythm suggestion
- Day 1–2 of each pattern: read approach, type out the flagship code yourself from memory (not copy-paste).
- Day 2–3: solve the sibling problems cold, using only the one-line approach hint.
- Every 4th day: re-solve one flagship problem from an earlier pattern to keep it fresh before the exam.
