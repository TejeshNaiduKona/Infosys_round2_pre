# Algorithm Selection Guide — What to Use, Where, and WHY

For each problem type: the technique, its complexity, the brute-force alternative, and the *reasoning* for why it wins. This is the "why" layer behind the pattern cheat sheet.

---

## 1. ARRAY / SUBARRAY PROBLEMS

| Problem statement | Algorithm | Complexity | Why it's best |
|---|---|---|---|
| "Max sum of a contiguous subarray" | **Kadane's Algorithm** | O(n) | Brute force checks all O(n²) subarrays. Kadane exploits the fact that a negative running sum can never help a future subarray — so you reset it. One pass, one comparison per element, no extra space. |
| "Count/find subarray with sum = K" (array can have negatives) | **Prefix sum + hashmap** | O(n) | Sliding window fails here because negatives break the "shrink when too big" monotonic logic. Prefix sum turns "does a subarray summing to K exist ending here" into "have I seen prefix_sum - K before" — an O(1) hashmap lookup instead of O(n) re-scanning. |
| "Longest/shortest subarray meeting a sum/count condition" (all positive/non-negative) | **Sliding window** | O(n) | When all elements are non-negative, the window sum only grows as you extend right and only shrinks as you contract left — this monotonicity lets each pointer move forward only, giving O(n) total instead of recomputing sums from scratch (brute force O(n²) or O(n³)). |
| "Find missing number / duplicate in range 1..n" | **Cyclic sort** or **XOR** | O(n), O(1) space | Since values are constrained to exactly the range of indices, each number has a "correct home." Placing each number at its index in one pass avoids sorting (O(n log n)) or hashing (O(n) extra space). XOR works because `a ^ a = 0`, so pairing cancels everything except the odd one out — no extra space at all. |
| "Kth largest/smallest element" | **Heap (heapq)**, or **Quickselect** | O(n log k) heap / O(n) avg quickselect | Full sort is O(n log n) but you only need one element, not the whole order. A size-k heap tracks just the top k candidates. Quickselect (partition step of quicksort) discards half the irrelevant elements each round, like binary search over the array, averaging O(n). |
| "Rotate array by k" | **Reverse in 3 steps** (whole, then two parts) | O(n) time, O(1) space | Naive rotation with an extra array costs O(n) space. Triple-reverse achieves the same result in-place — an elegant trick that avoids the temp array entirely. |

---

## 2. SEARCHING

| Problem statement | Algorithm | Complexity | Why it's best |
|---|---|---|---|
| "Find element/index in a **sorted** array" | **Binary Search** | O(log n) | Sorted order lets you eliminate half the search space every comparison, versus linear scan checking every element. This is the single biggest complexity jump in basic DSA (10⁵ elements: ~17 comparisons vs 100,000). |
| "Minimize the maximum" / "maximize the minimum" (e.g., allocate books, ship capacity, painter's partition) | **Binary search on the answer** | O(n log(range)) | You're not searching the array — you're searching the space of *possible answers*, which is monotonic (if capacity X works, every capacity > X also works). That monotonicity is what makes binary search valid here even though there's no sorted array in sight. Brute-force trying every capacity is O(n × range). |
| "Search in row-wise and column-wise sorted matrix" | **Start from top-right (or bottom-left) corner** | O(m+n) | From the top-right corner, moving left decreases value, moving down increases it — giving a single directional decision at each step, unlike binary search per row (O(m log n)) or full scan (O(mn)). |

---

## 3. TWO POINTERS

| Problem statement | Algorithm | Complexity | Why it's best |
|---|---|---|---|
| "Pair with given sum in sorted array" | **Two pointers (ends moving inward)** | O(n) | Sorted order means: if current sum is too small, only moving the left pointer right can increase it; if too big, only moving right pointer left can decrease it. This deterministic direction avoids checking all O(n²) pairs. |
| "3Sum / 4Sum" | **Sort + fix one/two elements + two pointers** | O(n²) for 3Sum | Fixing the outer element(s) and reducing the inner search to two-pointer turns an O(n³) brute force into O(n²) — sorting once (O(n log n)) is cheap compared to the savings on the search itself. |
| "Remove duplicates in-place from sorted array" | **Slow/fast pointer** | O(n), O(1) space | Since duplicates are adjacent in sorted order, one pointer can track the "write position" while another scans ahead — no extra array needed, unlike a set-based dedup which costs O(n) space. |

---

## 4. HASHING

| Problem statement | Algorithm | Complexity | Why it's best |
|---|---|---|---|
| "Two Sum (unsorted array)" | **Hashmap (value → index)** | O(n) | Sorting first would cost O(n log n) and lose original indices. A hashmap lets you check "does target - x exist" in O(1) per element, so one pass suffices. |
| "Group anagrams" | **Hashmap keyed by sorted string (or char count tuple)** | O(n·k log k) | Anagrams share a canonical form (sorted letters). Hashing on that canonical form groups them in one pass instead of comparing every pair of strings O(n²). |
| "First non-repeating character" | **Hashmap/Counter for frequency, then re-scan** | O(n) | Counting frequencies first turns "is this character unique" into an O(1) lookup on the second pass, rather than re-scanning the whole string for each character (O(n²)). |

---

## 5. STACK-BASED

| Problem statement | Algorithm | Complexity | Why it's best |
|---|---|---|---|
| "Valid parentheses / balanced brackets" | **Stack** | O(n) | Brackets must close in LIFO order — the most recently opened bracket must close first. A stack directly models this; nothing else naturally captures "last opened, first closed." |
| "Next greater/smaller element for every array element" | **Monotonic stack** | O(n) | Naive approach checks every element against every element to its right = O(n²). A monotonic stack keeps only "candidates that haven't found their answer yet," and each element is pushed/popped at most once — so total work across the whole array is O(n), not O(n) per element. |
| "Evaluate expression / postfix" | **Stack** | O(n) | Operators need their most recent operands — again a LIFO dependency that a stack captures directly. |

---

## 6. LINKED LIST

| Problem statement | Algorithm | Complexity | Why it's best |
|---|---|---|---|
| "Detect a cycle" | **Floyd's slow/fast pointer** | O(n) time, O(1) space | Using a hashset to track visited nodes also works but costs O(n) space. Two pointers moving at different speeds will *mathematically* meet inside a cycle (like two runners on a loop track) without needing to remember anything — O(1) space. |
| "Find the middle node" | **Slow/fast pointer** | O(n), one pass | Naive approach counts length first (one pass), then walks to n/2 (second pass) — two passes. Slow/fast pointers get there in a single pass since fast covers 2x distance. |
| "Reverse a linked list" | **Iterative pointer reversal** | O(n) time, O(1) space | Recursive reversal also works but costs O(n) call-stack space. Iterative in-place pointer flipping needs no extra memory. |

---

## 7. TREES

| Problem statement | Algorithm | Complexity | Why it's best |
|---|---|---|---|
| "Level-by-level output" | **BFS with a queue** | O(n) | Traversal order must match tree depth level by level — a queue's FIFO nature naturally processes nodes in the order they were discovered (breadth-first), which recursion (naturally depth-first) doesn't give you without extra bookkeeping. |
| "Lowest Common Ancestor" | **Recursive bottom-up search** | O(n) | Instead of finding root-to-node paths for both targets separately (two traversals + path comparison), a single post-order recursion returns "found node" info upward, letting the first point where both branches report success be identified in one pass. |
| "Is this a valid BST" | **Recursive with min/max bounds passed down** | O(n) | Just checking `left.val < node.val < right.val` locally misses violations from grandparents. Passing down a valid (min, max) range at each recursive call enforces the *global* BST property, not just the local one — checking this any other way needs an inorder traversal + sorted check (more code, same complexity but easier to get wrong). |

---

## 8. GRAPHS

| Problem statement | Algorithm | Complexity | Why it's best |
|---|---|---|---|
| "Shortest path, unweighted graph" | **BFS** | O(V+E) | BFS explores nodes in increasing order of distance from source (layer by layer), so the first time you reach a node is guaranteed to be via the shortest path. DFS gives *a* path, not the shortest one. |
| "Shortest path, weighted graph, non-negative weights" | **Dijkstra's (with a min-heap)** | O((V+E) log V) | Greedily picking the closest unvisited node and relaxing its edges guarantees correctness because once a node is finalized with the minimum distance, no future (non-negative) edge can improve it. The heap ensures you always pick the next-closest node in O(log V) instead of scanning all nodes O(V). |
| "Connected components / detect cycle in undirected graph" | **Union-Find (DSU)** | ~O(α(n)) per op (near O(1)) | Union-Find directly tracks "which set does this node belong to," so merging components and checking "are these already connected" (i.e., a cycle) is near-constant time with path compression — far cheaper than re-running DFS from scratch for every check. |
| "Order tasks with dependencies" | **Topological sort (Kahn's / DFS-based)** | O(V+E) | Only a DAG has a valid linear ordering respecting dependencies. Kahn's algorithm repeatedly removes nodes with no remaining incoming edges — this directly encodes "do this only after its prerequisites are done," and also detects cycles (if you can't process all nodes, a cycle exists). |

---

## 9. DYNAMIC PROGRAMMING

| Problem statement | Algorithm | Complexity | Why it's best |
|---|---|---|---|
| "Count ways to reach N (climbing stairs, coin combinations)" | **1D DP (tabulation)** | O(n) | Naive recursion recomputes the same subproblems exponentially (O(2ⁿ)). Storing each subproblem's result once (memoization/tabulation) means each is computed exactly once — this is the entire point of DP: trade O(n) or O(n·k) space for turning exponential time into polynomial time. |
| "0/1 Knapsack — max value under weight constraint" | **2D DP** | O(n·W) | Brute force tries all 2ⁿ subsets. DP recognizes that the best solution for capacity W using items 1..i only depends on the best solutions for smaller capacities/fewer items — overlapping subproblems + optimal substructure, DP's two defining conditions. |
| "Longest Common Subsequence / Edit Distance" | **2D DP** | O(m·n) | Comparing two sequences character-by-character has choices at every position (match, skip one, skip other) that repeat across many alignments. A DP table caches the answer for every (i, j) prefix pair once, versus exponential re-exploration in naive recursion. |
| "Longest Increasing Subsequence" | **DP with binary search (patience sorting)** | O(n log n) | Plain DP is O(n²) (for each element, check all previous). The binary-search variant maintains the smallest possible "tail" for each subsequence length, and since that tail array is always sorted, binary search finds the right position to update in O(log n) — cutting a full O(n) inner loop down. |

---

## 10. GREEDY

| Problem statement | Algorithm | Complexity | Why it's best |
|---|---|---|---|
| "Maximum number of non-overlapping activities/meetings" | **Sort by end time, greedily pick** | O(n log n) | Picking the activity that finishes earliest always leaves the most room for future activities — this can be proven correct via an exchange argument (swapping any other valid solution to match the greedy choice never makes it worse). No need for DP since there's no benefit to "trying other options" — greedy's local choice is always part of *some* optimal solution. |
| "Minimum platforms / meeting rooms needed" | **Sort start & end times separately, sweep** | O(n log n) | This reduces to tracking, at every point in time, how many intervals are simultaneously active — sorting turns this into a simple two-pointer sweep instead of checking overlaps pairwise (O(n²)). |
| "Fractional Knapsack" | **Greedy by value/weight ratio** | O(n log n) | Because you can take *fractions* of items, always taking the highest value-density item first is provably optimal — no exchange can improve on it. (Note: this greedy does NOT work for 0/1 Knapsack, which is why that needs DP instead — a classic exam trap.) |

---

## 11. BIT MANIPULATION

| Problem statement | Algorithm | Complexity | Why it's best |
|---|---|---|---|
| "Find the single non-repeating number (all others appear twice)" | **XOR all elements** | O(n), O(1) space | `a ^ a = 0` and `a ^ 0 = a`, so XORing the whole array cancels every pair, leaving only the unpaired element. No hashmap (O(n) space) needed. |
| "Check if a number is a power of 2" | **`x & (x-1) == 0`** | O(1) | Powers of 2 have exactly one set bit. Subtracting 1 flips all bits after (and including) the lowest set bit; ANDing with the original clears that bit — if the result is 0, there was only one bit to begin with. Far faster than a loop dividing by 2. |

---

## Quick decision rule of thumb
1. **Is the input sorted, or can you sort it cheaply?** → Think binary search / two pointers.
2. **Do you need "all X" (all subsets/permutations)?** → Backtracking.
3. **Are you asked to count ways or optimize a value with choices/constraints?** → DP (check for overlapping subproblems first — if none, it's just recursion).
4. **Is there a provably-safe locally optimal choice (exchange argument holds)?** → Greedy (cheaper than DP if applicable).
5. **Do you need frequency, existence, or complement lookups fast?** → Hashing.
6. **Is there a LIFO/"most recent unresolved" dependency?** → Stack.
7. **Shortest path / reachability?** → BFS (unweighted) or Dijkstra (weighted, non-negative).
8. **Range sum/count over subarrays repeatedly?** → Prefix sum (+ hashmap if negatives are involved).

If two techniques both give correct answers, the "best" one for an OA is whichever has the lower time complexity for the given constraints (check n's upper bound in the problem statement — that's usually a direct hint at the expected complexity).
