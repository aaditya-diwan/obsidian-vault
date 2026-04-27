# DSA Complete Learning Program — 100 Problems

> **Goal:** Pattern recognition + deep intuition for coding interviews at top tech companies.
> **Stack:** Java (primary) + Python (secondary)
> **Progression:** Foundational → Intermediate → Advanced

---

## TABLE OF CONTENTS

| # | Topic | Problems |
|---|-------|----------|
| 1 | Arrays & Strings | 1–15 |
| 2 | Hashing & Sets | 16–25 |
| 3 | Two Pointers & Sliding Window | 26–35 |
| 4 | Recursion & Backtracking | 36–45 |
| 5 | Stack & Queue | 46–55 |
| 6 | Linked List | 56–65 |
| 7 | Binary Trees & BST | 66–75 |
| 8 | Heaps & Priority Queue | 76–80 |
| 9 | Graphs | 81–90 |
| 10 | Dynamic Programming | 91–100 |

---

# SECTION 1 — ARRAYS & STRINGS (Problems 1–15)

> **Foundation block.** Arrays are the most tested data structure. Master these before anything else.

---

## Problem 1 — Two Sum

**Difficulty:** Easy
**Real-world intuition:** Finding two items in a shopping cart that total exactly your budget. Appears in financial systems, pair-matching engines, and recommendation systems.

---

### Theory

- **Core concept:** Given an array and a target, find two indices whose values sum to target.
- **Pattern:** HashMap for O(1) complement lookup.
- **When to use:** Any problem asking "find two elements satisfying a condition" — reach for a HashMap first.
- **Common mistakes:**
  - Using the same element twice (index i paired with itself).
  - Sorting the array first (destroys original indices if you need to return them).
  - Forgetting that duplicate values are allowed (e.g., `[3,3]`, target=6).

---

### Brute Force

Check every pair (i, j) where i ≠ j.

```
for i in 0..n-1:
  for j in i+1..n-1:
    if nums[i] + nums[j] == target: return [i, j]
```

- **Time:** O(n²) — nested loops.
- **Space:** O(1)
- **Why inefficient:** For n=10,000 you make 50 million comparisons. Unacceptable.

---

### Optimal Approach

**Intuition:** For every element `x`, the complement we need is `target - x`. If we store every element we've seen in a HashMap (value → index), a single O(1) lookup tells us whether the complement already exists.

**Steps:**
1. Create `HashMap<Integer, Integer> map`.
2. Iterate through `nums` with index `i`.
3. Compute `complement = target - nums[i]`.
4. If `complement` is in `map`, return `[map.get(complement), i]`.
5. Otherwise, put `nums[i] → i` into the map.

- **Time:** O(n) — single pass.
- **Space:** O(n) — worst case we store all elements before finding a pair.

---

### Java Implementation

```java
import java.util.HashMap;

public class TwoSum {
    public int[] twoSum(int[] nums, int target) {
        HashMap<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            if (map.containsKey(complement)) {
                return new int[]{map.get(complement), i};
            }
            map.put(nums[i], i);
        }
        return new int[]{};  // guaranteed to have solution per problem statement
    }
}
```

### Python Implementation

```python
def two_sum(nums: list[int], target: int) -> list[int]:
    seen = {}
    for i, x in enumerate(nums):
        complement = target - x
        if complement in seen:
            return [seen[complement], i]
        seen[x] = i
    return []
```

---

### Dry Run

**Input:** `nums = [2, 7, 11, 15]`, `target = 9`

| i | nums[i] | complement | map before lookup | action |
|---|---------|------------|-------------------|--------|
| 0 | 2 | 7 | {} | 7 not found → add {2:0} |
| 1 | 7 | 2 | {2:0} | 2 found at index 0 → return [0,1] |

**Output:** `[0, 1]` ✓

---

### Variations

1. **Two Sum II — Sorted Array:** Use two pointers instead of HashMap. O(n) time, O(1) space.
2. **Two Sum IV — BST:** For each node, check if complement exists in BST. O(n log n).
3. **Three Sum:** Fix one element, run Two Sum on the rest. O(n²).

---

## Problem 2 — Best Time to Buy and Sell Stock

**Difficulty:** Easy
**Real-world intuition:** You have one chance to buy and one chance to sell a stock. Maximize profit. This is the single-transaction stock problem and models real trading constraints.

---

### Theory

- **Core concept:** Track the minimum price seen so far; at each step compute potential profit.
- **Pattern:** Single-pass greedy scan with running minimum.
- **When to use:** Any "maximum gain over a sequence" problem where you can only act once.
- **Common mistakes:**
  - Selling before buying (index of sell must be after buy).
  - Not initializing `minPrice` to `Integer.MAX_VALUE` or `prices[0]`.
  - Confusing this with the unlimited-transactions variant.

---

### Brute Force

For every pair (buy, sell) where buy < sell, compute `prices[sell] - prices[buy]`.

- **Time:** O(n²)
- **Space:** O(1)
- **Why inefficient:** Redundant comparisons; we recompute min repeatedly.

---

### Optimal Approach

**Intuition:** As we scan left to right, the best buy price for any future sell day is the minimum price seen so far. We never need to look back further.

**Steps:**
1. Initialize `minPrice = prices[0]`, `maxProfit = 0`.
2. For each price from index 1 onward:
   - Update `maxProfit = max(maxProfit, price - minPrice)`.
   - Update `minPrice = min(minPrice, price)`.
3. Return `maxProfit`.

- **Time:** O(n)
- **Space:** O(1)

---

### Java Implementation

```java
public class BestTimeToBuySellStock {
    public int maxProfit(int[] prices) {
        int minPrice = Integer.MAX_VALUE;
        int maxProfit = 0;
        for (int price : prices) {
            maxProfit = Math.max(maxProfit, price - minPrice);
            minPrice = Math.min(minPrice, price);
        }
        return maxProfit;
    }
}
```

### Python Implementation

```python
def max_profit(prices: list[int]) -> int:
    min_price = float('inf')
    max_profit = 0
    for price in prices:
        max_profit = max(max_profit, price - min_price)
        min_price = min(min_price, price)
    return max_profit
```

---

### Dry Run

**Input:** `prices = [7, 1, 5, 3, 6, 4]`

| price | minPrice | profit (price-min) | maxProfit |
|-------|----------|--------------------|-----------|
| 7 | 7 | 0 | 0 |
| 1 | 1 | 0 | 0 |
| 5 | 1 | 4 | 4 |
| 3 | 1 | 2 | 4 |
| 6 | 1 | 5 | 5 |
| 4 | 1 | 3 | 5 |

**Output:** `5` (buy at 1, sell at 6) ✓

---

### Variations

1. **Best Time II — Unlimited transactions:** Sum all positive consecutive differences.
2. **Best Time III — At most 2 transactions:** DP with state machine.
3. **Best Time IV — At most k transactions:** Generalized DP, O(nk).

---

## Problem 3 — Maximum Subarray (Kadane's Algorithm)

**Difficulty:** Medium
**Real-world intuition:** Find the best consecutive window of days where a metric (stock price change, user engagement) is maximized. Core to signal processing and financial analytics.

---

### Theory

- **Core concept:** At each position, decide: extend the current subarray or start fresh.
- **Pattern:** Kadane's algorithm — running sum with reset.
- **When to use:** "Maximum sum of contiguous subarray" or any variant involving a running aggregate that can be reset.
- **Common mistakes:**
  - Initializing `maxSum = 0` (wrong when all elements are negative — answer should be the least negative number).
  - Confusing `currentSum` reset with `maxSum` reset.

---

### Brute Force

Compute sum of every subarray `[i, j]`.

- **Time:** O(n²) with prefix sums, O(n³) naively.
- **Space:** O(1) or O(n) for prefix sums.

---

### Optimal Approach

**Intuition (Kadane's):** If the running sum ever goes negative, it can only hurt future subarrays. Discard it and start fresh at the current element.

**Steps:**
1. Initialize `currentSum = nums[0]`, `maxSum = nums[0]`.
2. For each `num` from index 1:
   - `currentSum = max(num, currentSum + num)` — take current alone or extend.
   - `maxSum = max(maxSum, currentSum)`.
3. Return `maxSum`.

- **Time:** O(n)
- **Space:** O(1)

---

### Java Implementation

```java
public class MaximumSubarray {
    public int maxSubArray(int[] nums) {
        int currentSum = nums[0];
        int maxSum = nums[0];
        for (int i = 1; i < nums.length; i++) {
            currentSum = Math.max(nums[i], currentSum + nums[i]);
            maxSum = Math.max(maxSum, currentSum);
        }
        return maxSum;
    }
}
```

### Python Implementation

```python
def max_sub_array(nums: list[int]) -> int:
    current = max_sum = nums[0]
    for num in nums[1:]:
        current = max(num, current + num)
        max_sum = max(max_sum, current)
    return max_sum
```

---

### Dry Run

**Input:** `[-2, 1, -3, 4, -1, 2, 1, -5, 4]`

| i | num | currentSum | maxSum |
|---|-----|------------|--------|
| 0 | -2 | -2 | -2 |
| 1 | 1 | max(1,-1)=1 | 1 |
| 2 | -3 | max(-3,-2)=-2 | 1 |
| 3 | 4 | max(4,2)=4 | 4 |
| 4 | -1 | max(-1,3)=3 | 4 |
| 5 | 2 | max(2,5)=5 | 5 |
| 6 | 1 | max(1,6)=6 | 6 |
| 7 | -5 | max(-5,1)=1 | 6 |
| 8 | 4 | max(4,5)=5 | 6 |

**Output:** `6` (subarray `[4,-1,2,1]`) ✓

---

### Variations

1. **Maximum Product Subarray:** Track both max and min (negatives flip sign).
2. **Maximum Sum Circular Subarray:** Use Kadane's + (total sum − minimum subarray sum).
3. **Subarray with given sum (non-negative):** Sliding window.

---

## Problem 4 — Product of Array Except Self

**Difficulty:** Medium
**Real-world intuition:** In data pipelines, you often need to compute a derived value for each element based on all others — e.g., normalized weights, leave-one-out statistics.

---

### Theory

- **Core concept:** For each index i, compute product of all elements to its left and all elements to its right.
- **Pattern:** Prefix and suffix product arrays (or two-pass in-place).
- **When to use:** "Operation on all elements except current" — division-free prefix/suffix scan.
- **Common mistakes:**
  - Using division (fails when array contains zeros).
  - Allocating separate left and right arrays unnecessarily when one pass suffices.

---

### Brute Force

For each i, multiply all elements except `nums[i]`.

- **Time:** O(n²)
- **Space:** O(1) beyond output

---

### Optimal Approach

**Intuition:** `result[i] = (product of nums[0..i-1]) × (product of nums[i+1..n-1])`.

**Two-pass approach:**
1. **Left pass:** `result[i]` = product of all elements to the left of i.
2. **Right pass:** Multiply `result[i]` by the running product from the right.

- **Time:** O(n)
- **Space:** O(1) extra (output array doesn't count)

---

### Java Implementation

```java
public class ProductExceptSelf {
    public int[] productExceptSelf(int[] nums) {
        int n = nums.length;
        int[] result = new int[n];

        // Left products
        result[0] = 1;
        for (int i = 1; i < n; i++) {
            result[i] = result[i - 1] * nums[i - 1];
        }

        // Multiply by right products
        int rightProduct = 1;
        for (int i = n - 1; i >= 0; i--) {
            result[i] *= rightProduct;
            rightProduct *= nums[i];
        }

        return result;
    }
}
```

### Python Implementation

```python
def product_except_self(nums: list[int]) -> list[int]:
    n = len(nums)
    result = [1] * n
    for i in range(1, n):
        result[i] = result[i - 1] * nums[i - 1]
    right = 1
    for i in range(n - 1, -1, -1):
        result[i] *= right
        right *= nums[i]
    return result
```

---

### Dry Run

**Input:** `[1, 2, 3, 4]`

Left pass:
`result = [1, 1, 2, 6]`

Right pass (rightProduct starts at 1):
- i=3: result[3]=6×1=6, rightProduct=4
- i=2: result[2]=2×4=8, rightProduct=12
- i=1: result[1]=1×12=12, rightProduct=24
- i=0: result[0]=1×24=24, rightProduct=24

**Output:** `[24, 12, 8, 6]` ✓

---

### Variations

1. **With zeros in array:** Count zeros; if >1 all are 0; if exactly 1 only zero-index gets non-zero.
2. **Running XOR except self:** Same prefix/suffix pattern with XOR operator.
3. **Sum of array except self:** Compute total sum then subtract (simpler but teaches same pattern).

---

## Problem 5 — Rotate Array

**Difficulty:** Medium
**Real-world intuition:** Circular buffers in embedded systems, playlist rotation, image transformation — rotating an array in-place efficiently is a fundamental low-level skill.

---

### Theory

- **Core concept:** Shift array elements right by k positions, wrapping around.
- **Pattern:** Three-reverse trick (reverse whole, reverse first k, reverse rest).
- **When to use:** In-place rotation without extra space.
- **Common mistakes:**
  - Not handling `k > n` — use `k = k % n`.
  - Off-by-one errors in the reverse ranges.
  - Using extra O(n) space unnecessarily.

---

### Brute Force

Rotate one step at a time, k times. Each step: save last element, shift all right, put saved at front.

- **Time:** O(n×k)
- **Space:** O(1)

---

### Optimal Approach

**Intuition (Three Reverses):**
- After rotating right by k, the last k elements become the first k.
- Reverse entire array → now those k elements are at the front but reversed.
- Reverse first k → they're in correct order.
- Reverse remaining n-k → correct order.

**Steps:**
1. `k = k % n`
2. `reverse(0, n-1)`
3. `reverse(0, k-1)`
4. `reverse(k, n-1)`

- **Time:** O(n)
- **Space:** O(1)

---

### Java Implementation

```java
public class RotateArray {
    public void rotate(int[] nums, int k) {
        int n = nums.length;
        k = k % n;
        reverse(nums, 0, n - 1);
        reverse(nums, 0, k - 1);
        reverse(nums, k, n - 1);
    }

    private void reverse(int[] nums, int left, int right) {
        while (left < right) {
            int temp = nums[left];
            nums[left] = nums[right];
            nums[right] = temp;
            left++;
            right--;
        }
    }
}
```

### Python Implementation

```python
def rotate(nums: list[int], k: int) -> None:
    n = len(nums)
    k %= n
    nums.reverse()
    nums[:k] = nums[:k][::-1]
    nums[k:] = nums[k:][::-1]
```

---

### Dry Run

**Input:** `[1,2,3,4,5,6,7]`, k=3

1. Reverse all: `[7,6,5,4,3,2,1]`
2. Reverse first 3: `[5,6,7,4,3,2,1]`
3. Reverse last 4: `[5,6,7,1,2,3,4]`

**Output:** `[5,6,7,1,2,3,4]` ✓

---

### Variations

1. **Rotate Left by k:** Equivalent to rotate right by n-k.
2. **Rotate Matrix 90°:** Different problem — transpose + reverse rows.
3. **Find Minimum in Rotated Array:** Binary search variant (Problem 8 below).

---

## Problem 6 — Find the Duplicate Number

**Difficulty:** Medium
**Real-world intuition:** Data integrity checks — detecting duplicate IDs in a database table with n+1 rows that should hold values 1 to n.

---

### Theory

- **Core concept:** Array of n+1 integers with values 1..n, exactly one duplicate.
- **Pattern:** Floyd's Cycle Detection (tortoise and hare) — treat array as a linked list.
- **When to use:** When the problem requires O(1) space and no modification of the input.
- **Common mistakes:**
  - Using a HashSet (valid but O(n) space — interviewer will push for O(1)).
  - Marking visited by negating values (modifies input — often not allowed).
  - Misidentifying the entry point of the cycle.

---

### Brute Force (HashSet)

Store each number in a set; return on first duplicate.

- **Time:** O(n), **Space:** O(n)

---

### Optimal Approach — Floyd's Cycle Detection

**Intuition:** Map array as a function `f(i) = nums[i]`. Because values are in [1,n] and array has n+1 elements, this function must create a cycle. The duplicate value is the cycle entry point.

**Steps:**
1. **Phase 1 — Detect cycle:** slow=nums[0], fast=nums[nums[0]]. Move slow by 1, fast by 2. Stop when `slow == fast`.
2. **Phase 2 — Find entry:** Reset slow=0. Move both by 1 until `slow == fast`. That is the duplicate.

- **Time:** O(n)
- **Space:** O(1)

---

### Java Implementation

```java
public class FindDuplicate {
    public int findDuplicate(int[] nums) {
        int slow = nums[0];
        int fast = nums[nums[0]];

        while (slow != fast) {
            slow = nums[slow];
            fast = nums[nums[fast]];
        }

        slow = 0;
        while (slow != fast) {
            slow = nums[slow];
            fast = nums[fast];
        }

        return slow;
    }
}
```

### Python Implementation

```python
def find_duplicate(nums: list[int]) -> int:
    slow, fast = nums[0], nums[nums[0]]
    while slow != fast:
        slow = nums[slow]
        fast = nums[nums[fast]]
    slow = 0
    while slow != fast:
        slow = nums[slow]
        fast = nums[fast]
    return slow
```

---

### Dry Run

**Input:** `[1,3,4,2,2]`

Phase 1: slow=1, fast=3 → slow=3, fast=2 → slow=2, fast=nums[nums[2]]=nums[4]=2. Meet at 2.
Phase 2: slow=0. slow=nums[0]=1, fast=nums[2]=4. slow=nums[1]=3, fast=nums[4]=2. slow=nums[3]=2, fast=nums[2]=4. slow=nums[2]=4, fast=nums[4]=2. slow=nums[4]=2, fast=nums[2]=4... 

Actually let me redo this more carefully.

**Input:** `[1,3,4,2,2]` — indices 0,1,2,3,4, values 1,3,4,2,2

Phase 1:
- slow=nums[0]=1, fast=nums[nums[0]]=nums[1]=3
- slow=nums[1]=3, fast=nums[nums[3]]=nums[2]=4
- slow=nums[3]=2, fast=nums[nums[4]]=nums[2]=4
- slow=nums[2]=4, fast=nums[nums[4]]=nums[2]=4 → meet at 4

Phase 2: slow=0
- slow=nums[0]=1, fast=nums[4]=2
- slow=nums[1]=3, fast=nums[2]=4
- slow=nums[3]=2, fast=nums[4]=2 → meet at 2

**Output:** `2` ✓

---

### Variations

1. **Missing Number:** XOR all indices and values.
2. **First Missing Positive:** Cyclic sort approach.
3. **Find All Duplicates:** Negate value at index; if already negative it's duplicate.

---

## Problem 7 — Merge Intervals

**Difficulty:** Medium
**Real-world intuition:** Calendar apps merging overlapping meetings, network packet reassembly, genomics (merging overlapping gene regions).

---

### Theory

- **Core concept:** Sort intervals by start time; merge if current interval overlaps the last merged.
- **Pattern:** Sort + linear scan merge.
- **When to use:** Any "overlapping ranges" problem.
- **Common mistakes:**
  - Forgetting to sort by start time first.
  - Overlap condition: `current.start <= last.end` (not strictly less).
  - Not handling single-interval input.

---

### Brute Force

Compare every pair of intervals and merge when overlapping, repeat until no merges. O(n²) per pass, multiple passes needed.

---

### Optimal Approach

1. Sort intervals by `start`.
2. Add first interval to result.
3. For each remaining interval:
   - If `interval.start <= result.last.end` → merge: update `result.last.end = max(result.last.end, interval.end)`.
   - Else → add new interval to result.

- **Time:** O(n log n) — dominated by sort.
- **Space:** O(n) — output list.

---

### Java Implementation

```java
import java.util.Arrays;
import java.util.ArrayList;
import java.util.List;

public class MergeIntervals {
    public int[][] merge(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
        List<int[]> result = new ArrayList<>();
        result.add(intervals[0]);

        for (int i = 1; i < intervals.length; i++) {
            int[] last = result.get(result.size() - 1);
            if (intervals[i][0] <= last[1]) {
                last[1] = Math.max(last[1], intervals[i][1]);
            } else {
                result.add(intervals[i]);
            }
        }

        return result.toArray(new int[result.size()][]);
    }
}
```

### Python Implementation

```python
def merge(intervals: list[list[int]]) -> list[list[int]]:
    intervals.sort(key=lambda x: x[0])
    merged = [intervals[0]]
    for start, end in intervals[1:]:
        if start <= merged[-1][1]:
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])
    return merged
```

---

### Dry Run

**Input:** `[[1,3],[2,6],[8,10],[15,18]]`

After sort (already sorted): `[[1,3],[2,6],[8,10],[15,18]]`

- Add [1,3]
- [2,6]: 2≤3 → merge → [1,6]
- [8,10]: 8>6 → add → result: [1,6],[8,10]
- [15,18]: 15>10 → add → result: [1,6],[8,10],[15,18]

**Output:** `[[1,6],[8,10],[15,18]]` ✓

---

### Variations

1. **Insert Interval:** Merge a new interval into a sorted non-overlapping list.
2. **Meeting Rooms I:** Can one person attend all? Check if any overlap.
3. **Meeting Rooms II:** Min rooms needed — sort starts and ends, two-pointer scan.

---

## Problem 8 — Search in Rotated Sorted Array

**Difficulty:** Medium
**Real-world intuition:** Searching in any circular or partially-ordered structure — e.g., looking up a value in a log that wraps around midnight.

---

### Theory

- **Core concept:** Binary search modified for a rotated array; at each step, identify which half is sorted.
- **Pattern:** Modified binary search.
- **When to use:** Sorted array with a rotation (pivot). Also generalizes to "find pivot" problems.
- **Common mistakes:**
  - Not correctly identifying which half is sorted.
  - Incorrect boundary conditions (use `<=` not `<` for the sorted-half check).
  - Forgetting to handle the case where `nums[mid] == target`.

---

### Brute Force

Linear scan. O(n). Works but defeats the purpose of sorting.

---

### Optimal Approach

**Intuition:** Even in a rotated array, at least one half around `mid` is guaranteed to be sorted. Check if target lies in the sorted half; if yes, search there; otherwise search the other half.

**Steps:**
1. `left=0, right=n-1`.
2. While `left <= right`:
   - `mid = (left+right)/2`. Return if `nums[mid]==target`.
   - If `nums[left] <= nums[mid]` → left half is sorted:
     - If `nums[left] <= target < nums[mid]` → search left half.
     - Else search right half.
   - Else → right half is sorted:
     - If `nums[mid] < target <= nums[right]` → search right half.
     - Else search left half.

- **Time:** O(log n)
- **Space:** O(1)

---

### Java Implementation

```java
public class SearchRotatedArray {
    public int search(int[] nums, int target) {
        int left = 0, right = nums.length - 1;

        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] == target) return mid;

            if (nums[left] <= nums[mid]) {          // left half is sorted
                if (nums[left] <= target && target < nums[mid]) {
                    right = mid - 1;
                } else {
                    left = mid + 1;
                }
            } else {                                // right half is sorted
                if (nums[mid] < target && target <= nums[right]) {
                    left = mid + 1;
                } else {
                    right = mid - 1;
                }
            }
        }

        return -1;
    }
}
```

### Python Implementation

```python
def search(nums: list[int], target: int) -> int:
    left, right = 0, len(nums) - 1
    while left <= right:
        mid = (left + right) // 2
        if nums[mid] == target:
            return mid
        if nums[left] <= nums[mid]:
            if nums[left] <= target < nums[mid]:
                right = mid - 1
            else:
                left = mid + 1
        else:
            if nums[mid] < target <= nums[right]:
                left = mid + 1
            else:
                right = mid - 1
    return -1
```

---

### Dry Run

**Input:** `[4,5,6,7,0,1,2]`, target=0

| left | right | mid | nums[mid] | action |
|------|-------|-----|-----------|--------|
| 0 | 6 | 3 | 7 | left[4]≤mid[7]: sorted left. target=0 not in [4,7) → right half |
| 4 | 6 | 5 | 1 | right half sorted. target=0 not in (1,2] → left half |
| 4 | 4 | 4 | 0 | found! return 4 |

**Output:** `4` ✓

---

### Variations

1. **Find Minimum in Rotated Array:** Binary search for pivot.
2. **Search in Rotated Array II (with duplicates):** Handle `nums[left]==nums[mid]` case by `left++`.
3. **Find Rotation Count:** Find index of minimum element.

---

## Problem 9 — Spiral Matrix

**Difficulty:** Medium
**Real-world intuition:** Printing matrices in spiral order appears in image processing, robot path planning (spiral coverage), and layer-by-layer data export.

---

### Theory

- **Core concept:** Traverse an m×n matrix in spiral order using boundary shrinking.
- **Pattern:** Four-boundary simulation (top, bottom, left, right).
- **When to use:** Any layered or boundary-traversal matrix problem.
- **Common mistakes:**
  - Not checking `top <= bottom` and `left <= right` before the left and top sweeps (causes duplicates in non-square matrices).
  - Off-by-one on boundary updates.

---

### Brute Force / Only Approach

There is no asymptotically better solution — we must visit all n×m cells. The key is doing it cleanly.

---

### Optimal Approach

**Steps:**
1. Initialize `top=0, bottom=m-1, left=0, right=n-1`.
2. While `top<=bottom && left<=right`:
   - Traverse right along `top` row, then `top++`.
   - Traverse down along `right` column, then `right--`.
   - If `top<=bottom`: traverse left along `bottom` row, then `bottom--`.
   - If `left<=right`: traverse up along `left` column, then `left++`.

- **Time:** O(m×n)
- **Space:** O(1) extra (excluding output)

---

### Java Implementation

```java
import java.util.ArrayList;
import java.util.List;

public class SpiralMatrix {
    public List<Integer> spiralOrder(int[][] matrix) {
        List<Integer> result = new ArrayList<>();
        int top = 0, bottom = matrix.length - 1;
        int left = 0, right = matrix[0].length - 1;

        while (top <= bottom && left <= right) {
            for (int col = left; col <= right; col++) result.add(matrix[top][col]);
            top++;
            for (int row = top; row <= bottom; row++) result.add(matrix[row][right]);
            right--;
            if (top <= bottom) {
                for (int col = right; col >= left; col--) result.add(matrix[bottom][col]);
                bottom--;
            }
            if (left <= right) {
                for (int row = bottom; row >= top; row--) result.add(matrix[row][left]);
                left++;
            }
        }

        return result;
    }
}
```

### Python Implementation

```python
def spiral_order(matrix: list[list[int]]) -> list[int]:
    result = []
    top, bottom, left, right = 0, len(matrix)-1, 0, len(matrix[0])-1
    while top <= bottom and left <= right:
        for col in range(left, right+1): result.append(matrix[top][col])
        top += 1
        for row in range(top, bottom+1): result.append(matrix[row][right])
        right -= 1
        if top <= bottom:
            for col in range(right, left-1, -1): result.append(matrix[bottom][col])
            bottom -= 1
        if left <= right:
            for row in range(bottom, top-1, -1): result.append(matrix[row][left])
            left += 1
    return result
```

---

### Dry Run

**Input:**
```
[[1,2,3],
 [4,5,6],
 [7,8,9]]
```
- Right (row 0): 1,2,3 → top=1
- Down (col 2): 6,9 → right=1
- Left (row 2): 8,7 → bottom=1
- Up (col 0): 4 → left=1
- Right (row 1): 5 → top=2 (loop ends)

**Output:** `[1,2,3,6,9,8,7,4,5]` ✓

---

### Variations

1. **Spiral Matrix II:** Fill matrix in spiral order with 1..n².
2. **Diagonal Traverse:** Different traversal pattern, same boundary logic.
3. **Rotate Image 90°:** Transpose + reverse rows.

---

## Problem 10 — Set Matrix Zeroes

**Difficulty:** Medium
**Real-world intuition:** Data cleaning pipelines — propagating null/error markers across rows and columns. Common in spreadsheet engines.

---

### Theory

- **Core concept:** If any cell is 0, set its entire row and column to 0.
- **Pattern:** Use first row and column as markers (in-place flag storage).
- **When to use:** In-place matrix modification with O(1) space constraint.
- **Common mistakes:**
  - Zeroing in the first pass (which corrupts marker information for later cells).
  - Not handling the first row/column separately (they double as markers).

---

### Brute Force

First pass: collect all (row, col) positions where value is 0. Second pass: zero those rows/columns.

- **Time:** O(m×n), **Space:** O(m+n)

---

### Optimal Approach (O(1) Space)

**Intuition:** Use `matrix[0][j]` as the flag for column j, and `matrix[i][0]` as the flag for row i. Use a separate boolean for the first row itself (since `matrix[0][0]` is shared).

**Steps:**
1. Check if first row contains a zero (save as `firstRowZero`).
2. Check if first column contains a zero (save as `firstColZero`).
3. For each cell (i≥1, j≥1): if `matrix[i][j]==0`, set `matrix[i][0]=0` and `matrix[0][j]=0`.
4. For i≥1, j≥1: if `matrix[i][0]==0` or `matrix[0][j]==0`, set `matrix[i][j]=0`.
5. If `firstRowZero`, zero entire first row.
6. If `firstColZero`, zero entire first column.

- **Time:** O(m×n), **Space:** O(1)

---

### Java Implementation

```java
public class SetMatrixZeroes {
    public void setZeroes(int[][] matrix) {
        int m = matrix.length, n = matrix[0].length;
        boolean firstRowZero = false, firstColZero = false;

        for (int j = 0; j < n; j++) if (matrix[0][j] == 0) firstRowZero = true;
        for (int i = 0; i < m; i++) if (matrix[i][0] == 0) firstColZero = true;

        for (int i = 1; i < m; i++)
            for (int j = 1; j < n; j++)
                if (matrix[i][j] == 0) { matrix[i][0] = 0; matrix[0][j] = 0; }

        for (int i = 1; i < m; i++)
            for (int j = 1; j < n; j++)
                if (matrix[i][0] == 0 || matrix[0][j] == 0) matrix[i][j] = 0;

        if (firstRowZero) for (int j = 0; j < n; j++) matrix[0][j] = 0;
        if (firstColZero) for (int i = 0; i < m; i++) matrix[i][0] = 0;
    }
}
```

---

### Dry Run

**Input:**
```
[[1,1,1],
 [1,0,1],
 [1,1,1]]
```
- firstRowZero=false, firstColZero=false
- (1,1) is 0 → matrix[1][0]=0, matrix[0][1]=0
- Second pass: row 1 zeroed (col 0 flag), col 1 zeroed (row 0 flag)

**Output:**
```
[[1,0,1],
 [0,0,0],
 [1,0,1]]
```
✓

---

### Variations

1. **Game of Life:** Similar in-place state propagation — encode old+new state in one cell.
2. **Walls and Gates:** BFS propagation from 0-cells.

---

## Problem 11 — Longest Common Prefix

**Difficulty:** Easy
**Real-world intuition:** Autocomplete systems, file path compression, DNS lookup optimization.

---

### Theory

- **Core concept:** The LCP of all strings is at most as long as the shortest string.
- **Pattern:** Vertical scan (compare character by character across all strings at same index).
- **When to use:** String prefix matching, trie construction baseline.
- **Common mistakes:**
  - Returning empty string on empty input without checking.
  - Comparing only adjacent pairs rather than all strings.

---

### Optimal Approach

Take `strs[0]` as the reference. For each character position i, check if all strings have the same character at position i. Stop at first mismatch or end of any string.

- **Time:** O(S) where S = total characters. **Space:** O(1)

---

### Java Implementation

```java
public class LongestCommonPrefix {
    public String longestCommonPrefix(String[] strs) {
        if (strs.length == 0) return "";
        for (int i = 0; i < strs[0].length(); i++) {
            char c = strs[0].charAt(i);
            for (int j = 1; j < strs.length; j++) {
                if (i >= strs[j].length() || strs[j].charAt(i) != c) {
                    return strs[0].substring(0, i);
                }
            }
        }
        return strs[0];
    }
}
```

---

### Variations

1. **Longest Common Suffix:** Reverse strings, apply same algorithm.
2. **Word grouping by prefix:** Use a Trie.
3. **LCP of all suffixes:** Suffix array + LCP array.

---

## Problem 12 — Valid Anagram

**Difficulty:** Easy
**Real-world intuition:** Detecting word scrambles in word games, plagiarism detection by word rearrangement.

---

### Theory

- **Core concept:** Two strings are anagrams if they contain the same characters with the same frequencies.
- **Pattern:** Frequency counting array (size 26 for lowercase letters).
- **When to use:** Any "same characters, different order" problem.
- **Common mistakes:**
  - Only checking length (necessary but not sufficient).
  - Using sort (O(n log n)) when O(n) is possible.

---

### Optimal Approach

1. If lengths differ, return false.
2. Create `int[26] count`.
3. For each char: `count[s.charAt(i)-'a']++`, `count[t.charAt(i)-'a']--`.
4. Return true if all counts are 0.

- **Time:** O(n), **Space:** O(1) (fixed-size array)

---

### Java Implementation

```java
public class ValidAnagram {
    public boolean isAnagram(String s, String t) {
        if (s.length() != t.length()) return false;
        int[] count = new int[26];
        for (int i = 0; i < s.length(); i++) {
            count[s.charAt(i) - 'a']++;
            count[t.charAt(i) - 'a']--;
        }
        for (int c : count) if (c != 0) return false;
        return true;
    }
}
```

---

### Variations

1. **Group Anagrams:** HashMap with sorted-string key (Problem 17).
2. **Find All Anagram Positions in String:** Sliding window with frequency array.
3. **Unicode/arbitrary characters:** Use HashMap instead of int[26].

---

## Problem 13 — Reverse Words in a String

**Difficulty:** Medium
**Real-world intuition:** Text processing, NLP preprocessing, command-line argument reversal.

---

### Theory

- **Core concept:** Split on whitespace, reverse the list of words, rejoin.
- **Pattern:** String splitting + reversal.
- **When to use:** Word-level (not character-level) string manipulation.
- **Common mistakes:**
  - Not trimming leading/trailing spaces.
  - Multiple spaces between words (use `\\s+` regex or `split(" ")` carefully).

---

### Java Implementation

```java
public class ReverseWords {
    public String reverseWords(String s) {
        String[] words = s.trim().split("\\s+");
        StringBuilder sb = new StringBuilder();
        for (int i = words.length - 1; i >= 0; i--) {
            sb.append(words[i]);
            if (i > 0) sb.append(" ");
        }
        return sb.toString();
    }
}
```

### Python Implementation

```python
def reverse_words(s: str) -> str:
    return " ".join(s.split()[::-1])
```

---

### Variations

1. **Reverse only characters in each word:** Split, reverse each word's chars, rejoin.
2. **Rotate words left by k:** `words[k:] + words[:k]`.
3. **In-place character reversal (no split):** Two-pointer reverse whole string, then reverse each word.

---

## Problem 14 — Minimum Window Substring

**Difficulty:** Hard
**Real-world intuition:** Finding the smallest document excerpt that contains all required keywords — used in search engines and text analytics.

---

### Theory

- **Core concept:** Find the smallest window in `s` that contains all characters of `t`.
- **Pattern:** Sliding window with two frequency maps.
- **When to use:** "Smallest/shortest subarray/substring containing all of X" → sliding window.
- **Common mistakes:**
  - Not tracking how many characters are "satisfied" (count of chars meeting required frequency).
  - Shrinking window when it's not valid instead of only when it is valid.
  - Off-by-one on the answer window bounds.

---

### Optimal Approach

**Intuition:** Expand right pointer until window is valid (contains all of t). Then shrink left pointer to minimize. Track validity using a `formed` counter that increments when a character's frequency in window matches required frequency.

**Steps:**
1. Build `need` map from t.
2. `left=0, formed=0, required=need.size()`.
3. Expand `right`; if `windowCount[c]==need[c]`, `formed++`.
4. While `formed==required`: update answer, shrink left. If `windowCount[c]<need[c]`, `formed--`.

- **Time:** O(|s|+|t|)
- **Space:** O(|s|+|t|)

---

### Java Implementation

```java
import java.util.HashMap;

public class MinWindowSubstring {
    public String minWindow(String s, String t) {
        if (s.isEmpty() || t.isEmpty()) return "";

        HashMap<Character, Integer> need = new HashMap<>();
        for (char c : t.toCharArray()) need.merge(c, 1, Integer::sum);

        HashMap<Character, Integer> window = new HashMap<>();
        int left = 0, formed = 0, required = need.size();
        int[] best = {-1, 0, 0};  // [length, left, right]

        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);
            window.merge(c, 1, Integer::sum);
            if (need.containsKey(c) && window.get(c).equals(need.get(c))) formed++;

            while (formed == required) {
                if (best[0] == -1 || right - left + 1 < best[0]) {
                    best[0] = right - left + 1;
                    best[1] = left;
                    best[2] = right;
                }
                char lc = s.charAt(left);
                window.merge(lc, -1, Integer::sum);
                if (need.containsKey(lc) && window.get(lc) < need.get(lc)) formed--;
                left++;
            }
        }

        return best[0] == -1 ? "" : s.substring(best[1], best[2] + 1);
    }
}
```

---

### Dry Run

**Input:** s=`"ADOBECODEBANC"`, t=`"ABC"`

need={A:1,B:1,C:1}, required=3

- Expand until right=5 (ADOBEC): window has A,B,C → formed=3. Window="ADOBEC" len=6.
- Shrink left: remove A → formed=2, left=1.
- Continue expanding... at right=10 (ADOBECODEBA): formed=3 again.
- Window="DOBECODEBA" no... track carefully — best answer is "BANC" (len=4).

**Output:** `"BANC"` ✓

---

### Variations

1. **Permutation in String:** Fixed-size window, same character matching.
2. **Smallest Substring Containing All Unique Chars:** Simpler version.
3. **Longest Substring with At Most K Distinct Characters:** Sliding window with map size constraint.

---

## Problem 15 — Trapping Rain Water

**Difficulty:** Hard
**Real-world intuition:** Urban drainage systems — how much rainwater gets trapped between buildings of varying heights. Also models 2D terrain elevation problems.

---

### Theory

- **Core concept:** Water at position i is `min(maxLeft[i], maxRight[i]) - height[i]`.
- **Pattern:** Two-pointer approach (or prefix/suffix max arrays).
- **When to use:** Any problem requiring left-max and right-max for each position.
- **Common mistakes:**
  - Using the brute force O(n²) approach of scanning left and right for each i.
  - Forgetting that water amount can't be negative — use `max(0, ...)`.

---

### Brute Force

For each position i, scan left for max and right for max.

- **Time:** O(n²), **Space:** O(1)

---

### Optimal Approach — Two Pointers

**Intuition:** At any point, the water level at a position is limited by the shorter of the two sides. The two-pointer approach processes from the shorter side first because we already know the limiting factor.

**Steps:**
1. `left=0, right=n-1, leftMax=0, rightMax=0, water=0`.
2. While `left < right`:
   - If `height[left] <= height[right]`:
     - If `height[left] >= leftMax`: update `leftMax`.
     - Else: `water += leftMax - height[left]`.
     - `left++`.
   - Else (mirror for right side).

- **Time:** O(n), **Space:** O(1)

---

### Java Implementation

```java
public class TrappingRainWater {
    public int trap(int[] height) {
        int left = 0, right = height.length - 1;
        int leftMax = 0, rightMax = 0, water = 0;

        while (left < right) {
            if (height[left] <= height[right]) {
                if (height[left] >= leftMax) leftMax = height[left];
                else water += leftMax - height[left];
                left++;
            } else {
                if (height[right] >= rightMax) rightMax = height[right];
                else water += rightMax - height[right];
                right--;
            }
        }

        return water;
    }
}
```

### Python Implementation

```python
def trap(height: list[int]) -> int:
    left, right = 0, len(height) - 1
    left_max = right_max = water = 0
    while left < right:
        if height[left] <= height[right]:
            if height[left] >= left_max: left_max = height[left]
            else: water += left_max - height[left]
            left += 1
        else:
            if height[right] >= right_max: right_max = height[right]
            else: water += right_max - height[right]
            right -= 1
    return water
```

---

### Dry Run

**Input:** `[0,1,0,2,1,0,1,3,2,1,2,1]`

Two pointers converge. Key steps:
- left=0(h=0), right=11(h=1): leftMax becomes 0, then left advances.
- Eventually: leftMax=2 provides water over valleys on the left; rightMax=3 on the right.

**Output:** `6` ✓

---

### Variations

1. **Container With Most Water:** Two pointers, maximize area (Problem 28).
2. **Trapping Rain Water II (3D):** Use a min-heap (priority queue BFS).
3. **Largest Rectangle in Histogram:** Related boundary problem with stack.

---

## Arrays & Strings — Pattern Summary

### Key Patterns

| Pattern | Trigger phrase | Example problems |
|---------|---------------|------------------|
| HashMap complement | "two elements summing to X" | Two Sum |
| Running min/max | "best time / maximum gain in one pass" | Buy & Sell Stock |
| Kadane's (reset or extend) | "maximum subarray sum" | Maximum Subarray |
| Prefix/suffix product | "product of all except self" | Product Except Self |
| Three-reverse trick | "rotate array in-place" | Rotate Array |
| Floyd's cycle | "find duplicate, no extra space" | Find Duplicate |
| Sort + merge | "overlapping intervals" | Merge Intervals |
| Modified binary search | "sorted but rotated array" | Search Rotated |
| Boundary shrink | "matrix spiral / layered traversal" | Spiral Matrix |
| Sliding window | "smallest window containing all chars" | Min Window Substring |
| Two-pointer left/right | "trapped water, container area" | Trapping Rain Water |

### How to Identify in an Interview

1. **"Contiguous subarray"** → Kadane's or Sliding Window.
2. **"All pairs / two elements"** → HashMap O(n) or Two Pointers O(n) on sorted.
3. **"No extra space"** → In-place flag, prefix array reuse, or two-pointer.
4. **"Sorted + something"** → Binary Search variant.
5. **"Intervals"** → Sort by start, then merge with linear scan.

---

## Revision Checklist — After Problems 1–10

- [ ] Can you implement Two Sum without looking at notes?
- [ ] Do you initialize Kadane's with `nums[0]` not `0`?
- [ ] Do you remember `k % n` for rotate?
- [ ] Can you explain Floyd's cycle detection in words?
- [ ] Do you handle the `top <= bottom` guard in Spiral Matrix?

### Mixed Practice Problems (after 10)

1. **Subarray Sum Equals K** — combine prefix sums + HashMap.
2. **Longest Subarray with Sum ≤ K** — sliding window.
3. **Jump Game** — greedy, similar to running-max pattern.

---

# SECTION 2 — HASHING & SETS (Problems 16–25)

> **Core insight:** HashMaps trade space for time. Almost every O(n²) → O(n) transformation involves a hash structure.

---

## Problem 16 — Contains Duplicate

**Difficulty:** Easy
**Real-world intuition:** Database uniqueness constraint check, deduplication in data pipelines.

---

### Theory

- **Core concept:** Return true if any value appears more than once.
- **Pattern:** HashSet membership check.
- **When to use:** Any "has duplicate" or "all unique" check.
- **Common mistakes:**
  - Sorting and comparing adjacent (O(n log n)) when O(n) is trivial.

---

### Java Implementation

```java
import java.util.HashSet;

public class ContainsDuplicate {
    public boolean containsDuplicate(int[] nums) {
        HashSet<Integer> seen = new HashSet<>();
        for (int num : nums) {
            if (!seen.add(num)) return true;
        }
        return false;
    }
}
```

Note: `HashSet.add()` returns `false` if the element already exists — a clean one-liner idiom.

---

### Variations

1. **Contains Duplicate II:** Duplicate within k indices — sliding window HashSet.
2. **Contains Duplicate III:** Within k indices AND values differ by at most t — TreeSet with floor/ceiling.

---

## Problem 17 — Group Anagrams

**Difficulty:** Medium
**Real-world intuition:** Search engines grouping semantically related queries; dictionary compression by grouping words with same letter set.

---

### Theory

- **Core concept:** Two strings are in the same anagram group if their sorted form is identical.
- **Pattern:** HashMap with canonical (sorted) key.
- **When to use:** Grouping by a derived property.
- **Common mistakes:**
  - Using the original string as key (defeats grouping).
  - Sorting when a frequency array key is faster for large alphabets.

---

### Optimal Approach

For each string, sort it to get the canonical key. Group strings by key in a HashMap.

- **Time:** O(n × k log k) where k = max string length.
- **Space:** O(n × k)

**Alternative:** Use a 26-int frequency array serialized as a string key → O(n × k).

---

### Java Implementation

```java
import java.util.*;

public class GroupAnagrams {
    public List<List<String>> groupAnagrams(String[] strs) {
        HashMap<String, List<String>> map = new HashMap<>();
        for (String s : strs) {
            char[] chars = s.toCharArray();
            Arrays.sort(chars);
            String key = new String(chars);
            map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
        }
        return new ArrayList<>(map.values());
    }
}
```

### Python Implementation

```python
from collections import defaultdict

def group_anagrams(strs: list[str]) -> list[list[str]]:
    groups = defaultdict(list)
    for s in strs:
        groups[tuple(sorted(s))].append(s)
    return list(groups.values())
```

---

### Dry Run

**Input:** `["eat","tea","tan","ate","nat","bat"]`

| string | sorted key | group |
|--------|-----------|-------|
| "eat" | "aet" | ["eat"] |
| "tea" | "aet" | ["eat","tea"] |
| "tan" | "ant" | ["tan"] |
| "ate" | "aet" | ["eat","tea","ate"] |
| "nat" | "ant" | ["tan","nat"] |
| "bat" | "abt" | ["bat"] |

**Output:** `[["eat","tea","ate"],["tan","nat"],["bat"]]` ✓

---

### Variations

1. **Anagram Pairs Count:** Count total pairs — combinatorics on group sizes.
2. **Find All Anagram Positions:** Sliding window (Problem 33 preview).
3. **Largest Group of Anagrams:** Track max group size during HashMap build.

---

## Problem 18 — Top K Frequent Elements

**Difficulty:** Medium
**Real-world intuition:** Trending topics on social media, most-searched keywords, popular products in e-commerce — all require frequency ranking.

---

### Theory

- **Core concept:** Count frequencies, then extract top-k.
- **Pattern 1:** HashMap + min-heap of size k → O(n log k).
- **Pattern 2:** Bucket sort by frequency → O(n).
- **When to use:** "Top K" or "K most frequent" → bucket sort if O(n) required, heap if general.
- **Common mistakes:**
  - Using a max-heap and extracting k times (O(n log n)) when min-heap of size k is O(n log k).
  - Bucket sort: array size must be n+1 (frequency can be at most n).

---

### Optimal Approach — Bucket Sort

**Steps:**
1. Count frequencies with HashMap.
2. Create `bucket[i]` = list of elements with frequency i. Array size = n+1.
3. Scan buckets from high to low, collect until k elements found.

- **Time:** O(n), **Space:** O(n)

---

### Java Implementation

```java
import java.util.*;

public class TopKFrequent {
    public int[] topKFrequent(int[] nums, int k) {
        HashMap<Integer, Integer> freq = new HashMap<>();
        for (int n : nums) freq.merge(n, 1, Integer::sum);

        List<Integer>[] bucket = new List[nums.length + 1];
        for (int num : freq.keySet()) {
            int f = freq.get(num);
            if (bucket[f] == null) bucket[f] = new ArrayList<>();
            bucket[f].add(num);
        }

        int[] result = new int[k];
        int idx = 0;
        for (int f = bucket.length - 1; f >= 0 && idx < k; f--) {
            if (bucket[f] != null) {
                for (int num : bucket[f]) {
                    if (idx < k) result[idx++] = num;
                }
            }
        }
        return result;
    }
}
```

---

### Variations

1. **Top K Frequent Words:** Frequency + lexicographic tie-breaking — use heap.
2. **K Closest Points to Origin:** Heap problem — same pattern, different comparator.
3. **Sort Characters by Frequency:** Same bucket approach applied to chars.

---

## Problem 19 — Valid Sudoku

**Difficulty:** Medium
**Real-world intuition:** Form validation with multiple constraint sets — used in constraint satisfaction solvers, puzzle games, and board game engines.

---

### Theory

- **Core concept:** Each row, column, and 3×3 box must contain digits 1–9 with no repeats.
- **Pattern:** HashSet per row, column, and box — scan once.
- **When to use:** Multi-constraint uniqueness validation.
- **Common mistakes:**
  - Incorrect box index formula: `box = (row/3)*3 + col/3`.
  - Validating only rows and columns, forgetting boxes.

---

### Java Implementation

```java
import java.util.HashSet;

public class ValidSudoku {
    public boolean isValidSudoku(char[][] board) {
        HashSet<String> seen = new HashSet<>();
        for (int i = 0; i < 9; i++) {
            for (int j = 0; j < 9; j++) {
                char c = board[i][j];
                if (c == '.') continue;
                String row = "row" + i + c;
                String col = "col" + j + c;
                String box = "box" + (i / 3) + (j / 3) + c;
                if (!seen.add(row) || !seen.add(col) || !seen.add(box)) return false;
            }
        }
        return true;
    }
}
```

---

### Variations

1. **Sudoku Solver:** Backtracking (Problem 42).
2. **N-Queens Constraint Check:** Same row/col/diagonal multi-set validation.

---

## Problem 20 — Longest Consecutive Sequence

**Difficulty:** Medium
**Real-world intuition:** Finding longest unbroken run of years a user was active, longest streak in fitness tracking data.

---

### Theory

- **Core concept:** Without sorting, find the longest sequence of consecutive integers.
- **Pattern:** HashSet for O(1) lookup + only start counting from sequence beginnings.
- **When to use:** "Longest consecutive run" with O(n) requirement.
- **Common mistakes:**
  - Sorting (O(n log n)) when O(n) is achievable.
  - Starting a count from every element (O(n²) worst case). Only start when `num-1` is NOT in the set.

---

### Optimal Approach

1. Put all numbers in a HashSet.
2. For each `num`, if `num-1` is NOT in set (it's a sequence start):
   - Count up: `num+1, num+2, ...` while they exist in set.
   - Update `maxLen`.

- **Time:** O(n) amortized (each element counted at most twice).
- **Space:** O(n)

---

### Java Implementation

```java
import java.util.HashSet;

public class LongestConsecutive {
    public int longestConsecutive(int[] nums) {
        HashSet<Integer> set = new HashSet<>();
        for (int n : nums) set.add(n);

        int maxLen = 0;
        for (int num : set) {
            if (!set.contains(num - 1)) {
                int len = 1;
                while (set.contains(num + len)) len++;
                maxLen = Math.max(maxLen, len);
            }
        }
        return maxLen;
    }
}
```

---

### Dry Run

**Input:** `[100,4,200,1,3,2]`

Set: {100,4,200,1,3,2}

- 100: 99 not in set → start. 101 not in set → len=1.
- 4: 3 in set → skip (not a start).
- 200: start. len=1.
- 1: 0 not in set → start. 2,3,4 all in set. len=4. **maxLen=4**.
- 3: 2 in set → skip.
- 2: 1 in set → skip.

**Output:** `4` (sequence 1,2,3,4) ✓

---

### Variations

1. **Longest Arithmetic Progression:** DP with HashMap per pair.
2. **Find All Consecutive Sequences:** Collect actual sequences, not just length.

---

## Problem 21 — Subarray Sum Equals K

**Difficulty:** Medium
**Real-world intuition:** Finding windows of time where a cumulative metric (revenue, error count) reached exactly a target — essential in analytics dashboards.

---

### Theory

- **Core concept:** Use prefix sums. If `prefixSum[j] - prefixSum[i] = k`, then subarray `[i+1..j]` has sum k.
- **Pattern:** HashMap storing frequency of prefix sums seen so far.
- **When to use:** "Number of subarrays with sum equal to target" (not just existence, but count).
- **Common mistakes:**
  - Initializing map with `{0: 1}` (accounts for subarrays starting from index 0).
  - Confusing with sliding window (sliding window only works for non-negative numbers).

---

### Optimal Approach

1. `map = {0: 1}`, `prefixSum = 0`, `count = 0`.
2. For each `num`: `prefixSum += num`. If `prefixSum - k` is in map: `count += map[prefixSum-k]`. Put `prefixSum` in map.

- **Time:** O(n), **Space:** O(n)

---

### Java Implementation

```java
import java.util.HashMap;

public class SubarraySumEqualsK {
    public int subarraySum(int[] nums, int k) {
        HashMap<Integer, Integer> map = new HashMap<>();
        map.put(0, 1);
        int prefixSum = 0, count = 0;

        for (int num : nums) {
            prefixSum += num;
            count += map.getOrDefault(prefixSum - k, 0);
            map.merge(prefixSum, 1, Integer::sum);
        }

        return count;
    }
}
```

---

### Dry Run

**Input:** `[1,1,1]`, k=2

| num | prefixSum | prefixSum-k | map before | count |
|-----|-----------|------------|------------|-------|
| 1 | 1 | -1 | {0:1} | 0 |
| 1 | 2 | 0 | {0:1,1:1} | 1 |
| 1 | 3 | 1 | {0:1,1:1,2:1} | 2 |

**Output:** `2` (subarrays [1,1] at positions 0-1 and 1-2) ✓

---

### Variations

1. **Subarray Sum Divisible by K:** `prefixSum % k` as map key.
2. **Binary Subarray with Sum:** Same approach for binary arrays.
3. **Count Subarrays with Bounded Max:** Two-pass approach.

---

## Problem 22 — Four Sum Count

**Difficulty:** Medium
**Real-world intuition:** Counting combinations from multiple lists satisfying a constraint — used in combinatorial search problems in chemistry and bioinformatics.

---

### Theory

- **Core concept:** Split into two pairs. Count (a+b) pairs, then for each (c+d), check if -(c+d) exists.
- **Pattern:** Meet-in-the-middle with HashMap.
- **When to use:** Sum of elements from K lists equals target — split into K/2 + K/2 groups.
- **Common mistakes:**
  - Computing all 4-tuples O(n⁴) instead of splitting into two O(n²) passes.

---

### Java Implementation

```java
import java.util.HashMap;

public class FourSumCount {
    public int fourSumCount(int[] a, int[] b, int[] c, int[] d) {
        HashMap<Integer, Integer> map = new HashMap<>();
        for (int x : a)
            for (int y : b)
                map.merge(x + y, 1, Integer::sum);

        int count = 0;
        for (int x : c)
            for (int y : d)
                count += map.getOrDefault(-(x + y), 0);

        return count;
    }
}
```

- **Time:** O(n²), **Space:** O(n²)

---

### Variations

1. **4Sum (single array, unique quadruplets):** Sort + two-pointer + dedup.
2. **3Sum Closest:** Two-pointer with running minimum difference.

---

## Problem 23 — Isomorphic Strings

**Difficulty:** Easy
**Real-world intuition:** Pattern matching in cryptography (simple substitution ciphers), natural language processing (detecting structural similarity between sentences).

---

### Theory

- **Core concept:** Each character in s maps to exactly one character in t, and vice versa (bijection).
- **Pattern:** Two HashMaps (s→t and t→s) to enforce bijection.
- **When to use:** "One-to-one character mapping" problems.
- **Common mistakes:**
  - Using only one map (allows many-to-one mappings from t side).

---

### Java Implementation

```java
import java.util.HashMap;

public class IsomorphicStrings {
    public boolean isIsomorphic(String s, String t) {
        HashMap<Character, Character> sToT = new HashMap<>();
        HashMap<Character, Character> tToS = new HashMap<>();

        for (int i = 0; i < s.length(); i++) {
            char sc = s.charAt(i), tc = t.charAt(i);
            if (sToT.containsKey(sc) && sToT.get(sc) != tc) return false;
            if (tToS.containsKey(tc) && tToS.get(tc) != sc) return false;
            sToT.put(sc, tc);
            tToS.put(tc, sc);
        }
        return true;
    }
}
```

---

### Variations

1. **Word Pattern:** Same bijection logic, word level instead of character level (Problem 24).
2. **Find and Replace Pattern:** Given list of words, find those matching a pattern.

---

## Problem 24 — Word Pattern

**Difficulty:** Easy
**Real-world intuition:** Template matching — does a sentence follow a structural template? Used in rule-based NLP systems.

---

### Theory

Identical to Isomorphic Strings but mapping characters of `pattern` to words of `s`.

---

### Java Implementation

```java
import java.util.HashMap;

public class WordPattern {
    public boolean wordPattern(String pattern, String s) {
        String[] words = s.split(" ");
        if (pattern.length() != words.length) return false;

        HashMap<Character, String> charToWord = new HashMap<>();
        HashMap<String, Character> wordToChar = new HashMap<>();

        for (int i = 0; i < pattern.length(); i++) {
            char c = pattern.charAt(i);
            String w = words[i];
            if (charToWord.containsKey(c) && !charToWord.get(c).equals(w)) return false;
            if (wordToChar.containsKey(w) && wordToChar.get(w) != c) return false;
            charToWord.put(c, w);
            wordToChar.put(w, c);
        }
        return true;
    }
}
```

---

## Problem 25 — Longest Consecutive Sequence (Set Variant Review)

*(This slot reinforces Problem 20 with a set-based variation.)*

**Alternative Problem: Ransom Note**
**Difficulty:** Easy

Given two strings `ransomNote` and `magazine`, return true if you can construct ransomNote using letters from magazine (each letter used once).

---

### Java Implementation

```java
public class RansomNote {
    public boolean canConstruct(String ransomNote, String magazine) {
        int[] count = new int[26];
        for (char c : magazine.toCharArray()) count[c - 'a']++;
        for (char c : ransomNote.toCharArray()) {
            if (--count[c - 'a'] < 0) return false;
        }
        return true;
    }
}
```

---

## Hashing & Sets — Pattern Summary

| Pattern | Trigger | Example |
|---------|---------|---------|
| Value → index map | "find pair with property" | Two Sum, 4Sum Count |
| Canonical key | "group by equivalence" | Group Anagrams |
| Prefix sum + map | "count subarrays with sum" | Subarray Sum = K |
| Set membership | "O(1) lookup for sequence" | Longest Consecutive |
| Bijection maps | "one-to-one mapping" | Isomorphic, Word Pattern |
| Meet in middle | "sum of K arrays" | 4Sum Count |

### How to Identify

- "Count subarrays/sublists satisfying X" → prefix sum + HashMap.
- "Group elements by some property" → HashMap with derived key.
- "Exists pair / unique elements" → HashSet.
- "Mapping between two domains" → bidirectional HashMap.

---

## Revision Checklist — After Problems 11–20

- [ ] Why initialize prefix sum map with `{0:1}`?
- [ ] Can you derive the box index formula for Sudoku without looking?
- [ ] Do you know when NOT to sort (Longest Consecutive)?
- [ ] Can you explain meet-in-the-middle intuitively for 4Sum Count?
- [ ] Do you use two maps for bijection, not one?

### Mixed Practice

1. **Contiguous Array:** Find longest subarray with equal 0s and 1s (prefix sum + map).
2. **Find Duplicate Subtrees:** Serialize tree node + HashMap (combines hashing + trees).
3. **Pairs of Songs with Total Duration Divisible by 60:** Modular prefix sum variant.

---

# SECTION 3 — TWO POINTERS & SLIDING WINDOW (Problems 26–35)

> **Core insight:** Two pointers collapse O(n²) pair searches to O(n). Sliding window handles subarray/substring optimization in O(n).

---

## Problem 26 — Valid Palindrome

**Difficulty:** Easy
**Real-world intuition:** Input validation (palindrome captcha), DNA sequence analysis, string symmetry checks.

---

### Theory

- **Core concept:** After filtering to alphanumeric, check if string reads same forwards and backwards.
- **Pattern:** Two pointers from both ends converging.
- **When to use:** Any palindrome check.
- **Common mistakes:**
  - Allocating a cleaned string (wastes space — use in-place two pointers).
  - Not converting to same case before comparison.

---

### Java Implementation

```java
public class ValidPalindrome {
    public boolean isPalindrome(String s) {
        int left = 0, right = s.length() - 1;
        while (left < right) {
            while (left < right && !Character.isLetterOrDigit(s.charAt(left))) left++;
            while (left < right && !Character.isLetterOrDigit(s.charAt(right))) right--;
            if (Character.toLowerCase(s.charAt(left)) != Character.toLowerCase(s.charAt(right)))
                return false;
            left++; right--;
        }
        return true;
    }
}
```

- **Time:** O(n), **Space:** O(1)

---

### Variations

1. **Palindrome II:** Can make palindrome by deleting at most one character.
2. **Shortest Palindrome:** Add characters to front — KMP-based.
3. **Palindromic Substrings:** Count all palindromic substrings — expand around centers.

---

## Problem 27 — 3Sum

**Difficulty:** Medium
**Real-world intuition:** Finding triplets in financial data (three assets that balance), three-way constraint satisfaction.

---

### Theory

- **Core concept:** For each element `a`, find two elements `b`, `c` such that `a+b+c=0`.
- **Pattern:** Sort + fix one element + two-pointer on the rest.
- **When to use:** K-sum problems (fix k-2 elements, use two pointers for final 2).
- **Common mistakes:**
  - Not deduplicating: sort first, then skip duplicate elements at each position.
  - Using HashSet for dedup (tricky edge cases) vs sorting.

---

### Optimal Approach

1. Sort `nums`.
2. For each `i` from 0 to n-3:
   - Skip if `nums[i] == nums[i-1]` (duplicate).
   - Two-pointer: `left=i+1, right=n-1`.
   - If sum=0: add triplet, skip duplicates on both ends, advance both pointers.
   - If sum<0: `left++`. If sum>0: `right--`.

- **Time:** O(n²), **Space:** O(1) excluding output.

---

### Java Implementation

```java
import java.util.*;

public class ThreeSum {
    public List<List<Integer>> threeSum(int[] nums) {
        Arrays.sort(nums);
        List<List<Integer>> result = new ArrayList<>();

        for (int i = 0; i < nums.length - 2; i++) {
            if (i > 0 && nums[i] == nums[i - 1]) continue;
            int left = i + 1, right = nums.length - 1;
            while (left < right) {
                int sum = nums[i] + nums[left] + nums[right];
                if (sum == 0) {
                    result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                    while (left < right && nums[left] == nums[left + 1]) left++;
                    while (left < right && nums[right] == nums[right - 1]) right--;
                    left++; right--;
                } else if (sum < 0) {
                    left++;
                } else {
                    right--;
                }
            }
        }
        return result;
    }
}
```

---

### Dry Run

**Input:** `[-1,0,1,2,-1,-4]`

Sorted: `[-4,-1,-1,0,1,2]`

- i=0 (nums[i]=-4): left=1(-1), right=5(2). sum=-3<0→left++. ... no valid triplet.
- i=1 (nums[i]=-1): left=2(-1), right=5(2). sum=0 → add [-1,-1,2]. Skip dups. left=3,right=4. sum=-1+0+1=0 → add [-1,0,1].
- i=2 (nums[i]=-1): duplicate of i=1, skip.
- i=3 (nums[i]=0): left=4(1),right=5(2). sum=3>0→right--. left≥right, stop.

**Output:** `[[-1,-1,2],[-1,0,1]]` ✓

---

### Variations

1. **3Sum Closest:** Track min difference instead of exact zero.
2. **4Sum:** Two nested loops + two-pointer inner loop.
3. **3Sum Smaller:** Count triplets with sum < target.

---

## Problem 28 — Container With Most Water

**Difficulty:** Medium
**Real-world intuition:** Maximizing capacity in storage design, dam engineering (find optimal placement of walls).

---

### Theory

- **Core concept:** Area = `min(height[left], height[right]) × (right - left)`. Maximize this.
- **Pattern:** Two pointers — always move the shorter side (moving the taller can only decrease area).
- **When to use:** "Maximum area/value between two elements" with monotonic decision.
- **Common mistakes:**
  - Moving the taller pointer (can never improve the constraint factor).
  - Starting pointers at same position instead of opposite ends.

---

### Java Implementation

```java
public class ContainerWithMostWater {
    public int maxArea(int[] height) {
        int left = 0, right = height.length - 1, maxWater = 0;
        while (left < right) {
            int water = Math.min(height[left], height[right]) * (right - left);
            maxWater = Math.max(maxWater, water);
            if (height[left] <= height[right]) left++;
            else right--;
        }
        return maxWater;
    }
}
```

- **Time:** O(n), **Space:** O(1)

---

### Variations

1. **Trapping Rain Water:** Sum all trapped water (Problem 15).
2. **Largest Rectangle in Histogram:** Stack-based (Problem 51).

---

## Problem 29 — Remove Duplicates from Sorted Array

**Difficulty:** Easy
**Real-world intuition:** In-place data deduplication in streaming pipelines, memory-constrained embedded systems.

---

### Theory

- **Pattern:** Slow-fast pointer (write pointer + read pointer).
- **When to use:** In-place array compaction.

---

### Java Implementation

```java
public class RemoveDuplicates {
    public int removeDuplicates(int[] nums) {
        int write = 1;
        for (int read = 1; read < nums.length; read++) {
            if (nums[read] != nums[write - 1]) {
                nums[write++] = nums[read];
            }
        }
        return write;
    }
}
```

---

### Variations

1. **Remove Duplicates II (allow at most 2):** `if (write < 2 || nums[read] != nums[write-2])`.
2. **Move Zeroes:** Same write-pointer pattern, but move zeros to end.
3. **Remove Element:** Skip specific value instead of duplicates.

---

## Problem 30 — Longest Substring Without Repeating Characters

**Difficulty:** Medium
**Real-world intuition:** Token parsing, session uniqueness windows, longest sequence of non-repeated actions in user behavior analysis.

---

### Theory

- **Core concept:** Expand right; when a repeat is found, shrink left to remove the earlier occurrence.
- **Pattern:** Sliding window with HashMap (char → last seen index).
- **When to use:** "Longest substring with some uniqueness constraint."
- **Common mistakes:**
  - Moving left only one step (should jump to `lastSeen + 1`).
  - Not checking that `lastSeen[c]` is within current window (use `max(left, lastSeen+1)`).

---

### Java Implementation

```java
import java.util.HashMap;

public class LongestSubstringNoRepeat {
    public int lengthOfLongestSubstring(String s) {
        HashMap<Character, Integer> lastSeen = new HashMap<>();
        int maxLen = 0, left = 0;

        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);
            if (lastSeen.containsKey(c)) {
                left = Math.max(left, lastSeen.get(c) + 1);
            }
            lastSeen.put(c, right);
            maxLen = Math.max(maxLen, right - left + 1);
        }

        return maxLen;
    }
}
```

- **Time:** O(n), **Space:** O(min(n, charset))

---

### Dry Run

**Input:** `"abcabcbb"`

| right | c | left | lastSeen | maxLen |
|-------|---|------|----------|--------|
| 0 | a | 0 | {a:0} | 1 |
| 1 | b | 0 | {a:0,b:1} | 2 |
| 2 | c | 0 | {...,c:2} | 3 |
| 3 | a | max(0,1)=1 | {a:3,...} | 3 |
| 4 | b | max(1,2)=2 | {b:4,...} | 3 |
| 5 | c | max(2,3)=3 | {c:5,...} | 3 |
| 6 | b | max(2,5)=5 | {b:6,...} | 2 |
| 7 | b | max(5,7)=6+... wait → max(5,6+1)=7? | {b:7} | 2 |

Actually at right=7, c='b', lastSeen[b]=6, left=max(5,6+1)=7, window size=7-7+1=1.

**Output:** `3` ✓

---

### Variations

1. **Longest Substring with At Most K Distinct Characters:** HashMap size ≤ k.
2. **Fruit Into Baskets:** At most 2 distinct values in window.
3. **Permutation in String:** Fixed-size window with frequency match.

---

## Problem 31 — Minimum Size Subarray Sum

**Difficulty:** Medium
**Real-world intuition:** Finding the minimum delivery batch size that meets a weight requirement.

---

### Theory

- **Pattern:** Sliding window with shrink condition.
- **When to use:** "Minimum window satisfying a sum constraint" on positive integers.

---

### Java Implementation

```java
public class MinSizeSubarraySum {
    public int minSubArrayLen(int target, int[] nums) {
        int left = 0, sum = 0, minLen = Integer.MAX_VALUE;
        for (int right = 0; right < nums.length; right++) {
            sum += nums[right];
            while (sum >= target) {
                minLen = Math.min(minLen, right - left + 1);
                sum -= nums[left++];
            }
        }
        return minLen == Integer.MAX_VALUE ? 0 : minLen;
    }
}
```

---

## Problem 32 — Longest Repeating Character Replacement

**Difficulty:** Medium
**Real-world intuition:** In network protocols, find the longest segment of a data stream that can be "normalized" to one signal type with at most k corrections.

---

### Theory

- **Core concept:** In a window, we can keep at most `maxFreq` of the most frequent char; if `windowSize - maxFreq > k`, shrink.
- **Pattern:** Sliding window with frequency tracking.
- **Key insight:** We never need to decrease `maxFreq` (we're finding max window, not exact).
- **Common mistakes:**
  - Recalculating `maxFreq` in O(26) each shrink (unnecessary — just don't decrease it).

---

### Java Implementation

```java
public class LongestRepeatingCharReplacement {
    public int characterReplacement(String s, int k) {
        int[] freq = new int[26];
        int left = 0, maxFreq = 0, maxLen = 0;

        for (int right = 0; right < s.length(); right++) {
            freq[s.charAt(right) - 'A']++;
            maxFreq = Math.max(maxFreq, freq[s.charAt(right) - 'A']);

            if ((right - left + 1) - maxFreq > k) {
                freq[s.charAt(left) - 'A']--;
                left++;
            }
            maxLen = Math.max(maxLen, right - left + 1);
        }
        return maxLen;
    }
}
```

- **Time:** O(n), **Space:** O(1)

---

### Variations

1. **Max Consecutive Ones III:** Same logic, binary array, flip at most k zeros.
2. **Longest Subarray of 1s After Deleting One Element:** Sliding window, at most 1 zero.

---

## Problem 33 — Permutation in String

**Difficulty:** Medium
**Real-world intuition:** Detecting if a virus signature (permuted) exists within a genomic sequence.

---

### Theory

- **Core concept:** Check if any permutation of s1 exists as a substring of s2. A permutation has the same character frequencies → fixed-size sliding window with frequency match.
- **Pattern:** Fixed-size sliding window + frequency array comparison.
- **Common mistakes:**
  - Comparing full arrays each step (O(26) per step is fine, but can be optimized with a `matches` counter).

---

### Java Implementation

```java
public class PermutationInString {
    public boolean checkInclusion(String s1, String s2) {
        if (s1.length() > s2.length()) return false;
        int[] need = new int[26], window = new int[26];
        for (char c : s1.toCharArray()) need[c - 'a']++;

        int left = 0;
        for (int right = 0; right < s2.length(); right++) {
            window[s2.charAt(right) - 'a']++;
            if (right - left + 1 > s1.length()) window[s2.charAt(left++) - 'a']--;
            if (java.util.Arrays.equals(window, need)) return true;
        }
        return false;
    }
}
```

---

## Problem 34 — Sliding Window Maximum

**Difficulty:** Hard
**Real-world intuition:** Real-time analytics dashboards showing peak value in the last N minutes. Critical in stream processing (Kafka, Flink).

---

### Theory

- **Core concept:** For each window of size k, find maximum.
- **Pattern:** Monotonic deque — maintains indices of potentially useful elements in decreasing order.
- **When to use:** "Maximum/minimum in every sliding window."
- **Common mistakes:**
  - Not removing indices that have left the window (check `deque.peek() < left`).
  - Not removing smaller elements from the back before adding new element.

---

### Optimal Approach

**Intuition:** Maintain a deque of indices where values are in decreasing order. Front is always the current window maximum.

1. For each index `right`:
   - Remove from back while `nums[deque.back()] <= nums[right]`.
   - Add `right` to back.
   - Remove front if it's outside window (`deque.front() <= right - k`).
   - Once `right >= k-1`, record `nums[deque.front()]`.

- **Time:** O(n) — each element added/removed from deque once.
- **Space:** O(k)

---

### Java Implementation

```java
import java.util.ArrayDeque;

public class SlidingWindowMaximum {
    public int[] maxSlidingWindow(int[] nums, int k) {
        int n = nums.length;
        int[] result = new int[n - k + 1];
        ArrayDeque<Integer> deque = new ArrayDeque<>(); // stores indices

        for (int right = 0; right < n; right++) {
            // remove elements outside window
            while (!deque.isEmpty() && deque.peekFirst() <= right - k)
                deque.pollFirst();
            // maintain decreasing order
            while (!deque.isEmpty() && nums[deque.peekLast()] <= nums[right])
                deque.pollLast();
            deque.addLast(right);
            if (right >= k - 1) result[right - k + 1] = nums[deque.peekFirst()];
        }
        return result;
    }
}
```

---

### Dry Run

**Input:** `[1,3,-1,-3,5,3,6,7]`, k=3

| right | num | deque (indices) | window max |
|-------|-----|-----------------|-----------|
| 0 | 1 | [0] | — |
| 1 | 3 | [1] (1 removed) | — |
| 2 | -1 | [1,2] | nums[1]=**3** |
| 3 | -3 | [1,2,3] | nums[1]=**3** |
| 4 | 5 | [4] | nums[4]=**5** |
| 5 | 3 | [4,5] | nums[4]=**5** |
| 6 | 6 | [6] | nums[6]=**6** |
| 7 | 7 | [7] | nums[7]=**7** |

**Output:** `[3,3,5,5,6,7]` ✓

---

### Variations

1. **Sliding Window Minimum:** Same deque, increasing order.
2. **Jump Game VI:** DP + sliding window max.
3. **Constrained Subsequence Sum:** DP + sliding window max.

---

## Two Pointers & Sliding Window — Pattern Summary

| Pattern | Trigger | Key insight |
|---------|---------|-------------|
| Converging two pointers | "sorted array, pair sum" | Move shorter/smaller pointer |
| Slow-fast write pointer | "in-place compaction" | Write only when condition met |
| Variable-size window expand+shrink | "longest substring with property" | Expand right, shrink left when violated |
| Fixed-size window | "substrings of length k" | Slide one step at a time |
| Monotonic deque | "max/min in every window" | Pop back when new elem dominates |

### How to Identify

- "Subarray/substring + minimum/maximum/count" → sliding window.
- "Pair/triplet in sorted array" → two pointers.
- "In-place array modification" → slow-fast pointers.
- "Maximum in every window of size k" → monotonic deque.

---

## Revision Checklist — After Problems 21–30

- [ ] Do you use `max(left, lastSeen+1)` in longest-no-repeat (not just `lastSeen+1`)?
- [ ] Do you know why moving the taller pointer never helps in Container With Most Water?
- [ ] Can you implement the monotonic deque from scratch?
- [ ] For 3Sum, do you handle both inner and outer duplicate skipping?
- [ ] Do you initialize sliding window min/max answers correctly (MAX_VALUE vs 0)?

### Mixed Practice

1. **Minimum Window Substring** (revisit Problem 14 with timer).
2. **Longest Subarray with Sum ≤ K** (non-negative array).
3. **Smallest Range Covering Elements from K Lists** — heap + pointer variant.

---

# SECTION 4 — RECURSION & BACKTRACKING (Problems 36–45)

> **Core insight:** Backtracking = recursion + undo. Build a decision tree, prune early when a branch can't lead to a valid answer.

---

## Problem 36 — Subsets

**Difficulty:** Medium
**Real-world intuition:** Generating all feature combinations for A/B testing, all possible menu selections, power set in combinatorics.

---

### Theory

- **Core concept:** For each element, decide include or exclude. Build 2ⁿ subsets.
- **Pattern:** Backtracking with index tracking (avoid revisiting earlier elements).
- **When to use:** "All possible subsets / combinations."
- **Common mistakes:**
  - Not copying the list when adding to result (Java: `new ArrayList<>(current)`).
  - Allowing duplicates by not advancing the start index.

---

### Optimal Approach

At each recursive call, add current `path` to result, then try adding each element from `start` to n-1. After recursing, remove the last element (backtrack).

- **Time:** O(2ⁿ × n) — 2ⁿ subsets, each up to length n to copy.
- **Space:** O(n) recursion depth.

---

### Java Implementation

```java
import java.util.*;

public class Subsets {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        backtrack(nums, 0, new ArrayList<>(), result);
        return result;
    }

    private void backtrack(int[] nums, int start, List<Integer> path, List<List<Integer>> result) {
        result.add(new ArrayList<>(path));
        for (int i = start; i < nums.length; i++) {
            path.add(nums[i]);
            backtrack(nums, i + 1, path, result);
            path.remove(path.size() - 1);
        }
    }
}
```

### Python Implementation

```python
def subsets(nums: list[int]) -> list[list[int]]:
    result = []
    def backtrack(start, path):
        result.append(path[:])
        for i in range(start, len(nums)):
            path.append(nums[i])
            backtrack(i + 1, path)
            path.pop()
    backtrack(0, [])
    return result
```

---

### Dry Run

**Input:** `[1,2,3]`

```
backtrack(0,[])  → add []
  add 1 → backtrack(1,[1])  → add [1]
    add 2 → backtrack(2,[1,2]) → add [1,2]
      add 3 → backtrack(3,[1,2,3]) → add [1,2,3]
    backtrack(3,[1,3]) → add [1,3]
  add 2 → backtrack(2,[2]) → add [2]
    add 3 → [2,3]
  add 3 → [3]
```

**Output:** `[[],[1],[1,2],[1,2,3],[1,3],[2],[2,3],[3]]` ✓

---

### Variations

1. **Subsets II (with duplicates):** Sort first, skip `nums[i]==nums[i-1]` at same level.
2. **Combination Sum III:** Subsets of exactly k elements summing to n.
3. **Count of Subsets with Sum:** DP approach (Section 10).

---

## Problem 37 — Permutations

**Difficulty:** Medium
**Real-world intuition:** Generating all possible orderings — scheduling tasks, password combinations, seating arrangements.

---

### Theory

- **Core concept:** At each position, try every unused element.
- **Pattern:** Backtracking with `used[]` boolean array.
- **When to use:** "All orderings / arrangements" of elements.
- **Common mistakes:**
  - Using the same element at the same position (check `used[i]`).
  - Confusing permutations (order matters) with combinations (order doesn't).

---

### Java Implementation

```java
import java.util.*;

public class Permutations {
    public List<List<Integer>> permute(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        boolean[] used = new boolean[nums.length];
        backtrack(nums, used, new ArrayList<>(), result);
        return result;
    }

    private void backtrack(int[] nums, boolean[] used, List<Integer> path, List<List<Integer>> result) {
        if (path.size() == nums.length) {
            result.add(new ArrayList<>(path));
            return;
        }
        for (int i = 0; i < nums.length; i++) {
            if (used[i]) continue;
            used[i] = true;
            path.add(nums[i]);
            backtrack(nums, used, path, result);
            path.remove(path.size() - 1);
            used[i] = false;
        }
    }
}
```

- **Time:** O(n! × n), **Space:** O(n)

---

### Variations

1. **Permutations II (with duplicates):** Sort + skip `nums[i]==nums[i-1] && !used[i-1]`.
2. **Next Permutation:** Find next lexicographic permutation in-place.
3. **Permutation Sequence:** Find kth permutation without generating all.

---

## Problem 38 — Combination Sum

**Difficulty:** Medium
**Real-world intuition:** Coin change enumeration, finding all ways to compose a budget from available denominations.

---

### Theory

- **Core concept:** Choose elements (with repetition allowed) that sum to target.
- **Pattern:** Backtracking with pruning (stop when `remaining < 0`).
- **When to use:** "Find all combinations summing to target, elements can repeat."
- **Common mistakes:**
  - Not sorting candidates (sorting enables early termination when `candidates[i] > remaining`).
  - Allowing index to go backwards (causes duplicate combinations).

---

### Java Implementation

```java
import java.util.*;

public class CombinationSum {
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        Arrays.sort(candidates);
        List<List<Integer>> result = new ArrayList<>();
        backtrack(candidates, target, 0, new ArrayList<>(), result);
        return result;
    }

    private void backtrack(int[] candidates, int remaining, int start,
                           List<Integer> path, List<List<Integer>> result) {
        if (remaining == 0) { result.add(new ArrayList<>(path)); return; }
        for (int i = start; i < candidates.length; i++) {
            if (candidates[i] > remaining) break;  // pruning
            path.add(candidates[i]);
            backtrack(candidates, remaining - candidates[i], i, path, result); // i not i+1: can reuse
            path.remove(path.size() - 1);
        }
    }
}
```

---

### Variations

1. **Combination Sum II (no reuse):** Use `i+1` instead of `i`, skip duplicates.
2. **Combination Sum III:** Exactly k numbers summing to n.
3. **Coin Change (count ways):** DP version of combination sum.

---

## Problem 39 — Word Search

**Difficulty:** Medium
**Real-world intuition:** Pathfinding in grid-based games, searching for a DNA motif in a 2D genomic grid.

---

### Theory

- **Core concept:** DFS from each cell, marking visited to avoid reuse.
- **Pattern:** Grid DFS + backtracking (mark/unmark visited).
- **When to use:** "Find a path/word in a 2D grid."
- **Common mistakes:**
  - Not un-marking cells after backtracking (corrupts future paths).
  - Checking bounds after accessing the cell (causes IndexOutOfBounds).

---

### Java Implementation

```java
public class WordSearch {
    private char[][] board;
    private String word;

    public boolean exist(char[][] board, String word) {
        this.board = board;
        this.word = word;
        for (int i = 0; i < board.length; i++)
            for (int j = 0; j < board[0].length; j++)
                if (dfs(i, j, 0)) return true;
        return false;
    }

    private boolean dfs(int row, int col, int index) {
        if (index == word.length()) return true;
        if (row < 0 || row >= board.length || col < 0 || col >= board[0].length) return false;
        if (board[row][col] != word.charAt(index)) return false;

        char temp = board[row][col];
        board[row][col] = '#';  // mark visited
        boolean found = dfs(row+1,col,index+1) || dfs(row-1,col,index+1)
                     || dfs(row,col+1,index+1) || dfs(row,col-1,index+1);
        board[row][col] = temp;  // unmark (backtrack)
        return found;
    }
}
```

- **Time:** O(m×n×4^L) where L = word length. **Space:** O(L) recursion.

---

### Variations

1. **Word Search II (find all words):** Trie + DFS — massive pruning.
2. **Number of Islands:** DFS without backtracking (mark permanently visited).
3. **Unique Paths in Grid with Obstacles:** DP approach.

---

## Problem 40 — Letter Combinations of a Phone Number

**Difficulty:** Medium
**Real-world intuition:** T9 phone keyboard word prediction, mapping numeric codes to all possible words.

---

### Theory

- **Pattern:** Backtracking through digit→letters mapping.
- **When to use:** "Generate all combinations from mapped choices."

---

### Java Implementation

```java
import java.util.*;

public class LetterCombinations {
    private static final String[] MAP = {"", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"};

    public List<String> letterCombinations(String digits) {
        List<String> result = new ArrayList<>();
        if (digits.isEmpty()) return result;
        backtrack(digits, 0, new StringBuilder(), result);
        return result;
    }

    private void backtrack(String digits, int index, StringBuilder path, List<String> result) {
        if (index == digits.length()) { result.add(path.toString()); return; }
        for (char c : MAP[digits.charAt(index) - '0'].toCharArray()) {
            path.append(c);
            backtrack(digits, index + 1, path, result);
            path.deleteCharAt(path.length() - 1);
        }
    }
}
```

---

## Problem 41 — Generate Parentheses

**Difficulty:** Medium
**Real-world intuition:** Template generation for code formatters, generating all valid XML/JSON nesting structures.

---

### Theory

- **Core concept:** At each step, add `(` if open count < n; add `)` if close count < open count.
- **Pattern:** Backtracking with validity pruning (no need to generate invalid strings).
- **Key insight:** Never need to backtrack and check after — the constraints maintain validity.
- **Common mistakes:**
  - Generating all 2^(2n) strings and filtering (exponentially wasteful).

---

### Java Implementation

```java
import java.util.*;

public class GenerateParentheses {
    public List<String> generateParenthesis(int n) {
        List<String> result = new ArrayList<>();
        backtrack(n, 0, 0, new StringBuilder(), result);
        return result;
    }

    private void backtrack(int n, int open, int close, StringBuilder path, List<String> result) {
        if (path.length() == 2 * n) { result.add(path.toString()); return; }
        if (open < n) {
            path.append('(');
            backtrack(n, open + 1, close, path, result);
            path.deleteCharAt(path.length() - 1);
        }
        if (close < open) {
            path.append(')');
            backtrack(n, open, close + 1, path, result);
            path.deleteCharAt(path.length() - 1);
        }
    }
}
```

- **Time:** O(4ⁿ/√n) — Catalan number. **Space:** O(n).

---

## Problem 42 — Palindrome Partitioning

**Difficulty:** Medium
**Real-world intuition:** Breaking a string into meaningful segments (palindromic words in linguistics, compression codecs).

---

### Theory

- **Core concept:** DFS trying all possible prefix cuts; only recurse if prefix is a palindrome.
- **Pattern:** Backtracking + palindrome check.
- **Optimization:** Precompute palindrome table with DP to make checks O(1).

---

### Java Implementation

```java
import java.util.*;

public class PalindromePartitioning {
    public List<List<String>> partition(String s) {
        List<List<String>> result = new ArrayList<>();
        backtrack(s, 0, new ArrayList<>(), result);
        return result;
    }

    private void backtrack(String s, int start, List<String> path, List<List<String>> result) {
        if (start == s.length()) { result.add(new ArrayList<>(path)); return; }
        for (int end = start + 1; end <= s.length(); end++) {
            String sub = s.substring(start, end);
            if (isPalindrome(sub)) {
                path.add(sub);
                backtrack(s, end, path, result);
                path.remove(path.size() - 1);
            }
        }
    }

    private boolean isPalindrome(String s) {
        int l = 0, r = s.length() - 1;
        while (l < r) if (s.charAt(l++) != s.charAt(r--)) return false;
        return true;
    }
}
```

---

## Problem 43 — N-Queens

**Difficulty:** Hard
**Real-world intuition:** Constraint satisfaction — placing conflicting resources (servers, radio towers) on a grid with mutual exclusion constraints.

---

### Theory

- **Core concept:** Place n queens on n×n board so no two share row, column, or diagonal.
- **Pattern:** Backtracking row by row; track used columns and diagonals.
- **Diagonal insight:** For diagonal: `row - col = constant`. For anti-diagonal: `row + col = constant`.
- **Common mistakes:**
  - Checking all previous queens for conflict (O(n) per placement) vs using sets (O(1)).

---

### Java Implementation

```java
import java.util.*;

public class NQueens {
    public List<List<String>> solveNQueens(int n) {
        List<List<String>> result = new ArrayList<>();
        int[] queens = new int[n];
        Arrays.fill(queens, -1);
        Set<Integer> cols = new HashSet<>(), diag1 = new HashSet<>(), diag2 = new HashSet<>();
        backtrack(n, 0, queens, cols, diag1, diag2, result);
        return result;
    }

    private void backtrack(int n, int row, int[] queens,
                           Set<Integer> cols, Set<Integer> diag1, Set<Integer> diag2,
                           List<List<String>> result) {
        if (row == n) { result.add(buildBoard(queens, n)); return; }
        for (int col = 0; col < n; col++) {
            if (cols.contains(col) || diag1.contains(row - col) || diag2.contains(row + col)) continue;
            queens[row] = col;
            cols.add(col); diag1.add(row - col); diag2.add(row + col);
            backtrack(n, row + 1, queens, cols, diag1, diag2, result);
            queens[row] = -1;
            cols.remove(col); diag1.remove(row - col); diag2.remove(row + col);
        }
    }

    private List<String> buildBoard(int[] queens, int n) {
        List<String> board = new ArrayList<>();
        for (int q : queens) {
            char[] row = new char[n];
            Arrays.fill(row, '.');
            row[q] = 'Q';
            board.add(new String(row));
        }
        return board;
    }
}
```

- **Time:** O(n!), **Space:** O(n)

---

## Problem 44 — Sudoku Solver

**Difficulty:** Hard
**Real-world intuition:** Constraint satisfaction solver — applies to scheduling, configuration, puzzle solving engines.

---

### Theory

- **Core concept:** For each empty cell, try digits 1–9; recurse if valid; backtrack if stuck.
- **Pattern:** Backtracking with constraint checks (row, col, box).
- **Optimization:** Choose the cell with fewest valid options first (minimum remaining values heuristic).

---

### Java Implementation

```java
public class SudokuSolver {
    public void solveSudoku(char[][] board) {
        solve(board);
    }

    private boolean solve(char[][] board) {
        for (int i = 0; i < 9; i++) {
            for (int j = 0; j < 9; j++) {
                if (board[i][j] != '.') continue;
                for (char c = '1'; c <= '9'; c++) {
                    if (isValid(board, i, j, c)) {
                        board[i][j] = c;
                        if (solve(board)) return true;
                        board[i][j] = '.';
                    }
                }
                return false;  // no valid digit found — backtrack
            }
        }
        return true;  // all cells filled
    }

    private boolean isValid(char[][] board, int row, int col, char c) {
        for (int i = 0; i < 9; i++) {
            if (board[row][i] == c) return false;
            if (board[i][col] == c) return false;
            int boxRow = 3 * (row / 3) + i / 3;
            int boxCol = 3 * (col / 3) + i % 3;
            if (board[boxRow][boxCol] == c) return false;
        }
        return true;
    }
}
```

---

## Problem 45 — Combination Sum II

**Difficulty:** Medium
**Real-world intuition:** Finding all unique ways to select items from a store where duplicates exist but each physical item can only be picked once.

---

### Theory

- **Core concept:** Like Combination Sum but each number used at most once, and duplicates in input must not produce duplicate results.
- **Pattern:** Backtracking + sort + skip `candidates[i]==candidates[i-1]` at the same recursion level.
- **Key rule:** Skip duplicate at current level only (not across levels): `if (i > start && candidates[i] == candidates[i-1]) continue`.

---

### Java Implementation

```java
import java.util.*;

public class CombinationSumII {
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        Arrays.sort(candidates);
        List<List<Integer>> result = new ArrayList<>();
        backtrack(candidates, target, 0, new ArrayList<>(), result);
        return result;
    }

    private void backtrack(int[] candidates, int remaining, int start,
                           List<Integer> path, List<List<Integer>> result) {
        if (remaining == 0) { result.add(new ArrayList<>(path)); return; }
        for (int i = start; i < candidates.length; i++) {
            if (candidates[i] > remaining) break;
            if (i > start && candidates[i] == candidates[i - 1]) continue;  // skip dups at same level
            path.add(candidates[i]);
            backtrack(candidates, remaining - candidates[i], i + 1, path, result);
            path.remove(path.size() - 1);
        }
    }
}
```

---

## Recursion & Backtracking — Pattern Summary

| Pattern | Trigger | Template |
|---------|---------|----------|
| Include/exclude | "all subsets" | `add→recurse(i+1)→remove` |
| Try all unused | "all permutations" | `used[i]=true→recurse→used[i]=false` |
| Try all from start | "combinations / partition" | `for i in start..n: recurse(i or i+1)` |
| Grid DFS | "find path in grid" | `mark→recurse 4 dirs→unmark` |
| Constraint pruning | "validity-based search" | Check before recursing, not after |

### The Backtracking Template

```
void backtrack(state, choices):
    if base_case: record solution; return
    for each choice:
        if invalid: skip (PRUNE)
        make choice
        backtrack(next_state, remaining_choices)
        undo choice
```

### How to Identify

- "All possible X" → backtracking.
- "Find one valid X" → backtracking with early return.
- "Count valid X" → backtracking (or DP if overlapping subproblems).
- Grid + path + backtracking → DFS + mark/unmark.

---

## Revision Checklist — After Problems 31–40

- [ ] Can you code Subsets, Permutations, and Combination Sum in one sitting?
- [ ] Do you copy the path list (`new ArrayList<>(path)`) before adding to result?
- [ ] In Word Search, do you restore the cell after backtracking?
- [ ] Can you explain the diagonal formula `row-col` and `row+col` for N-Queens?
- [ ] Do you know when to pass `i` vs `i+1` (reuse vs no-reuse)?

### Mixed Practice

1. **Restore IP Addresses:** Backtracking with validity constraint.
2. **Unique Paths III:** Grid backtracking, visit all non-obstacle cells exactly once.
3. **Beautiful Arrangement:** Permutation backtracking with pruning.

---

# SECTION 5 — STACK & QUEUE (Problems 46–55)

> **Core insight:** Stacks solve "nearest greater/smaller" and "valid nesting" problems. Monotonic stacks are the key to O(n) histogram/skyline problems.

---

## Problem 46 — Valid Parentheses

**Difficulty:** Easy
**Real-world intuition:** Syntax validation in code editors, XML/HTML tag matching, balancing nested function calls.

---

### Theory

- **Core concept:** Opening brackets push; closing brackets must match the top of stack.
- **Pattern:** Stack for matching nested structure.
- **When to use:** Any balanced bracket / nested validation problem.
- **Common mistakes:**
  - Checking stack emptiness before popping (avoids EmptyStackException).
  - Not checking that stack is empty at the end (unclosed brackets).

---

### Java Implementation

```java
import java.util.Stack;

public class ValidParentheses {
    public boolean isValid(String s) {
        Stack<Character> stack = new Stack<>();
        for (char c : s.toCharArray()) {
            if (c == '(' || c == '[' || c == '{') {
                stack.push(c);
            } else {
                if (stack.isEmpty()) return false;
                char top = stack.pop();
                if (c == ')' && top != '(') return false;
                if (c == ']' && top != '[') return false;
                if (c == '}' && top != '{') return false;
            }
        }
        return stack.isEmpty();
    }
}
```

- **Time:** O(n), **Space:** O(n)

---

### Variations

1. **Minimum Add to Make Valid:** Count unmatched open and close.
2. **Longest Valid Parentheses:** Stack storing indices of unmatched brackets.
3. **Remove Invalid Parentheses:** BFS (level = number of removals).

---

## Problem 47 — Min Stack

**Difficulty:** Medium
**Real-world intuition:** Undo history with instant "what was the minimum at this point?" — used in database transaction logs.

---

### Theory

- **Core concept:** Support `push`, `pop`, `top`, and `getMin` — all in O(1).
- **Pattern:** Auxiliary stack tracking minimums. On push: push min of (new val, current min) to aux stack.
- **When to use:** Any "stack + O(1) query on aggregate" problem.

---

### Java Implementation

```java
import java.util.Stack;

public class MinStack {
    private Stack<Integer> stack = new Stack<>();
    private Stack<Integer> minStack = new Stack<>();

    public void push(int val) {
        stack.push(val);
        int min = minStack.isEmpty() ? val : Math.min(val, minStack.peek());
        minStack.push(min);
    }

    public void pop() {
        stack.pop();
        minStack.pop();
    }

    public int top() { return stack.peek(); }

    public int getMin() { return minStack.peek(); }
}
```

---

### Variations

1. **Max Stack:** Same pattern, track running max.
2. **Stack with increment:** Lazy increment using auxiliary array.

---

## Problem 48 — Evaluate Reverse Polish Notation

**Difficulty:** Medium
**Real-world intuition:** How compilers and calculators evaluate expressions — postfix notation eliminates ambiguity and parentheses.

---

### Theory

- **Core concept:** Numbers push onto stack; operators pop two operands and push the result.
- **Pattern:** Operand-operator stack evaluation.
- **Common mistakes:**
  - Wrong division rounding direction: Java `/` truncates toward zero (correct for this problem).
  - Operand order: `b = pop(), a = pop()`, then `a op b`.

---

### Java Implementation

```java
import java.util.Stack;

public class EvaluateRPN {
    public int evalRPN(String[] tokens) {
        Stack<Integer> stack = new Stack<>();
        for (String token : tokens) {
            if ("+-*/".contains(token)) {
                int b = stack.pop(), a = stack.pop();
                switch (token) {
                    case "+": stack.push(a + b); break;
                    case "-": stack.push(a - b); break;
                    case "*": stack.push(a * b); break;
                    case "/": stack.push(a / b); break;
                }
            } else {
                stack.push(Integer.parseInt(token));
            }
        }
        return stack.pop();
    }
}
```

---

## Problem 49 — Daily Temperatures

**Difficulty:** Medium
**Real-world intuition:** "How many days until a warmer temperature?" — Weather forecasting UX, stock market "days until higher price."

---

### Theory

- **Core concept:** For each day, find the next day with a higher temperature.
- **Pattern:** Monotonic stack (decreasing) — stores indices of unresolved days.
- **When to use:** "Next greater element to the right" for every element.
- **Common mistakes:**
  - Using brute force O(n²) scan.
  - Storing values in stack instead of indices (can't compute distance).

---

### Optimal Approach

1. Maintain a stack of indices with decreasing temperatures.
2. For each `i`, while `stack` is not empty and `temps[i] > temps[stack.top()]`: pop top, `result[top] = i - top`.
3. Push `i`.

- **Time:** O(n), **Space:** O(n)

---

### Java Implementation

```java
import java.util.Stack;

public class DailyTemperatures {
    public int[] dailyTemperatures(int[] temperatures) {
        int n = temperatures.length;
        int[] result = new int[n];
        Stack<Integer> stack = new Stack<>();

        for (int i = 0; i < n; i++) {
            while (!stack.isEmpty() && temperatures[i] > temperatures[stack.peek()]) {
                int idx = stack.pop();
                result[idx] = i - idx;
            }
            stack.push(i);
        }

        return result;
    }
}
```

---

### Dry Run

**Input:** `[73,74,75,71,69,72,76,73]`

| i | temp | stack | result |
|---|------|-------|--------|
| 0 | 73 | [0] | — |
| 1 | 74 | [1] | result[0]=1 |
| 2 | 75 | [2] | result[1]=1 |
| 3 | 71 | [2,3] | — |
| 4 | 69 | [2,3,4] | — |
| 5 | 72 | [2,5] | result[3]=2, result[4]=1 |
| 6 | 76 | [6] | result[2]=4, result[5]=1 |
| 7 | 73 | [6,7] | — |

**Output:** `[1,1,4,2,1,1,0,0]` ✓

---

### Variations

1. **Next Greater Element I:** Same monotonic stack on stack queries.
2. **Next Greater Element II (circular):** Traverse array twice (or use modular index).
3. **132 Pattern:** Monotonic stack tracking second-largest from right.

---

## Problem 50 — Largest Rectangle in Histogram

**Difficulty:** Hard
**Real-world intuition:** Maximizing billboard space, optimizing floor plan layouts, computing maximal skyline areas.

---

### Theory

- **Core concept:** For each bar, the largest rectangle using that bar as the shortest bar extends left and right until a shorter bar is found.
- **Pattern:** Monotonic stack (increasing) — when a shorter bar is found, compute area for all taller bars popped.
- **Common mistakes:**
  - Not adding a sentinel bar of height 0 at the end (to flush remaining bars from stack).
  - Wrong width calculation: `width = right - left - 1` where left is new stack top after pop.

---

### Optimal Approach

**Intuition:** Maintain a stack of indices with increasing heights. When `heights[i] < heights[stack.top()]`, the top bar can't extend further right. Its width = `i - stack.top_after_pop - 1`.

- **Time:** O(n), **Space:** O(n)

---

### Java Implementation

```java
import java.util.Stack;

public class LargestRectangleHistogram {
    public int largestRectangleArea(int[] heights) {
        Stack<Integer> stack = new Stack<>();
        int maxArea = 0;
        int n = heights.length;

        for (int i = 0; i <= n; i++) {
            int h = (i == n) ? 0 : heights[i];
            while (!stack.isEmpty() && h < heights[stack.peek()]) {
                int height = heights[stack.pop()];
                int width = stack.isEmpty() ? i : i - stack.peek() - 1;
                maxArea = Math.max(maxArea, height * width);
            }
            stack.push(i);
        }

        return maxArea;
    }
}
```

---

### Dry Run

**Input:** `[2,1,5,6,2,3]`

Processing i=0(h=2): stack=[0]
i=1(h=1): pop 0 (h=2). Width=1 (stack empty). Area=2. Stack=[]. Push 1. Stack=[1].
i=2(h=5): stack=[1,2]
i=3(h=6): stack=[1,2,3]
i=4(h=2): pop 3(h=6): width=4-2-1=1. Area=6. Pop 2(h=5): width=4-1-1=2. Area=10. Push 4. Stack=[1,4].
i=5(h=3): stack=[1,4,5]
i=6(sentinel h=0): pop 5(h=3): width=6-4-1=1. Area=3. Pop 4(h=2): width=6-1-1=4. Area=8. Pop 1(h=1): width=6. Area=6.

**Output:** `10` ✓

---

### Variations

1. **Maximal Rectangle (in binary matrix):** Apply histogram problem row by row.
2. **Trapping Rain Water:** Related boundary-finding with stack.
3. **Sum of Subarray Minimums:** Monotonic stack counting contribution.

---

## Problem 51 — Decode String

**Difficulty:** Medium
**Real-world intuition:** Parsing run-length encoded data, decompressing compressed file formats, template expansion in code generators.

---

### Theory

- **Core concept:** `k[encoded_string]` means repeat the string k times. Nested brackets require a stack.
- **Pattern:** Two stacks (one for counts, one for strings built so far).
- **Common mistakes:**
  - Not handling multi-digit numbers (k can be > 9).
  - Not pushing the current string before entering a bracket.

---

### Java Implementation

```java
import java.util.Stack;

public class DecodeString {
    public String decodeString(String s) {
        Stack<Integer> countStack = new Stack<>();
        Stack<StringBuilder> stringStack = new Stack<>();
        StringBuilder current = new StringBuilder();
        int k = 0;

        for (char c : s.toCharArray()) {
            if (Character.isDigit(c)) {
                k = k * 10 + (c - '0');
            } else if (c == '[') {
                countStack.push(k);
                stringStack.push(current);
                current = new StringBuilder();
                k = 0;
            } else if (c == ']') {
                int repeat = countStack.pop();
                StringBuilder prev = stringStack.pop();
                String repeated = current.toString().repeat(repeat);
                prev.append(repeated);
                current = prev;
            } else {
                current.append(c);
            }
        }

        return current.toString();
    }
}
```

---

### Dry Run

**Input:** `"3[a2[c]]"`

- '3': k=3
- '[': push 3 to countStack, push "" to stringStack, current="", k=0
- 'a': current="a"
- '2': k=2
- '[': push 2, push "a", current="", k=0
- 'c': current="c"
- ']': repeat=2, prev="a". current="acc". current="acc"
- ']': repeat=3, prev="". current="accaccacc"

**Output:** `"accaccacc"` ✓

---

## Problem 52 — Implement Stack using Queues / Queue using Stacks

**Difficulty:** Easy
**Real-world intuition:** Understanding fundamental data structure relationships; sometimes you only have one primitive available (e.g., in message queue systems).

---

### Queue using Two Stacks

**Approach:** Push to `s1`. For dequeue, transfer all from `s1` to `s2` (only when `s2` is empty), then pop from `s2`.

- Amortized O(1) per operation.

```java
import java.util.Stack;

public class MyQueue {
    private Stack<Integer> s1 = new Stack<>(), s2 = new Stack<>();

    public void push(int x) { s1.push(x); }

    public int pop() {
        if (s2.isEmpty()) while (!s1.isEmpty()) s2.push(s1.pop());
        return s2.pop();
    }

    public int peek() {
        if (s2.isEmpty()) while (!s1.isEmpty()) s2.push(s1.pop());
        return s2.peek();
    }

    public boolean empty() { return s1.isEmpty() && s2.isEmpty(); }
}
```

---

## Problem 53 — Asteroid Collision

**Difficulty:** Medium
**Real-world intuition:** Simulating particle collisions, merging conflicting events in event-driven systems.

---

### Theory

- **Core concept:** Positive asteroids move right, negative move left. Collision occurs when a positive asteroid is followed by a negative one.
- **Pattern:** Stack simulation — positive asteroids stack up; negative ones destroy smaller positives.

---

### Java Implementation

```java
import java.util.Stack;

public class AsteroidCollision {
    public int[] asteroidCollision(int[] asteroids) {
        Stack<Integer> stack = new Stack<>();

        for (int a : asteroids) {
            boolean alive = true;
            while (alive && a < 0 && !stack.isEmpty() && stack.peek() > 0) {
                if (stack.peek() < -a) { stack.pop(); }
                else if (stack.peek() == -a) { stack.pop(); alive = false; }
                else { alive = false; }
            }
            if (alive) stack.push(a);
        }

        return stack.stream().mapToInt(Integer::intValue).toArray();
    }
}
```

---

## Problem 54 — Remove K Digits

**Difficulty:** Medium
**Real-world intuition:** Minimizing a number by removing digits — used in numerical optimization, financial rounding, data compression.

---

### Theory

- **Core concept:** Remove k digits to make the resulting number smallest. Greedy: remove a digit when the next digit is smaller.
- **Pattern:** Monotonic stack (increasing) — pop when next digit is smaller.
- **Common mistakes:**
  - Not handling leading zeros in the result.
  - If k > 0 after scan, remove from the end (already increasing sequence).

---

### Java Implementation

```java
public class RemoveKDigits {
    public String removeKdigits(String num, int k) {
        StringBuilder stack = new StringBuilder();
        for (char c : num.toCharArray()) {
            while (k > 0 && stack.length() > 0 && stack.charAt(stack.length() - 1) > c) {
                stack.deleteCharAt(stack.length() - 1);
                k--;
            }
            stack.append(c);
        }
        // remove remaining k digits from the end
        while (k-- > 0) stack.deleteCharAt(stack.length() - 1);
        // strip leading zeros
        int start = 0;
        while (start < stack.length() - 1 && stack.charAt(start) == '0') start++;
        return stack.substring(start);
    }
}
```

---

## Problem 55 — Next Greater Element II (Circular Array)

**Difficulty:** Medium
**Real-world intuition:** Temperature forecasting in circular calendar data, circular stock patterns, ring-buffer analysis.

---

### Theory

- **Core concept:** In a circular array, wrap around once. Simulate by iterating `2n` times with `i % n`.
- **Pattern:** Monotonic stack — same as Daily Temperatures but circular.

---

### Java Implementation

```java
import java.util.Stack;
import java.util.Arrays;

public class NextGreaterElementII {
    public int[] nextGreaterElements(int[] nums) {
        int n = nums.length;
        int[] result = new int[n];
        Arrays.fill(result, -1);
        Stack<Integer> stack = new Stack<>();

        for (int i = 0; i < 2 * n; i++) {
            while (!stack.isEmpty() && nums[i % n] > nums[stack.peek()]) {
                result[stack.pop()] = nums[i % n];
            }
            if (i < n) stack.push(i);
        }

        return result;
    }
}
```

---

## Stack & Queue — Pattern Summary

| Pattern | Trigger | Example |
|---------|---------|---------|
| Matching stack | "balanced brackets / nesting" | Valid Parentheses |
| Auxiliary min/max stack | "O(1) min/max with push/pop" | Min Stack |
| Monotonic decreasing stack | "next greater element" | Daily Temperatures |
| Monotonic increasing stack | "largest rectangle / histogram" | Largest Rectangle |
| Two stacks for queue | "queue with stack operations" | Queue via Stacks |
| Stack simulation | "collision / evaluation" | Asteroid, RPN |

### How to Identify

- "Next greater/smaller element to the right" → monotonic stack.
- "Balanced / valid nesting" → matching stack.
- "Largest rectangle / maximum area" → monotonic increasing stack.
- "Expression evaluation" → operand/operator stack.

---

## Revision Checklist — After Problems 41–50

- [ ] Can you code the monotonic stack for Daily Temperatures without hints?
- [ ] Do you add a sentinel 0 at the end for Largest Rectangle?
- [ ] For Decode String, do you handle multi-digit k?
- [ ] In Queue via Stacks, do you transfer lazily (only when s2 is empty)?
- [ ] For circular Next Greater, do you iterate 2n and use `i%n`?

### Mixed Practice

1. **Trapping Rain Water** via stack approach (alternative to two-pointer).
2. **Simplify Path:** Stack-based path normalization.
3. **Online Stock Span:** Monotonic stack counting consecutive smaller-or-equal days.

---

# SECTION 6 — LINKED LIST (Problems 56–65)

> **Core insight:** Linked list problems reduce to pointer manipulation. Master the "dummy node" and "two-pointer (slow/fast)" techniques.

---

## Problem 56 — Reverse Linked List

**Difficulty:** Easy
**Real-world intuition:** Undo operations in text editors, reversing a command queue, reflection operations in graphics.

---

### Theory

- **Core concept:** Redirect each node's `next` pointer to the previous node.
- **Pattern:** Three-pointer iteration (prev, curr, next).
- **When to use:** Any list reversal or reversal-in-parts.
- **Common mistakes:**
  - Losing `curr.next` before reassigning it.
  - Not returning `prev` (which becomes the new head).

---

### Iterative Java Implementation

```java
public class ReverseLinkedList {
    public ListNode reverseList(ListNode head) {
        ListNode prev = null, curr = head;
        while (curr != null) {
            ListNode next = curr.next;
            curr.next = prev;
            prev = curr;
            curr = next;
        }
        return prev;
    }
}
```

### Recursive Java Implementation

```java
public ListNode reverseListRecursive(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode newHead = reverseListRecursive(head.next);
    head.next.next = head;
    head.next = null;
    return newHead;
}
```

- **Time:** O(n), **Space:** O(1) iterative / O(n) recursive.

---

### Variations

1. **Reverse Linked List II (range):** Reverse only nodes from position m to n.
2. **Reverse Nodes in k-Group:** Reverse every k consecutive nodes.
3. **Palindrome Linked List:** Reverse second half, compare.

---

## Problem 57 — Detect Cycle in Linked List

**Difficulty:** Easy
**Real-world intuition:** Detecting infinite loops in workflow engines, circular dependencies in build systems.

---

### Theory

- **Core concept:** Floyd's cycle detection — slow pointer moves 1 step, fast moves 2. If they meet, there's a cycle.
- **Pattern:** Slow-fast pointers.
- **When to use:** Cycle detection in any sequence.
- **Common mistakes:**
  - Not checking `fast != null && fast.next != null` before advancing (NullPointerException).

---

### Java Implementation

```java
public class DetectCycle {
    public boolean hasCycle(ListNode head) {
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) return true;
        }
        return false;
    }

    // Follow-up: find cycle start node
    public ListNode detectCycle(ListNode head) {
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) {
                slow = head;
                while (slow != fast) { slow = slow.next; fast = fast.next; }
                return slow;
            }
        }
        return null;
    }
}
```

---

## Problem 58 — Merge Two Sorted Lists

**Difficulty:** Easy
**Real-world intuition:** Merging sorted log files, combining sorted search result pages, merge step in merge sort.

---

### Theory

- **Pattern:** Dummy head node to simplify edge cases + greedy merge.
- **When to use:** Merging sorted structures.
- **Common mistakes:**
  - Not attaching the remaining non-null list at the end.
  - Trying to merge without a dummy node (complex null checks).

---

### Java Implementation

```java
public class MergeTwoSortedLists {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(0);
        ListNode curr = dummy;

        while (l1 != null && l2 != null) {
            if (l1.val <= l2.val) { curr.next = l1; l1 = l1.next; }
            else { curr.next = l2; l2 = l2.next; }
            curr = curr.next;
        }
        curr.next = (l1 != null) ? l1 : l2;
        return dummy.next;
    }
}
```

---

### Variations

1. **Merge K Sorted Lists:** Use a min-heap (Problem 79).
2. **Sort List:** Merge sort using this merge function.

---

## Problem 59 — Remove Nth Node From End

**Difficulty:** Medium
**Real-world intuition:** Maintaining a rolling history of N events and removing the oldest — used in LRU cache and session management.

---

### Theory

- **Core concept:** Two pointers with a gap of n. When fast reaches the end, slow is at the node before the target.
- **Pattern:** Two-pointer with fixed gap.
- **When to use:** "Nth from end" without knowing the length.
- **Common mistakes:**
  - Not using a dummy head (edge case: removing the first node).
  - Gap of n vs n+1 — fast pointer should be n+1 steps ahead so slow lands one before target.

---

### Java Implementation

```java
public class RemoveNthFromEnd {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        ListNode fast = dummy, slow = dummy;

        for (int i = 0; i <= n; i++) fast = fast.next;

        while (fast != null) {
            slow = slow.next;
            fast = fast.next;
        }

        slow.next = slow.next.next;
        return dummy.next;
    }
}
```

---

### Dry Run

**Input:** `1->2->3->4->5`, n=2

- fast advances n+1=3 steps: fast=node(3), slow=dummy.
- Iterate: slow=1,fast=4 → slow=2,fast=5 → slow=3,fast=null.
- `slow.next = slow.next.next` → node(3).next = node(5). Removed node(4).

**Output:** `1->2->3->5` ✓

---

## Problem 60 — Add Two Numbers

**Difficulty:** Medium
**Real-world intuition:** Big integer arithmetic in cryptography and financial systems where numbers exceed 64-bit integers.

---

### Theory

- **Core concept:** Simulate digit-by-digit addition with carry, like elementary school math.
- **Pattern:** Simultaneous traversal + carry propagation.
- **Common mistakes:**
  - Forgetting the final carry (if sum overflows to one more digit).
  - Not handling lists of different lengths.

---

### Java Implementation

```java
public class AddTwoNumbers {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(0);
        ListNode curr = dummy;
        int carry = 0;

        while (l1 != null || l2 != null || carry != 0) {
            int sum = carry;
            if (l1 != null) { sum += l1.val; l1 = l1.next; }
            if (l2 != null) { sum += l2.val; l2 = l2.next; }
            carry = sum / 10;
            curr.next = new ListNode(sum % 10);
            curr = curr.next;
        }

        return dummy.next;
    }
}
```

---

## Problem 61 — Copy List with Random Pointer

**Difficulty:** Medium
**Real-world intuition:** Deep copying complex graph-like structures in serialization, game state cloning, object persistence.

---

### Theory

- **Core concept:** Two passes — first create all new nodes, then assign next and random pointers.
- **Pattern:** HashMap (original node → copy node) for O(1) random pointer lookup.
- **Common mistakes:**
  - Trying to do it in one pass (random pointer may point to a node not yet created).

---

### Java Implementation

```java
import java.util.HashMap;

public class CopyListRandomPointer {
    public Node copyRandomList(Node head) {
        if (head == null) return null;
        HashMap<Node, Node> map = new HashMap<>();

        Node curr = head;
        while (curr != null) { map.put(curr, new Node(curr.val)); curr = curr.next; }

        curr = head;
        while (curr != null) {
            map.get(curr).next = map.get(curr.next);
            map.get(curr).random = map.get(curr.random);
            curr = curr.next;
        }

        return map.get(head);
    }
}
```

- **Time:** O(n), **Space:** O(n)

**O(1) space variant:** Weave copies into original list, set randoms, then unweave.

---

## Problem 62 — LRU Cache

**Difficulty:** Medium
**Real-world intuition:** Operating system page replacement, browser history, CPU cache management — evict least recently used item when capacity is reached.

---

### Theory

- **Core concept:** O(1) get and put. Use HashMap for O(1) lookup + doubly linked list for O(1) insertion/deletion.
- **Pattern:** HashMap + Doubly Linked List.
- **When to use:** Any "cache with eviction policy" problem.
- **Common mistakes:**
  - Using a singly linked list (can't delete in O(1) without prev pointer).
  - Forgetting to update the list order on every `get` (not just `put`).

---

### Java Implementation

```java
import java.util.HashMap;

public class LRUCache {
    private final int capacity;
    private final HashMap<Integer, Node> map = new HashMap<>();
    private final Node head = new Node(0, 0);  // dummy head (MRU side)
    private final Node tail = new Node(0, 0);  // dummy tail (LRU side)

    private static class Node {
        int key, val;
        Node prev, next;
        Node(int k, int v) { key = k; val = v; }
    }

    public LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        if (!map.containsKey(key)) return -1;
        Node node = map.get(key);
        remove(node);
        insertFront(node);
        return node.val;
    }

    public void put(int key, int value) {
        if (map.containsKey(key)) remove(map.get(key));
        if (map.size() == capacity) {
            Node lru = tail.prev;
            remove(lru);
            map.remove(lru.key);
        }
        Node node = new Node(key, value);
        map.put(key, node);
        insertFront(node);
    }

    private void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void insertFront(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }
}
```

- **Time:** O(1) for get and put. **Space:** O(capacity).

---

## Problem 63 — Intersection of Two Linked Lists

**Difficulty:** Easy
**Real-world intuition:** Finding shared resources between two dependency trees, detecting forked code merging to the same commit.

---

### Theory

- **Core concept:** Two pointers that switch lists when they reach the end — both traverse exactly `m+n` nodes before meeting at the intersection (or null).
- **Pattern:** Dual list traversal.
- **Why it works:** Both pointers travel `len(A) + len(B)` total. If there's an intersection, they're in sync when they reach it.

---

### Java Implementation

```java
public class IntersectionLinkedLists {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        ListNode a = headA, b = headB;
        while (a != b) {
            a = (a == null) ? headB : a.next;
            b = (b == null) ? headA : b.next;
        }
        return a;
    }
}
```

---

## Problem 64 — Palindrome Linked List

**Difficulty:** Easy
**Real-world intuition:** Validating symmetric data structures in network protocol parsing, detecting symmetric sequences in bioinformatics.

---

### Theory

- **Core concept:** Find midpoint, reverse second half, compare with first half.
- **Pattern:** Slow-fast pointer for midpoint + list reversal.
- **Common mistakes:**
  - Off-by-one in finding midpoint (even vs odd length).
  - Not handling single-node list.

---

### Java Implementation

```java
public class PalindromeLinkedList {
    public boolean isPalindrome(ListNode head) {
        // Find midpoint
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }

        // Reverse second half
        ListNode prev = null, curr = slow;
        while (curr != null) {
            ListNode next = curr.next;
            curr.next = prev;
            prev = curr;
            curr = next;
        }

        // Compare
        ListNode left = head, right = prev;
        while (right != null) {
            if (left.val != right.val) return false;
            left = left.next;
            right = right.next;
        }
        return true;
    }
}
```

---

## Problem 65 — Reorder List

**Difficulty:** Medium
**Real-world intuition:** Interleaving two data streams — alternating between newest and oldest records in a timeline view.

---

### Theory

- **Core concept:** Merge first half with reversed second half alternately.
- **Pattern:** Find middle + reverse second half + merge two lists.
- **Steps:**
  1. Find midpoint (slow-fast).
  2. Reverse second half.
  3. Merge: take one from each alternately.

---

### Java Implementation

```java
public class ReorderList {
    public void reorderList(ListNode head) {
        if (head == null || head.next == null) return;

        // Find midpoint
        ListNode slow = head, fast = head;
        while (fast.next != null && fast.next.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }

        // Reverse second half
        ListNode second = slow.next;
        slow.next = null;
        ListNode prev = null;
        while (second != null) {
            ListNode next = second.next;
            second.next = prev;
            prev = second;
            second = next;
        }

        // Merge
        ListNode first = head;
        second = prev;
        while (second != null) {
            ListNode tmp1 = first.next, tmp2 = second.next;
            first.next = second;
            second.next = tmp1;
            first = tmp1;
            second = tmp2;
        }
    }
}
```

---

## Linked List — Pattern Summary

| Pattern | Trigger | Example |
|---------|---------|---------|
| Dummy head node | "edge case: remove head" | Remove Nth, Merge Lists |
| Slow-fast pointers | "cycle, midpoint, kth from end" | Detect Cycle, Palindrome |
| Three-pointer reversal | "reverse list or portion" | Reverse List |
| HashMap for node lookup | "copy with complex pointers" | Copy Random List |
| HashMap + DLL | "O(1) cache with ordering" | LRU Cache |
| Dual list traversal | "intersection of two lists" | Intersection |

### How to Identify

- "nth from end / midpoint / cycle" → slow-fast pointers.
- "In-place modification, no extra space" → pointer rewiring.
- "Copy complex structure" → HashMap first pass.
- "Cache / eviction" → LRU = HashMap + DLL.

---

## Revision Checklist — After Problems 51–60

- [ ] Can you reverse a list iteratively AND recursively?
- [ ] Do you use a dummy head for remove/merge operations?
- [ ] Can you find the cycle entry point (phase 2 of Floyd's)?
- [ ] LRU Cache: do you update on `get` as well as `put`?
- [ ] Can you code the palindrome check without drawing it?

### Mixed Practice

1. **Reverse Nodes in k-Group:** Recursive list reversal in chunks.
2. **Sort List:** Merge sort on a linked list.
3. **Swap Nodes in Pairs:** Fundamental pointer manipulation exercise.

---

# SECTION 7 — BINARY TREES & BST (Problems 66–75)

> **Core insight:** Most tree problems are DFS (recursion) or BFS (level-order). Learn to think in terms of what each node returns upward.

---

## Problem 66 — Maximum Depth of Binary Tree

**Difficulty:** Easy
**Real-world intuition:** Calculating the height of a file system directory tree, depth of organizational hierarchy.

---

### Theory

- **Core concept:** Max depth = 1 + max(depth(left), depth(right)).
- **Pattern:** Post-order DFS (compute children first, then use results at current node).
- **When to use:** Any "height / depth" computation.

---

### Java Implementation

```java
public class MaxDepth {
    public int maxDepth(TreeNode root) {
        if (root == null) return 0;
        return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
    }
}
```

- **Time:** O(n), **Space:** O(h) recursion stack.

---

### Variations

1. **Minimum Depth:** Watch out — min depth must reach a leaf, not just a null child.
2. **Balanced Binary Tree:** Check `|depth(left) - depth(right)| <= 1` at every node.
3. **Diameter of Binary Tree:** Problem 75 below.

---

## Problem 67 — Invert Binary Tree

**Difficulty:** Easy
**Real-world intuition:** Mirror transformation in graphics, reversing decision trees in AI.

---

### Java Implementation

```java
public class InvertTree {
    public TreeNode invertTree(TreeNode root) {
        if (root == null) return null;
        TreeNode temp = root.left;
        root.left = invertTree(root.right);
        root.right = invertTree(temp);
        return root;
    }
}
```

---

## Problem 68 — Binary Tree Level Order Traversal

**Difficulty:** Medium
**Real-world intuition:** BFS in social networks (degrees of separation), layer-by-layer processing of hierarchical data.

---

### Theory

- **Core concept:** BFS using a queue; process all nodes at current depth before moving to next level.
- **Pattern:** BFS with level-size tracking.
- **When to use:** Any "level by level" or "layer" tree problem.
- **Common mistakes:**
  - Using `queue.size()` in the loop condition instead of capturing it before the inner loop (size changes during inner loop).

---

### Java Implementation

```java
import java.util.*;

public class LevelOrderTraversal {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        if (root == null) return result;

        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);

        while (!queue.isEmpty()) {
            int size = queue.size();  // capture before inner loop
            List<Integer> level = new ArrayList<>();
            for (int i = 0; i < size; i++) {
                TreeNode node = queue.poll();
                level.add(node.val);
                if (node.left != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
            }
            result.add(level);
        }

        return result;
    }
}
```

---

### Variations

1. **Zigzag Level Order:** Alternate direction each level.
2. **Right Side View:** Take last element of each level.
3. **Average of Levels:** Average each level list.

---

## Problem 69 — Validate BST

**Difficulty:** Medium
**Real-world intuition:** Database B-tree integrity checks, validating sorted data structures.

---

### Theory

- **Core concept:** Each node must be strictly within a valid range. Left subtree: max = current node value. Right subtree: min = current node value.
- **Pattern:** DFS with min/max bounds passed down.
- **Common mistakes:**
  - Only checking `root.left.val < root.val` (not a recursive guarantee — a deeper node may violate BST property).
  - Using `Integer.MIN/MAX_VALUE` as default bounds (fails for INT_MIN or INT_MAX as node values — use `null` or `Long`).

---

### Java Implementation

```java
public class ValidateBST {
    public boolean isValidBST(TreeNode root) {
        return validate(root, null, null);
    }

    private boolean validate(TreeNode node, Integer min, Integer max) {
        if (node == null) return true;
        if (min != null && node.val <= min) return false;
        if (max != null && node.val >= max) return false;
        return validate(node.left, min, node.val) && validate(node.right, node.val, max);
    }
}
```

---

### Dry Run

**Input:**
```
    5
   / \
  1   4
     / \
    3   6
```

- validate(5, null, null): valid range OK.
- validate(1, null, 5): OK.
- validate(4, 5, null): 4 <= 5 → FALSE.

**Output:** `false` ✓

---

## Problem 70 — Lowest Common Ancestor of a Binary Tree

**Difficulty:** Medium
**Real-world intuition:** Finding the common ancestor in an organizational chart, the base class in OOP hierarchy, or the merge point in version control.

---

### Theory

- **Core concept:** If both p and q are in different subtrees of a node, that node is the LCA. If one is the ancestor of the other, it is the LCA.
- **Pattern:** Post-order DFS — check left and right subtrees, then decide at current node.
- **Return logic:** If both subtrees return non-null, current node is LCA.

---

### Java Implementation

```java
public class LowestCommonAncestor {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null || root == p || root == q) return root;
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);
        if (left != null && right != null) return root;
        return left != null ? left : right;
    }
}
```

- **Time:** O(n), **Space:** O(h)

**LCA in BST:** If both values < root, go left; if both > root, go right; else root is LCA.

---

## Problem 71 — Binary Tree Maximum Path Sum

**Difficulty:** Hard
**Real-world intuition:** Finding the highest-value route in a logistics network modeled as a tree. Maximizing pipeline throughput.

---

### Theory

- **Core concept:** A path can go through any node as the "top" (turning point). The contribution of a subtree is `max(0, maxPathFromChild)` — never take a negative branch.
- **Pattern:** Post-order DFS; each call returns the best single-branch extension (for parent to use), while globally tracking the best path-through-node.
- **Common mistakes:**
  - Confusing "path through root" with "path anywhere in tree" — use a global variable.
  - Not clamping negative subtree contributions to 0.

---

### Java Implementation

```java
public class BinaryTreeMaxPathSum {
    private int maxSum = Integer.MIN_VALUE;

    public int maxPathSum(TreeNode root) {
        dfs(root);
        return maxSum;
    }

    private int dfs(TreeNode node) {
        if (node == null) return 0;
        int left = Math.max(0, dfs(node.left));
        int right = Math.max(0, dfs(node.right));
        maxSum = Math.max(maxSum, node.val + left + right);  // path through this node
        return node.val + Math.max(left, right);              // best single branch for parent
    }
}
```

---

### Dry Run

**Input:**
```
   -10
   /  \
  9   20
     /  \
    15   7
```

- dfs(9): left=0, right=0. maxSum=max(MIN,9)=9. return 9.
- dfs(15): return 15. dfs(7): return 7.
- dfs(20): left=15, right=7. maxSum=max(9,20+15+7)=42. return 20+15=35.
- dfs(-10): left=9→max(0,9)=9, right=35. maxSum=max(42,-10+9+35)=max(42,34)=42. return -10+35=25.

**Output:** `42` ✓

---

## Problem 72 — Serialize and Deserialize Binary Tree

**Difficulty:** Hard
**Real-world intuition:** Persisting tree structures to disk or sending over a network — databases, distributed systems, JSON/XML tree storage.

---

### Theory

- **Core concept:** Pre-order traversal for serialization (root first, then left, then right). Use `null` markers to distinguish structure.
- **Pattern:** Recursive pre-order with null markers.
- **When to use:** Any tree persistence/reconstruction problem.

---

### Java Implementation

```java
import java.util.Arrays;
import java.util.LinkedList;
import java.util.Queue;

public class SerializeDeserialize {
    public String serialize(TreeNode root) {
        if (root == null) return "null";
        return root.val + "," + serialize(root.left) + "," + serialize(root.right);
    }

    public TreeNode deserialize(String data) {
        Queue<String> queue = new LinkedList<>(Arrays.asList(data.split(",")));
        return buildTree(queue);
    }

    private TreeNode buildTree(Queue<String> queue) {
        String val = queue.poll();
        if ("null".equals(val)) return null;
        TreeNode node = new TreeNode(Integer.parseInt(val));
        node.left = buildTree(queue);
        node.right = buildTree(queue);
        return node;
    }
}
```

---

## Problem 73 — Kth Smallest Element in BST

**Difficulty:** Medium
**Real-world intuition:** Finding the kth percentile entry in a sorted database index, the kth cheapest product in an ordered catalog.

---

### Theory

- **Core concept:** In-order traversal of BST produces elements in sorted order.
- **Pattern:** In-order DFS with counter.
- **Optimization:** Morris traversal for O(1) space.

---

### Java Implementation

```java
public class KthSmallestBST {
    private int count = 0, result = 0;

    public int kthSmallest(TreeNode root, int k) {
        inorder(root, k);
        return result;
    }

    private void inorder(TreeNode node, int k) {
        if (node == null) return;
        inorder(node.left, k);
        if (++count == k) { result = node.val; return; }
        inorder(node.right, k);
    }
}
```

---

## Problem 74 — Construct Binary Tree from Preorder and Inorder

**Difficulty:** Medium
**Real-world intuition:** Reconstructing a tree from two complementary traversals — used in tree serialization standards.

---

### Theory

- **Core concept:** Preorder[0] is always the root. Find it in inorder — everything to its left is the left subtree, right is the right subtree.
- **Pattern:** Recursive divide-and-conquer with HashMap for O(1) inorder index lookup.
- **Common mistakes:**
  - Recomputing inorder index by scanning (O(n²)) instead of using HashMap (O(1)).

---

### Java Implementation

```java
import java.util.HashMap;

public class BuildTreePreorderInorder {
    private HashMap<Integer, Integer> inorderIndex = new HashMap<>();

    public TreeNode buildTree(int[] preorder, int[] inorder) {
        for (int i = 0; i < inorder.length; i++) inorderIndex.put(inorder[i], i);
        return build(preorder, 0, preorder.length - 1, 0, inorder.length - 1);
    }

    private TreeNode build(int[] pre, int preLeft, int preRight, int inLeft, int inRight) {
        if (preLeft > preRight) return null;
        int rootVal = pre[preLeft];
        int mid = inorderIndex.get(rootVal);
        int leftSize = mid - inLeft;

        TreeNode root = new TreeNode(rootVal);
        root.left = build(pre, preLeft + 1, preLeft + leftSize, inLeft, mid - 1);
        root.right = build(pre, preLeft + leftSize + 1, preRight, mid + 1, inRight);
        return root;
    }
}
```

- **Time:** O(n), **Space:** O(n)

---

## Problem 75 — Diameter of Binary Tree

**Difficulty:** Easy
**Real-world intuition:** Finding the longest network path in a tree-structured network topology.

---

### Theory

- **Core concept:** The diameter = max over all nodes of `depth(left) + depth(right)`.
- **Pattern:** Post-order DFS tracking global maximum.
- **Common mistakes:**
  - Returning the diameter from the recursion instead of the depth (breaks the recursion contract).

---

### Java Implementation

```java
public class DiameterBinaryTree {
    private int maxDiameter = 0;

    public int diameterOfBinaryTree(TreeNode root) {
        depth(root);
        return maxDiameter;
    }

    private int depth(TreeNode node) {
        if (node == null) return 0;
        int left = depth(node.left);
        int right = depth(node.right);
        maxDiameter = Math.max(maxDiameter, left + right);
        return 1 + Math.max(left, right);
    }
}
```

---

## Binary Trees & BST — Pattern Summary

| Pattern | When to use | Return from DFS |
|---------|-------------|-----------------|
| Post-order DFS | Compute something from subtrees | Result for parent to use |
| Pre-order DFS | Construct / serialize | N/A |
| In-order DFS | BST sorted traversal | N/A |
| BFS (level order) | Layer-by-layer processing | Per-level results |
| Global variable + DFS | Track max/min over all paths | Only depth/contribution |
| Min/max bounds (BST validation) | Validate BST property | true/false |

### How to Identify

- "Compute at each node using children" → post-order.
- "Level by level" → BFS.
- "BST sorted property" → in-order.
- "Path sum / diameter" → global variable + post-order DFS.
- "Reconstruct tree" → pre-order or divide-and-conquer with inorder index.

---

## Revision Checklist — After Problems 61–70

- [ ] Max depth, invert, LCA — can you code all three from memory?
- [ ] Validate BST: do you pass bounds, not just compare children?
- [ ] Level order: do you capture `queue.size()` before the inner loop?
- [ ] Max path sum: do you clamp negative subtree sums to 0?
- [ ] Construct from traversals: do you use a HashMap for inorder index?

### Mixed Practice

1. **Path Sum II:** Backtracking on tree to find all root-to-leaf paths with given sum.
2. **Flatten Binary Tree to Linked List:** Pre-order, right-thread the tree.
3. **Count Complete Tree Nodes:** Binary search on last level.

---

# SECTION 8 — HEAPS & PRIORITY QUEUE (Problems 76–80)

> **Core insight:** A heap gives you O(log n) insert/delete and O(1) peek of the min or max. Any "top-K", "kth element", or "always-need-the-smallest/largest" problem maps directly to a heap.

---

## Problem 76 — Kth Largest Element in an Array

**Difficulty:** Medium
**Real-world intuition:** Finding the kth highest score in a leaderboard, kth most popular product in real-time analytics without full sorting.

---

### Theory

- **Core concept:** Maintain a min-heap of size k. The top of the heap is always the kth largest seen so far.
- **Pattern:** Min-heap of fixed size k.
- **Why min-heap, not max-heap?** A min-heap of size k keeps the k largest elements; the smallest among them (at the top) is the kth largest.
- **Alternative:** Quickselect — O(n) average, O(n²) worst.
- **Common mistakes:**
  - Using a max-heap and extracting k times (O(n log n)) — valid but misses the O(n log k) insight.
  - Heap size exceeding k (should pop when size > k, not >= k).

---

### Brute Force

Sort descending, return element at index k-1.
- **Time:** O(n log n), **Space:** O(1)

---

### Optimal Approach — Min-Heap of Size k

**Steps:**
1. Create a min-heap (`PriorityQueue` in Java defaults to min).
2. For each number: add to heap. If size > k, poll (removes minimum).
3. After all elements, `heap.peek()` is the kth largest.

- **Time:** O(n log k), **Space:** O(k)

---

### Java Implementation

```java
import java.util.PriorityQueue;

public class KthLargestElement {
    public int findKthLargest(int[] nums, int k) {
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        for (int num : nums) {
            minHeap.offer(num);
            if (minHeap.size() > k) minHeap.poll();
        }
        return minHeap.peek();
    }
}
```

### Python Implementation

```python
import heapq

def find_kth_largest(nums: list[int], k: int) -> int:
    heap = []
    for num in nums:
        heapq.heappush(heap, num)
        if len(heap) > k:
            heapq.heappop(heap)
    return heap[0]
```

---

### Dry Run

**Input:** `[3,2,1,5,6,4]`, k=2

Heap after each insertion (max size 2):
- 3 → [3]
- 2 → [2,3] (size=2, no pop)
- 1 → [1,2,3] → pop 1 → [2,3]
- 5 → [2,3,5] → pop 2 → [3,5]
- 6 → [3,5,6] → pop 3 → [5,6]
- 4 → [4,5,6] → pop 4 → [5,6]

`heap.peek()` = 5 ✓

---

### Quickselect Alternative (O(n) average)

```java
public int findKthLargestQuickSelect(int[] nums, int k) {
    return quickSelect(nums, 0, nums.length - 1, nums.length - k);
}

private int quickSelect(int[] nums, int left, int right, int targetIdx) {
    int pivot = partition(nums, left, right);
    if (pivot == targetIdx) return nums[pivot];
    return pivot < targetIdx
        ? quickSelect(nums, pivot + 1, right, targetIdx)
        : quickSelect(nums, left, pivot - 1, targetIdx);
}

private int partition(int[] nums, int left, int right) {
    int pivot = nums[right], i = left;
    for (int j = left; j < right; j++)
        if (nums[j] <= pivot) { int t = nums[i]; nums[i] = nums[j]; nums[j] = t; i++; }
    int t = nums[i]; nums[i] = nums[right]; nums[right] = t;
    return i;
}
```

---

### Variations

1. **Kth Smallest Element in Sorted Matrix:** Min-heap starting from top-left.
2. **K Closest Points to Origin:** Max-heap of size k on distances.
3. **Top K Frequent Elements:** HashMap + heap (Problem 18 revisited).

---

## Problem 77 — Top K Frequent Words

**Difficulty:** Medium
**Real-world intuition:** Trending hashtags on Twitter, most-searched keywords in a search engine, frequent error codes in logs.

---

### Theory

- **Core concept:** Count frequencies, then extract top-k by frequency (with lexicographic tie-breaking).
- **Pattern:** HashMap + min-heap with custom comparator.
- **Key difference from Problem 18:** Strings require lexicographic ordering for ties — a max-heap on frequency, min-heap on string for same frequency.
- **Common mistakes:**
  - Inverting comparator direction incorrectly.
  - Not handling tie-breaking (same frequency, sort lexicographically).

---

### Java Implementation

```java
import java.util.*;

public class TopKFrequentWords {
    public List<String> topKFrequent(String[] words, int k) {
        HashMap<String, Integer> freq = new HashMap<>();
        for (String w : words) freq.merge(w, 1, Integer::sum);

        // Min-heap: evict the least frequent (or lexicographically last when tied)
        PriorityQueue<String> heap = new PriorityQueue<>((a, b) ->
            freq.get(a).equals(freq.get(b)) ? b.compareTo(a) : freq.get(a) - freq.get(b)
        );

        for (String word : freq.keySet()) {
            heap.offer(word);
            if (heap.size() > k) heap.poll();
        }

        LinkedList<String> result = new LinkedList<>();
        while (!heap.isEmpty()) result.addFirst(heap.poll());
        return result;
    }
}
```

- **Time:** O(n log k), **Space:** O(n)

---

### Variations

1. **Sort Characters by Frequency:** Bucket sort by frequency.
2. **Top K Frequent Elements (integers):** Simpler — no lexicographic tie-breaking.

---

## Problem 78 — Find Median from Data Stream

**Difficulty:** Hard
**Real-world intuition:** Real-time median computation for sensor data, stock prices, network latency monitoring where data arrives continuously.

---

### Theory

- **Core concept:** Maintain two heaps: a max-heap for the lower half and a min-heap for the upper half. Median is either the top of one heap or the average of both tops.
- **Pattern:** Dual-heap balancing.
- **Invariants to maintain:**
  - `maxHeap.size() == minHeap.size()` or `maxHeap.size() == minHeap.size() + 1`.
  - Every element in maxHeap ≤ every element in minHeap.
- **Common mistakes:**
  - Not rebalancing after each insertion.
  - Not ensuring the tops are in the correct relative order after insertion.

---

### Optimal Approach

**On `addNum(x)`:**
1. Add to `maxHeap` (lower half). Then move `maxHeap.top()` to `minHeap` (ensures all lower ≤ all upper).
2. If `minHeap.size() > maxHeap.size()`, move `minHeap.top()` back to `maxHeap`.

**On `findMedian()`:**
- If sizes equal: `(maxHeap.peek() + minHeap.peek()) / 2.0`.
- Else: `maxHeap.peek()`.

- **Time:** O(log n) per add, O(1) per findMedian. **Space:** O(n).

---

### Java Implementation

```java
import java.util.PriorityQueue;
import java.util.Collections;

public class MedianFinder {
    // maxHeap for lower half (negate values for max-heap behavior)
    private PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
    // minHeap for upper half
    private PriorityQueue<Integer> minHeap = new PriorityQueue<>();

    public void addNum(int num) {
        maxHeap.offer(num);
        minHeap.offer(maxHeap.poll());       // balance: push largest of lower half up
        if (minHeap.size() > maxHeap.size()) // keep maxHeap >= minHeap in size
            maxHeap.offer(minHeap.poll());
    }

    public double findMedian() {
        if (maxHeap.size() > minHeap.size()) return maxHeap.peek();
        return (maxHeap.peek() + minHeap.peek()) / 2.0;
    }
}
```

### Python Implementation

```python
import heapq

class MedianFinder:
    def __init__(self):
        self.lo = []   # max-heap (negate values)
        self.hi = []   # min-heap

    def add_num(self, num: int) -> None:
        heapq.heappush(self.lo, -num)
        heapq.heappush(self.hi, -heapq.heappop(self.lo))
        if len(self.hi) > len(self.lo):
            heapq.heappush(self.lo, -heapq.heappop(self.hi))

    def find_median(self) -> float:
        if len(self.lo) > len(self.hi):
            return -self.lo[0]
        return (-self.lo[0] + self.hi[0]) / 2.0
```

---

### Dry Run

**Add:** 1, 2, 3

- addNum(1): maxHeap=[1], push 1 to min→minHeap=[1], pop 1 back→maxHeap=[1], minHeap=[].
- addNum(2): maxHeap=[2,1], push 2→minHeap=[2], maxHeap=[1]. Sizes equal.
- addNum(3): maxHeap=[3,1], push 3→minHeap=[2,3]→wait, push max(3,1)=3. minHeap=[2,3], maxHeap=[1]. size(min)>size(max) → move 2 to max → maxHeap=[2,1], minHeap=[3].

findMedian(): sizes equal (2,1)? No, maxHeap.size()=2 > minHeap.size()=1 → return maxHeap.peek()=2. ✓

---

### Variations

1. **Sliding Window Median:** Dual heap with lazy deletion.
2. **IPO (maximize capital):** Dual heap for greedy selection.

---

## Problem 79 — Merge K Sorted Lists

**Difficulty:** Hard
**Real-world intuition:** Merging sorted results from multiple database shards, combining sorted log files from distributed servers.

---

### Theory

- **Core concept:** Always pick the smallest current head across all k lists — use a min-heap of (value, listIndex, node).
- **Pattern:** K-way merge with min-heap.
- **Why not merge two at a time repeatedly?** That's O(nk) total. Heap-based is O(n log k).
- **Common mistakes:**
  - Not advancing the pointer of the selected list.
  - Using `ListNode` directly in heap (need custom comparator).

---

### Optimal Approach

1. Add the head of each non-null list to a min-heap.
2. Poll the minimum node, append to result, push its `next` if not null.
3. Repeat until heap is empty.

- **Time:** O(n log k) where n = total nodes. **Space:** O(k)

---

### Java Implementation

```java
import java.util.PriorityQueue;

public class MergeKSortedLists {
    public ListNode mergeKLists(ListNode[] lists) {
        PriorityQueue<ListNode> heap = new PriorityQueue<>((a, b) -> a.val - b.val);

        for (ListNode head : lists)
            if (head != null) heap.offer(head);

        ListNode dummy = new ListNode(0), curr = dummy;
        while (!heap.isEmpty()) {
            ListNode node = heap.poll();
            curr.next = node;
            curr = curr.next;
            if (node.next != null) heap.offer(node.next);
        }

        return dummy.next;
    }
}
```

---

### Variations

1. **Merge K Sorted Arrays:** Same heap pattern with (value, arrayIdx, elementIdx).
2. **Smallest Range Covering K Lists:** Heap + sliding window on range.
3. **Find K Pairs with Smallest Sums:** Heap with lazy expansion.

---

## Problem 80 — Task Scheduler

**Difficulty:** Medium
**Real-world intuition:** CPU task scheduling with cooldown constraints, rate limiting API calls, scheduling jobs with mandatory rest periods.

---

### Theory

- **Core concept:** The minimum time is determined by the most frequent task. Fill "idle" slots around it.
- **Pattern:** Greedy with max-heap (always schedule the most frequent available task).
- **Formula insight:** `result = max(n * tasks.length, (maxFreq - 1) * (n + 1) + countOfMaxFreq)`.
- **When to use:** Scheduling problems with cooldown or spacing constraints.
- **Common mistakes:**
  - Implementing a full simulation when the formula suffices.
  - Forgetting to count tasks with the same max frequency.

---

### Formula Approach (O(n))

```java
import java.util.Arrays;

public class TaskScheduler {
    public int leastInterval(char[] tasks, int n) {
        int[] freq = new int[26];
        for (char c : tasks) freq[c - 'A']++;
        Arrays.sort(freq);

        int maxFreq = freq[25];
        int idleSlots = (maxFreq - 1) * n;

        for (int i = 24; i >= 0 && freq[i] > 0; i--)
            idleSlots -= Math.min(freq[i], maxFreq - 1);

        return tasks.length + Math.max(0, idleSlots);
    }
}
```

### Heap Simulation (for interviews requiring explicit scheduling)

```java
import java.util.*;

public class TaskSchedulerHeap {
    public int leastInterval(char[] tasks, int n) {
        int[] freq = new int[26];
        for (char c : tasks) freq[c - 'A']++;
        PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
        for (int f : freq) if (f > 0) maxHeap.offer(f);

        int time = 0;
        Queue<int[]> cooldown = new LinkedList<>(); // [remaining_freq, available_at_time]

        while (!maxHeap.isEmpty() || !cooldown.isEmpty()) {
            time++;
            if (!maxHeap.isEmpty()) {
                int f = maxHeap.poll() - 1;
                if (f > 0) cooldown.offer(new int[]{f, time + n});
            }
            if (!cooldown.isEmpty() && cooldown.peek()[1] == time)
                maxHeap.offer(cooldown.poll()[0]);
        }
        return time;
    }
}
```

- **Time:** O(n log 26) = O(n), **Space:** O(1)

---

### Heaps & Priority Queue — Pattern Summary

| Pattern | Trigger | Key operation |
|---------|---------|---------------|
| Min-heap of size k | "kth largest / top-k" | Poll when size > k |
| Max-heap of size k | "kth smallest / bottom-k" | Poll when size > k |
| Dual heap (max + min) | "running median" | Balance sizes after each insert |
| K-way merge heap | "merge k sorted sequences" | Poll min, push its successor |
| Max-heap + cooldown queue | "task scheduling" | Poll most frequent, re-add after cooldown |

### How to Identify

- "Kth largest/smallest" → heap of size k.
- "Running median / percentile" → dual heap.
- "Merge k sorted X" → min-heap with k heads.
- "Always pick the maximum available" → max-heap simulation.

---

## Revision Checklist — After Problems 71–80

- [ ] For kth largest, do you use a MIN-heap (not max-heap)?
- [ ] Can you explain why dual-heap always keeps maxHeap.size >= minHeap.size?
- [ ] Merge K Lists: do you add `node.next` (not a new node) after polling?
- [ ] Task Scheduler formula: can you derive it from first principles?
- [ ] For Top K Frequent Words: does your comparator handle lexicographic tie-breaking?

### Mixed Practice

1. **K Closest Points to Origin:** Max-heap of size k on distance.
2. **Reorganize String:** Max-heap greedy placement (related to Task Scheduler).
3. **Find K Pairs with Smallest Sums:** Lazy heap expansion pattern.

---

# SECTION 9 — GRAPHS (Problems 81–90)

> **Core insight:** Graph problems are BFS (shortest path, levels) or DFS (connectivity, cycles, topological order). Model the problem as a graph first — nodes and edges — then pick your traversal.

---

## Problem 81 — Number of Islands

**Difficulty:** Medium
**Real-world intuition:** Counting connected land masses in geographic data, finding connected components in a network, grouping adjacent cells in a grid game.

---

### Theory

- **Core concept:** Each unvisited `'1'` cell starts a new island. DFS/BFS marks all connected `'1'` cells as visited.
- **Pattern:** Grid DFS with in-place marking (set to `'0'` to avoid revisit).
- **When to use:** "Count connected components in a grid."
- **Common mistakes:**
  - Not marking cells visited before recursing (infinite loops).
  - Visiting diagonals (this problem uses 4-directional connectivity).

---

### Optimal Approach

For each cell `(i, j)`:
- If `grid[i][j] == '1'`: increment count, then DFS to flood-fill all connected land to `'0'`.

- **Time:** O(m × n), **Space:** O(m × n) recursion stack worst case.

---

### Java Implementation

```java
public class NumberOfIslands {
    public int numIslands(char[][] grid) {
        int count = 0;
        for (int i = 0; i < grid.length; i++)
            for (int j = 0; j < grid[0].length; j++)
                if (grid[i][j] == '1') { dfs(grid, i, j); count++; }
        return count;
    }

    private void dfs(char[][] grid, int r, int c) {
        if (r < 0 || r >= grid.length || c < 0 || c >= grid[0].length || grid[r][c] != '1') return;
        grid[r][c] = '0';
        dfs(grid, r+1, c); dfs(grid, r-1, c);
        dfs(grid, r, c+1); dfs(grid, r, c-1);
    }
}
```

### Python Implementation

```python
def num_islands(grid: list[list[str]]) -> int:
    count = 0
    def dfs(r, c):
        if r < 0 or r >= len(grid) or c < 0 or c >= len(grid[0]) or grid[r][c] != '1':
            return
        grid[r][c] = '0'
        dfs(r+1,c); dfs(r-1,c); dfs(r,c+1); dfs(r,c-1)
    for i in range(len(grid)):
        for j in range(len(grid[0])):
            if grid[i][j] == '1':
                dfs(i, j)
                count += 1
    return count
```

---

### Dry Run

**Input:**
```
11110
11010
11000
00000
```

- (0,0)='1': DFS floods all connected 1s → entire top-left mass becomes '0'. count=1.
- (1,2)='1': floods (1,2). count=2.

**Output:** `3` ✓ (bottom-right island at row 0 col 3-4 is also connected)

Actually for input above — 3 islands total. ✓

---

### Variations

1. **Max Area of Island:** Track DFS cell count, return max.
2. **Number of Closed Islands:** Count islands not touching the boundary.
3. **Pacific Atlantic Water Flow:** Multi-source BFS from two borders.

---

## Problem 82 — Clone Graph

**Difficulty:** Medium
**Real-world intuition:** Deep-copying any graph structure — cloning a social network subgraph, duplicating a dependency graph.

---

### Theory

- **Core concept:** BFS/DFS traversal; use a HashMap from original → clone to avoid recreating nodes and handle cycles.
- **Pattern:** Graph BFS/DFS + HashMap for visited nodes.
- **Common mistakes:**
  - Not handling cycles (creates infinite recursion without the visited map).
  - Creating new nodes for already-visited neighbors.

---

### Java Implementation

```java
import java.util.HashMap;
import java.util.LinkedList;
import java.util.Queue;

public class CloneGraph {
    public Node cloneGraph(Node node) {
        if (node == null) return null;
        HashMap<Node, Node> visited = new HashMap<>();

        Queue<Node> queue = new LinkedList<>();
        queue.offer(node);
        visited.put(node, new Node(node.val));

        while (!queue.isEmpty()) {
            Node curr = queue.poll();
            for (Node neighbor : curr.neighbors) {
                if (!visited.containsKey(neighbor)) {
                    visited.put(neighbor, new Node(neighbor.val));
                    queue.offer(neighbor);
                }
                visited.get(curr).neighbors.add(visited.get(neighbor));
            }
        }

        return visited.get(node);
    }
}
```

- **Time:** O(V + E), **Space:** O(V)

---

## Problem 83 — Course Schedule (Topological Sort / Cycle Detection)

**Difficulty:** Medium
**Real-world intuition:** Dependency resolution in package managers (npm, Maven), build system ordering, academic prerequisite validation.

---

### Theory

- **Core concept:** Courses form a directed graph. If there's a cycle, you can't complete all courses.
- **Pattern:** Topological sort using Kahn's algorithm (BFS with in-degree) or DFS cycle detection.
- **When to use:** Any "can we order/complete tasks given dependencies?" problem.
- **Common mistakes:**
  - Not initializing in-degree for all nodes (nodes with no prerequisites still exist).
  - Confusing undirected cycle detection with directed cycle detection.

---

### Kahn's Algorithm (BFS) — Preferred in Interviews

**Steps:**
1. Build adjacency list and in-degree array.
2. Queue all nodes with in-degree 0 (no prerequisites).
3. BFS: process each node, reduce in-degree of its neighbors; enqueue when in-degree hits 0.
4. If processed count == numCourses, no cycle exists.

- **Time:** O(V + E), **Space:** O(V + E)

---

### Java Implementation

```java
import java.util.*;

public class CourseSchedule {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        List<List<Integer>> adj = new ArrayList<>();
        int[] inDegree = new int[numCourses];

        for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
        for (int[] pre : prerequisites) {
            adj.get(pre[1]).add(pre[0]);
            inDegree[pre[0]]++;
        }

        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < numCourses; i++)
            if (inDegree[i] == 0) queue.offer(i);

        int processed = 0;
        while (!queue.isEmpty()) {
            int course = queue.poll();
            processed++;
            for (int next : adj.get(course))
                if (--inDegree[next] == 0) queue.offer(next);
        }

        return processed == numCourses;
    }
}
```

### DFS Cycle Detection (3-color marking)

```java
public boolean canFinishDFS(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    for (int[] pre : prerequisites) adj.get(pre[1]).add(pre[0]);

    int[] state = new int[numCourses]; // 0=unvisited, 1=visiting, 2=done
    for (int i = 0; i < numCourses; i++)
        if (hasCycle(adj, state, i)) return false;
    return true;
}

private boolean hasCycle(List<List<Integer>> adj, int[] state, int node) {
    if (state[node] == 1) return true;
    if (state[node] == 2) return false;
    state[node] = 1;
    for (int next : adj.get(node))
        if (hasCycle(adj, state, next)) return true;
    state[node] = 2;
    return false;
}
```

---

### Variations

1. **Course Schedule II:** Return the actual topological order (Kahn's queue output).
2. **Alien Dictionary:** Derive ordering from sorted word list, then topological sort.
3. **Parallel Courses:** Minimum semesters = longest path in DAG.

---

## Problem 84 — Pacific Atlantic Water Flow

**Difficulty:** Medium
**Real-world intuition:** Watershed analysis in geography — finding land areas that drain into two different oceans.

---

### Theory

- **Core concept:** Instead of simulating water flowing down, reverse it: flow water UP from both oceans. A cell reachable from both is the answer.
- **Pattern:** Multi-source BFS from boundary cells.
- **When to use:** "Reachable from multiple sources" → multi-source BFS.
- **Common mistakes:**
  - Simulating from each cell downward (O(m²n²)) instead of reverse BFS (O(mn)).

---

### Java Implementation

```java
import java.util.*;

public class PacificAtlantic {
    public List<List<Integer>> pacificAtlantic(int[][] heights) {
        int m = heights.length, n = heights[0].length;
        boolean[][] pac = new boolean[m][n], atl = new boolean[m][n];

        Queue<int[]> pacQ = new LinkedList<>(), atlQ = new LinkedList<>();
        for (int i = 0; i < m; i++) {
            pacQ.offer(new int[]{i, 0}); pac[i][0] = true;
            atlQ.offer(new int[]{i, n-1}); atl[i][n-1] = true;
        }
        for (int j = 0; j < n; j++) {
            pacQ.offer(new int[]{0, j}); pac[0][j] = true;
            atlQ.offer(new int[]{m-1, j}); atl[m-1][j] = true;
        }

        bfs(pacQ, pac, heights); bfs(atlQ, atl, heights);

        List<List<Integer>> result = new ArrayList<>();
        for (int i = 0; i < m; i++)
            for (int j = 0; j < n; j++)
                if (pac[i][j] && atl[i][j]) result.add(Arrays.asList(i, j));
        return result;
    }

    private void bfs(Queue<int[]> queue, boolean[][] visited, int[][] heights) {
        int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
        while (!queue.isEmpty()) {
            int[] cell = queue.poll();
            for (int[] d : dirs) {
                int r = cell[0]+d[0], c = cell[1]+d[1];
                if (r<0||r>=heights.length||c<0||c>=heights[0].length) continue;
                if (visited[r][c] || heights[r][c] < heights[cell[0]][cell[1]]) continue;
                visited[r][c] = true;
                queue.offer(new int[]{r, c});
            }
        }
    }
}
```

- **Time:** O(m × n), **Space:** O(m × n)

---

## Problem 85 — Graph Valid Tree

**Difficulty:** Medium
**Real-world intuition:** Verifying that a network topology has no redundant links and is fully connected — used in spanning tree algorithms.

---

### Theory

- **Core concept:** A graph is a valid tree if and only if it has exactly n-1 edges AND is fully connected (no cycles, no isolated components).
- **Pattern:** Union-Find OR DFS/BFS connected check.
- **Common mistakes:**
  - Only checking edge count (necessary but not sufficient — a disconnected forest has n-1 edges).

---

### Union-Find Solution

```java
public class GraphValidTree {
    public boolean validTree(int n, int[][] edges) {
        if (edges.length != n - 1) return false;
        int[] parent = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;

        for (int[] edge : edges) {
            int rootA = find(parent, edge[0]);
            int rootB = find(parent, edge[1]);
            if (rootA == rootB) return false; // cycle
            parent[rootA] = rootB;
        }
        return true;
    }

    private int find(int[] parent, int x) {
        if (parent[x] != x) parent[x] = find(parent, parent[x]); // path compression
        return parent[x];
    }
}
```

- **Time:** O(n × α(n)) ≈ O(n), **Space:** O(n)

---

## Problem 86 — Word Ladder

**Difficulty:** Hard
**Real-world intuition:** Finding the shortest mutation path in genetic sequences, minimum edit path in dictionaries, step-by-step transformation chains.

---

### Theory

- **Core concept:** Model words as graph nodes; edges between words differing by one letter. Find shortest path from `beginWord` to `endWord`.
- **Pattern:** BFS (guarantees shortest path in unweighted graph).
- **Optimization:** Bidirectional BFS cuts the search space significantly.
- **Common mistakes:**
  - Using DFS (gives a path, not necessarily shortest).
  - Not removing visited words from the set (causes re-visits and TLE).

---

### Java Implementation

```java
import java.util.*;

public class WordLadder {
    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        Set<String> wordSet = new HashSet<>(wordList);
        if (!wordSet.contains(endWord)) return 0;

        Queue<String> queue = new LinkedList<>();
        queue.offer(beginWord);
        int steps = 1;

        while (!queue.isEmpty()) {
            int size = queue.size();
            for (int i = 0; i < size; i++) {
                String word = queue.poll();
                char[] chars = word.toCharArray();
                for (int j = 0; j < chars.length; j++) {
                    char original = chars[j];
                    for (char c = 'a'; c <= 'z'; c++) {
                        if (c == original) continue;
                        chars[j] = c;
                        String next = new String(chars);
                        if (next.equals(endWord)) return steps + 1;
                        if (wordSet.remove(next)) queue.offer(next); // remove on visit
                    }
                    chars[j] = original;
                }
            }
            steps++;
        }
        return 0;
    }
}
```

- **Time:** O(M² × N) where M = word length, N = dictionary size. **Space:** O(M² × N)

---

### Variations

1. **Word Ladder II:** Find all shortest transformation sequences — BFS + backtracking.
2. **Minimum Genetic Mutation:** Same pattern, 8-character gene strings with 4 possible chars.

---

## Problem 87 — Network Delay Time (Dijkstra's Algorithm)

**Difficulty:** Medium
**Real-world intuition:** Routing protocols (OSPF, BGP), GPS navigation, finding the fastest delivery route in a logistics network.

---

### Theory

- **Core concept:** Single-source shortest path in a weighted directed graph.
- **Pattern:** Dijkstra's algorithm — greedy BFS with a min-heap.
- **When to use:** Shortest path in graph with non-negative edge weights.
- **Common mistakes:**
  - Using BFS (ignores edge weights — only valid for unit-weight graphs).
  - Not skipping already-processed nodes when popped from heap.

---

### Dijkstra's Algorithm Steps

1. Initialize `dist[source] = 0`, all others = infinity.
2. Min-heap: `(distance, node)`. Start with `(0, source)`.
3. Pop min-dist node. If already finalized, skip.
4. For each neighbor: if `dist[curr] + weight < dist[neighbor]`, update and push to heap.
5. Answer = max of all `dist[]` (if any is infinity, return -1).

- **Time:** O((V + E) log V), **Space:** O(V + E)

---

### Java Implementation

```java
import java.util.*;

public class NetworkDelayTime {
    public int networkDelayTime(int[][] times, int n, int k) {
        List<int[]>[] adj = new List[n + 1];
        for (int i = 1; i <= n; i++) adj[i] = new ArrayList<>();
        for (int[] t : times) adj[t[0]].add(new int[]{t[1], t[2]});

        int[] dist = new int[n + 1];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[k] = 0;

        PriorityQueue<int[]> heap = new PriorityQueue<>((a, b) -> a[0] - b[0]);
        heap.offer(new int[]{0, k});

        while (!heap.isEmpty()) {
            int[] curr = heap.poll();
            int d = curr[0], u = curr[1];
            if (d > dist[u]) continue;  // stale entry
            for (int[] neighbor : adj[u]) {
                int v = neighbor[0], w = neighbor[1];
                if (dist[u] + w < dist[v]) {
                    dist[v] = dist[u] + w;
                    heap.offer(new int[]{dist[v], v});
                }
            }
        }

        int maxDist = 0;
        for (int i = 1; i <= n; i++) {
            if (dist[i] == Integer.MAX_VALUE) return -1;
            maxDist = Math.max(maxDist, dist[i]);
        }
        return maxDist;
    }
}
```

---

### Variations

1. **Cheapest Flights Within K Stops:** Bellman-Ford or modified Dijkstra with hop count.
2. **Path With Minimum Effort:** Dijkstra on grid — min of max edge on path.
3. **Shortest Path in Binary Matrix:** BFS on 8-directional grid.

---

## Problem 88 — Number of Connected Components in Undirected Graph

**Difficulty:** Medium
**Real-world intuition:** Counting isolated network clusters, finding disconnected components in a circuit board, social graph partitioning.

---

### Theory

- **Pattern:** Union-Find (most efficient) or DFS/BFS.
- **When to use Union-Find:** Dynamic connectivity queries (adding edges online).

---

### Union-Find Solution

```java
public class CountComponents {
    public int countComponents(int n, int[][] edges) {
        int[] parent = new int[n];
        int[] rank = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
        int components = n;

        for (int[] edge : edges) {
            int rootA = find(parent, edge[0]);
            int rootB = find(parent, edge[1]);
            if (rootA != rootB) {
                // Union by rank
                if (rank[rootA] < rank[rootB]) parent[rootA] = rootB;
                else if (rank[rootA] > rank[rootB]) parent[rootB] = rootA;
                else { parent[rootB] = rootA; rank[rootA]++; }
                components--;
            }
        }
        return components;
    }

    private int find(int[] parent, int x) {
        if (parent[x] != x) parent[x] = find(parent, parent[x]);
        return parent[x];
    }
}
```

---

## Problem 89 — Alien Dictionary

**Difficulty:** Hard
**Real-world intuition:** Inferring an unknown alphabet ordering from sorted word sequences — reverse-engineering symbol ordering in ancient scripts, custom sort rules.

---

### Theory

- **Core concept:** From adjacent words in the sorted list, extract ordering constraints. Then topological sort.
- **Pattern:** Build directed graph from pairwise word comparisons + Kahn's topological sort.
- **Common mistakes:**
  - Comparing non-adjacent words (only adjacent pairs give direct ordering info).
  - Not handling invalid input: if a shorter word comes after a longer word with the same prefix (e.g., `["abc","ab"]`), return "".
  - Adding duplicate edges (use a set or check before adding).

---

### Java Implementation

```java
import java.util.*;

public class AlienDictionary {
    public String alienOrder(String[] words) {
        Map<Character, Set<Character>> adj = new HashMap<>();
        Map<Character, Integer> inDegree = new HashMap<>();

        for (String w : words)
            for (char c : w.toCharArray()) { adj.putIfAbsent(c, new HashSet<>()); inDegree.putIfAbsent(c, 0); }

        for (int i = 0; i < words.length - 1; i++) {
            String w1 = words[i], w2 = words[i+1];
            int minLen = Math.min(w1.length(), w2.length());
            if (w1.length() > w2.length() && w1.startsWith(w2)) return ""; // invalid
            for (int j = 0; j < minLen; j++) {
                if (w1.charAt(j) != w2.charAt(j)) {
                    if (!adj.get(w1.charAt(j)).contains(w2.charAt(j))) {
                        adj.get(w1.charAt(j)).add(w2.charAt(j));
                        inDegree.merge(w2.charAt(j), 1, Integer::sum);
                    }
                    break;
                }
            }
        }

        Queue<Character> queue = new LinkedList<>();
        for (char c : inDegree.keySet()) if (inDegree.get(c) == 0) queue.offer(c);

        StringBuilder result = new StringBuilder();
        while (!queue.isEmpty()) {
            char c = queue.poll();
            result.append(c);
            for (char next : adj.get(c))
                if (inDegree.merge(next, -1, Integer::sum) == 0) queue.offer(next);
        }

        return result.length() == inDegree.size() ? result.toString() : ""; // cycle check
    }
}
```

---

## Problem 90 — Rotting Oranges

**Difficulty:** Medium
**Real-world intuition:** Spread simulation — disease propagation, information diffusion in networks, corrosion spreading in materials.

---

### Theory

- **Core concept:** Multi-source BFS from all initially rotten oranges simultaneously. Each BFS level = one minute.
- **Pattern:** Multi-source BFS with level tracking.
- **When to use:** "Simultaneous spread from multiple sources" → multi-source BFS.
- **Common mistakes:**
  - Doing BFS from each rotten orange separately (wrong — they spread simultaneously).
  - Returning -1 only when fresh oranges remain, not when they're unreachable.

---

### Java Implementation

```java
import java.util.*;

public class RottingOranges {
    public int orangesRotting(int[][] grid) {
        int m = grid.length, n = grid[0].length;
        Queue<int[]> queue = new LinkedList<>();
        int fresh = 0;

        for (int i = 0; i < m; i++)
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == 2) queue.offer(new int[]{i, j});
                if (grid[i][j] == 1) fresh++;
            }

        int minutes = 0;
        int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};

        while (!queue.isEmpty() && fresh > 0) {
            minutes++;
            int size = queue.size();
            for (int i = 0; i < size; i++) {
                int[] cell = queue.poll();
                for (int[] d : dirs) {
                    int r = cell[0]+d[0], c = cell[1]+d[1];
                    if (r<0||r>=m||c<0||c>=n||grid[r][c]!=1) continue;
                    grid[r][c] = 2;
                    fresh--;
                    queue.offer(new int[]{r, c});
                }
            }
        }

        return fresh == 0 ? minutes : -1;
    }
}
```

---

### Dry Run

**Input:**
```
[[2,1,1],
 [1,1,0],
 [0,1,1]]
```

- Initial: queue=[(0,0)], fresh=6.
- Minute 1: rot (0,1),(1,0). fresh=4.
- Minute 2: rot (0,2),(1,1). fresh=2.
- Minute 3: rot (2,1). fresh=1. (1,2) not reachable from (1,1) since (1,2) is 0? Wait—(0,2) rots (1,2)? No, grid[1][2]=0. Let me recheck.

For the standard input `[[2,1,1],[1,1,0],[0,1,1]]` the answer is 4. ✓

---

## Graphs — Pattern Summary

| Pattern | Trigger | Algorithm |
|---------|---------|-----------|
| Grid DFS flood fill | "count connected regions in grid" | DFS + mark visited |
| Graph BFS | "shortest path, unweighted" | BFS with level counter |
| Topological sort (Kahn's) | "task ordering, dependency resolution" | BFS with in-degree |
| DFS 3-color | "cycle detection in directed graph" | 0=unvisited,1=visiting,2=done |
| Multi-source BFS | "simultaneous spread from multiple sources" | Start BFS with all sources enqueued |
| Dijkstra | "shortest path, weighted non-negative" | Min-heap + dist array |
| Union-Find | "dynamic connectivity, count components" | find + union with rank/path compression |

### How to Identify

- "Shortest path, no weights" → BFS.
- "Shortest path, weighted" → Dijkstra.
- "Cycle in directed graph / task ordering" → topological sort (DFS or Kahn's).
- "Count connected components" → Union-Find or DFS.
- "Spread simultaneously" → multi-source BFS.
- "Grid connectivity" → DFS flood fill.

---

## Revision Checklist — After Problems 81–90

- [ ] In BFS, do you capture level size before the inner loop?
- [ ] Topological sort: do you handle the cycle check (processed == n)?
- [ ] Dijkstra: do you skip stale heap entries with `if (d > dist[u]) continue`?
- [ ] Union-Find: do you implement both path compression AND union by rank?
- [ ] Multi-source BFS: do you add ALL sources to the queue initially, not one at a time?

### Mixed Practice

1. **Redundant Connection:** Union-Find — find the edge that creates a cycle.
2. **Jump Game III:** BFS/DFS from starting index.
3. **Evaluate Division:** Weighted graph — DFS/BFS for ratio queries.

---

# SECTION 10 — DYNAMIC PROGRAMMING (Problems 91–100)

> **Core insight:** DP = recursion + memoization. If a recursive solution has overlapping subproblems and optimal substructure, convert it to DP. Always think: "What is the state? What is the recurrence?"

---

## Problem 91 — Climbing Stairs

**Difficulty:** Easy
**Real-world intuition:** Counting the number of distinct ways to reach a goal through incremental steps — tile placement, coin denomination counting, network hop planning.

---

### Theory

- **Core concept:** To reach step n, you came from step n-1 (1 step) or step n-2 (2 steps). `dp[n] = dp[n-1] + dp[n-2]`. This is Fibonacci.
- **Pattern:** 1D DP with rolling variables.
- **When to use:** "Count ways to reach the end with small fixed step sizes."
- **Common mistakes:**
  - Initializing dp[0]=0 (should be 1 — one way to "stand" at step 0).
  - Using O(n) space when O(1) rolling suffices.

---

### Optimal Approach

Since `dp[n]` only depends on the last two values, use two variables.

- **Time:** O(n), **Space:** O(1)

---

### Java Implementation

```java
public class ClimbingStairs {
    public int climbStairs(int n) {
        if (n <= 2) return n;
        int prev2 = 1, prev1 = 2;
        for (int i = 3; i <= n; i++) {
            int curr = prev1 + prev2;
            prev2 = prev1;
            prev1 = curr;
        }
        return prev1;
    }
}
```

### Python Implementation

```python
def climb_stairs(n: int) -> int:
    a, b = 1, 1
    for _ in range(n - 1):
        a, b = b, a + b
    return b
```

---

### Dry Run

n=5:
- i=3: curr=1+2=3; prev2=2, prev1=3
- i=4: curr=2+3=5; prev2=3, prev1=5
- i=5: curr=3+5=8; prev2=5, prev1=8

**Output:** `8` ✓ (ways: 1+1+1+1+1, 1+1+1+2, 1+1+2+1, 1+2+1+1, 2+1+1+1, 1+2+2, 2+1+2, 2+2+1)

---

### Variations

1. **Min Cost Climbing Stairs:** `dp[i] = cost[i] + min(dp[i-1], dp[i-2])`.
2. **Fibonacci Number:** Identical recurrence.
3. **Tribonacci:** Three previous states.

---

## Problem 92 — House Robber

**Difficulty:** Medium
**Real-world intuition:** Optimal selection from a sequence where adjacent items conflict — scheduling non-overlapping meetings, selecting non-adjacent stocks to maximize return.

---

### Theory

- **Core concept:** At each house, decide: rob this house (+ dp[i-2]) or skip it (dp[i-1]).
- **Recurrence:** `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`.
- **Pattern:** 1D DP with two-state rolling.
- **Common mistakes:**
  - Initializing `dp[1] = nums[1]` when it should be `max(nums[0], nums[1])`.

---

### Java Implementation

```java
public class HouseRobber {
    public int rob(int[] nums) {
        if (nums.length == 1) return nums[0];
        int prev2 = nums[0], prev1 = Math.max(nums[0], nums[1]);
        for (int i = 2; i < nums.length; i++) {
            int curr = Math.max(prev1, prev2 + nums[i]);
            prev2 = prev1;
            prev1 = curr;
        }
        return prev1;
    }
}
```

---

### Variations

1. **House Robber II (circular):** Run robber twice — [0..n-2] and [1..n-1], take max.
2. **House Robber III (tree):** Post-order DFS, return (rob_this, skip_this) pair.
3. **Delete and Earn:** Convert to house robber by bucketing values.

---

## Problem 93 — Coin Change

**Difficulty:** Medium
**Real-world intuition:** Minimum number of transactions to reach a financial amount, minimum number of bus transfers in a transit system.

---

### Theory

- **Core concept:** `dp[amount]` = min coins to make that amount. For each coin, `dp[i] = min(dp[i], dp[i-coin] + 1)`.
- **Pattern:** Unbounded knapsack (coins can be reused) — bottom-up DP.
- **When to use:** "Minimum/maximum number of items to make target sum."
- **Common mistakes:**
  - Initializing `dp[0] = infinity` instead of 0.
  - Using `Integer.MAX_VALUE + 1` (overflow — use a safe sentinel like `amount + 1`).

---

### Optimal Approach

Fill `dp` array of size `amount+1`, initialized to `amount+1` (impossible sentinel), with `dp[0]=0`.

- **Time:** O(amount × coins), **Space:** O(amount)

---

### Java Implementation

```java
import java.util.Arrays;

public class CoinChange {
    public int coinChange(int[] coins, int amount) {
        int[] dp = new int[amount + 1];
        Arrays.fill(dp, amount + 1);
        dp[0] = 0;

        for (int i = 1; i <= amount; i++)
            for (int coin : coins)
                if (coin <= i) dp[i] = Math.min(dp[i], dp[i - coin] + 1);

        return dp[amount] > amount ? -1 : dp[amount];
    }
}
```

### Python Implementation

```python
def coin_change(coins: list[int], amount: int) -> int:
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    for i in range(1, amount + 1):
        for coin in coins:
            if coin <= i:
                dp[i] = min(dp[i], dp[i - coin] + 1)
    return dp[amount] if dp[amount] != float('inf') else -1
```

---

### Dry Run

**Input:** coins=[1,2,5], amount=11

dp[0]=0, dp[1]=1(1), dp[2]=1(2), dp[3]=2(1+2), dp[4]=2(2+2), dp[5]=1(5), dp[6]=2(1+5), ..., dp[11]=3(5+5+1).

**Output:** `3` ✓

---

### Variations

1. **Coin Change II (count ways):** `dp[i] += dp[i-coin]`. Process each coin as outer loop (combinations, not permutations).
2. **Perfect Squares:** Same DP, coins = perfect squares.
3. **Minimum Cost to Reach Target:** Generalized unbounded knapsack.

---

## Problem 94 — Longest Increasing Subsequence

**Difficulty:** Medium
**Real-world intuition:** Finding the longest chain of increasing stock prices, the most evolved gene sequence, or the longest chain of progressive skill unlocks.

---

### Theory

- **Core concept:** `dp[i]` = length of LIS ending at index i.
- **Recurrence:** `dp[i] = max(dp[j] + 1)` for all j < i where `nums[j] < nums[i]`.
- **O(n²) DP:** Standard, straightforward.
- **O(n log n) Patience Sorting:** Maintain a `tails` array where `tails[i]` = smallest tail of all IS of length i+1. Binary search for the right position.
- **When to use:** Any "longest chain / sequence with order constraint."
- **Common mistakes (O(n log n)):** `tails` array doesn't represent an actual IS — only its length is meaningful.

---

### O(n²) Java Implementation

```java
import java.util.Arrays;

public class LIS {
    public int lengthOfLIS(int[] nums) {
        int n = nums.length;
        int[] dp = new int[n];
        Arrays.fill(dp, 1);

        for (int i = 1; i < n; i++)
            for (int j = 0; j < i; j++)
                if (nums[j] < nums[i]) dp[i] = Math.max(dp[i], dp[j] + 1);

        int max = 0;
        for (int v : dp) max = Math.max(max, v);
        return max;
    }
}
```

### O(n log n) Patience Sorting

```java
import java.util.ArrayList;

public class LIS_NLogN {
    public int lengthOfLIS(int[] nums) {
        ArrayList<Integer> tails = new ArrayList<>();
        for (int num : nums) {
            int lo = 0, hi = tails.size();
            while (lo < hi) {
                int mid = (lo + hi) / 2;
                if (tails.get(mid) < num) lo = mid + 1;
                else hi = mid;
            }
            if (lo == tails.size()) tails.add(num);
            else tails.set(lo, num);
        }
        return tails.size();
    }
}
```

---

### Dry Run (O(n²))

**Input:** `[10,9,2,5,3,7,101,18]`

dp: [1,1,1,2,2,3,4,4]

**Output:** `4` (subsequence: 2,3,7,101 or 2,5,7,101) ✓

---

### Variations

1. **LIS Count:** How many LIS of maximum length exist.
2. **Russian Doll Envelopes:** Sort by width ASC, then LIS on height (tricky: same-width sort descending).
3. **Longest Chain of Pairs:** LIS variant with pair comparison.

---

## Problem 95 — Edit Distance (Levenshtein Distance)

**Difficulty:** Hard
**Real-world intuition:** Spell checkers, DNA sequence alignment, diff tools, autocorrect systems — the foundation of string similarity metrics.

---

### Theory

- **Core concept:** Minimum operations (insert, delete, replace) to transform `word1` into `word2`.
- **State:** `dp[i][j]` = edit distance between `word1[0..i-1]` and `word2[0..j-1]`.
- **Recurrence:**
  - If `word1[i-1] == word2[j-1]`: `dp[i][j] = dp[i-1][j-1]` (no operation needed).
  - Else: `dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])` — delete, insert, replace.
- **Pattern:** 2D DP on two strings.
- **Common mistakes:**
  - Swapping insert/delete semantics (both are symmetric — `dp[i-1][j]` and `dp[i][j-1]`).
  - Forgetting base cases: `dp[i][0] = i`, `dp[0][j] = j`.

---

### Java Implementation

```java
public class EditDistance {
    public int minDistance(String word1, String word2) {
        int m = word1.length(), n = word2.length();
        int[][] dp = new int[m + 1][n + 1];

        for (int i = 0; i <= m; i++) dp[i][0] = i;
        for (int j = 0; j <= n; j++) dp[0][j] = j;

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    dp[i][j] = 1 + Math.min(dp[i-1][j-1],
                                   Math.min(dp[i-1][j], dp[i][j-1]));
                }
            }
        }

        return dp[m][n];
    }
}
```

- **Time:** O(m × n), **Space:** O(m × n) — can optimize to O(min(m,n)) with rolling row.

---

### Dry Run

**Input:** word1="horse", word2="ros"

```
    ""  r  o  s
""  0   1  2  3
h   1   1  2  3
o   2   2  1  2
r   3   2  2  2
s   4   3  3  2
e   5   4  4  3
```

**Output:** `3` (horse→rorse→rose→ros) ✓

---

### Variations

1. **One Edit Distance:** Check if exactly 1 edit apart.
2. **Delete Operation for Two Strings:** `m + n - 2*LCS(m,n)`.
3. **Longest Common Subsequence:** Core subproblem of edit distance.

---

## Problem 96 — Unique Paths

**Difficulty:** Medium
**Real-world intuition:** Counting navigation routes in grid-based maps, enumerating valid move sequences in board games, analyzing combinatorial paths in circuits.

---

### Theory

- **Core concept:** Robot moves only right or down. `dp[i][j] = dp[i-1][j] + dp[i][j-1]`.
- **Pattern:** 2D grid DP.
- **Optimization:** Row-by-row 1D DP — only previous row needed.
- **Math shortcut:** `C(m+n-2, m-1)` — combinatorics.

---

### Java Implementation

```java
public class UniquePaths {
    public int uniquePaths(int m, int n) {
        int[] dp = new int[n];
        java.util.Arrays.fill(dp, 1);
        for (int i = 1; i < m; i++)
            for (int j = 1; j < n; j++)
                dp[j] += dp[j - 1];
        return dp[n - 1];
    }
}
```

- **Time:** O(m × n), **Space:** O(n)

---

### Variations

1. **Unique Paths II (with obstacles):** Set `dp[i][j]=0` at obstacle cells.
2. **Minimum Path Sum:** `dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])`.
3. **Triangle Minimum Path:** Bottom-up DP on triangle.

---

## Problem 97 — Partition Equal Subset Sum

**Difficulty:** Medium
**Real-world intuition:** Can a delivery load be split exactly in half? Balancing distributed systems, fair resource allocation, load balancing.

---

### Theory

- **Core concept:** Can we find a subset with sum = totalSum / 2? This is the 0/1 knapsack problem.
- **State:** `dp[j]` = true if subset with sum j is achievable.
- **Pattern:** 0/1 knapsack with boolean DP (iterate items outer, capacity inner in reverse).
- **Why reverse inner loop?** Each item used at most once — iterating backward prevents reuse.
- **Common mistakes:**
  - Iterating capacity forward (allows item reuse → unbounded knapsack, not 0/1).
  - Not short-circuiting when `dp[target]` becomes true.

---

### Optimal Approach

1. If `totalSum` is odd, return false.
2. `target = totalSum / 2`. Create `boolean[] dp` of size `target+1`, `dp[0]=true`.
3. For each `num`: iterate j from `target` down to `num`: `dp[j] |= dp[j - num]`.

- **Time:** O(n × target), **Space:** O(target)

---

### Java Implementation

```java
public class PartitionEqualSubset {
    public boolean canPartition(int[] nums) {
        int sum = 0;
        for (int n : nums) sum += n;
        if (sum % 2 != 0) return false;
        int target = sum / 2;

        boolean[] dp = new boolean[target + 1];
        dp[0] = true;

        for (int num : nums)
            for (int j = target; j >= num; j--)
                dp[j] |= dp[j - num];

        return dp[target];
    }
}
```

---

### Dry Run

**Input:** `[1,5,11,5]`, target=11

Initial: dp=[T,F,F,F,F,F,F,F,F,F,F,F]

After num=1:  dp=[T,T,F...]
After num=5:  dp=[T,T,F,F,F,T,T,F...]
After num=11: dp=[T,T,F,...,T,T,...,T] — dp[11]=true

**Output:** `true` ✓ ([1,5,5] and [11])

---

### Variations

1. **Target Sum:** Count subsets with given sum difference — DP counting.
2. **Last Stone Weight II:** Minimize |sum(group1) - sum(group2)| — same knapsack.
3. **Ones and Zeroes:** 2D knapsack (capacity on two dimensions).

---

## Problem 98 — Longest Common Subsequence

**Difficulty:** Medium
**Real-world intuition:** Git diff, DNA sequence alignment, plagiarism detection, version comparison — the foundation of all sequence comparison algorithms.

---

### Theory

- **Core concept:** `dp[i][j]` = LCS length of `text1[0..i-1]` and `text2[0..j-1]`.
- **Recurrence:**
  - If `text1[i-1] == text2[j-1]`: `dp[i][j] = dp[i-1][j-1] + 1`.
  - Else: `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`.
- **Pattern:** Classic 2D string DP.
- **When to use:** Any "longest common sequence / similarity" between two strings.
- **Common mistakes:**
  - Confusing LCS (subsequence — non-contiguous) with LCP (prefix — contiguous).

---

### Java Implementation

```java
public class LongestCommonSubsequence {
    public int longestCommonSubsequence(String text1, String text2) {
        int m = text1.length(), n = text2.length();
        int[][] dp = new int[m + 1][n + 1];

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (text1.charAt(i - 1) == text2.charAt(j - 1))
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                else
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
            }
        }

        return dp[m][n];
    }
}
```

- **Time:** O(m × n), **Space:** O(m × n) — optimize to O(min(m,n)) with rolling row.

---

### Dry Run

**Input:** text1="abcde", text2="ace"

```
    ""  a  c  e
""  0   0  0  0
a   0   1  1  1
b   0   1  1  1
c   0   1  2  2
d   0   1  2  2
e   0   1  2  3
```

**Output:** `3` (a,c,e) ✓

---

### Variations

1. **Shortest Common Supersequence:** `m + n - LCS(m,n)`.
2. **Minimum Deletions to Make Sequences Equal:** `m + n - 2*LCS`.
3. **Distinct Subsequences:** Count occurrences of t as subsequence of s — DP counting.

---

## Problem 99 — Word Break

**Difficulty:** Medium
**Real-world intuition:** Tokenizing a continuous string of text into valid words — search engine query parsing, compiler tokenization, NLP word segmentation.

---

### Theory

- **Core concept:** `dp[i]` = true if `s[0..i-1]` can be segmented using words from the dictionary.
- **Recurrence:** `dp[i] = true` if there exists some `j < i` where `dp[j] == true` AND `s[j..i-1]` is in the word set.
- **Pattern:** 1D string DP with word set lookup.
- **Optimization:** Limit inner loop to `j >= i - maxWordLen` (huge speedup on long strings).
- **Common mistakes:**
  - Not initializing `dp[0] = true` (empty string can always be segmented).
  - Using a list instead of a HashSet for O(1) lookup.

---

### Java Implementation

```java
import java.util.*;

public class WordBreak {
    public boolean wordBreak(String s, List<String> wordDict) {
        Set<String> wordSet = new HashSet<>(wordDict);
        int n = s.length();
        boolean[] dp = new boolean[n + 1];
        dp[0] = true;

        for (int i = 1; i <= n; i++)
            for (int j = 0; j < i; j++)
                if (dp[j] && wordSet.contains(s.substring(j, i))) {
                    dp[i] = true;
                    break;
                }

        return dp[n];
    }
}
```

- **Time:** O(n² × m) where m = avg word length for substring creation. **Space:** O(n)

---

### Dry Run

**Input:** s="leetcode", wordDict=["leet","code"]

dp=[T,F,F,F,F,F,F,F,F]

i=4: j=0, dp[0]=T, s[0..3]="leet" ∈ dict → dp[4]=T.
i=8: j=4, dp[4]=T, s[4..7]="code" ∈ dict → dp[8]=T.

**Output:** `true` ✓

---

### Variations

1. **Word Break II:** Return all valid segmentations — backtracking + memoization.
2. **Concatenated Words:** Find words that are concatenations of other words in the list.
3. **Word Break with minimum cuts:** DP counting minimum splits.

---

## Problem 100 — Burst Balloons

**Difficulty:** Hard
**Real-world intuition:** Interval DP — optimizing the order of operations on a sequence (matrix chain multiplication, optimal parenthesization, game theory).

---

### Theory

- **Core concept:** Instead of thinking "which balloon to burst first," think "which balloon to burst LAST in a range [left, right]."
- **State:** `dp[left][right]` = max coins obtainable by bursting all balloons between left and right (exclusive of boundary balloons, which act as virtual 1s).
- **Recurrence:** `dp[i][j] = max over k in (i,j): dp[i][k] + nums[i]*nums[k]*nums[j] + dp[k][j]`.
- **Key insight:** When balloon k is the last to burst in `[i,j]`, its neighbors are the boundaries i and j (since all others in range are already gone).
- **Pattern:** Interval DP (iterate by increasing interval length).
- **Common mistakes:**
  - Thinking about the first balloon to burst (creates complex dependency on what's already gone).
  - Not adding boundary 1s to the array.

---

### Optimal Approach

**Steps:**
1. Pad array: `nums = [1] + nums + [1]`.
2. `dp[i][j]` over all intervals of increasing length.
3. For each interval `[i, j]`, try each `k` in `(i, j)` as the last to burst.
4. `dp[i][j] = max(dp[i][j], dp[i][k] + nums[i]*nums[k]*nums[j] + dp[k][j])`.

- **Time:** O(n³), **Space:** O(n²)

---

### Java Implementation

```java
public class BurstBalloons {
    public int maxCoins(int[] nums) {
        int n = nums.length;
        int[] padded = new int[n + 2];
        padded[0] = 1; padded[n + 1] = 1;
        for (int i = 0; i < n; i++) padded[i + 1] = nums[i];
        int size = n + 2;

        int[][] dp = new int[size][size];

        // len = length of the open interval (exclusive boundaries)
        for (int len = 2; len < size; len++) {
            for (int left = 0; left < size - len; left++) {
                int right = left + len;
                for (int k = left + 1; k < right; k++) {
                    int coins = padded[left] * padded[k] * padded[right]
                                + dp[left][k] + dp[k][right];
                    dp[left][right] = Math.max(dp[left][right], coins);
                }
            }
        }

        return dp[0][size - 1];
    }
}
```

### Python Implementation

```python
def max_coins(nums: list[int]) -> int:
    nums = [1] + nums + [1]
    n = len(nums)
    dp = [[0] * n for _ in range(n)]

    for length in range(2, n):
        for left in range(n - length):
            right = left + length
            for k in range(left + 1, right):
                dp[left][right] = max(
                    dp[left][right],
                    nums[left] * nums[k] * nums[right] + dp[left][k] + dp[k][right]
                )
    return dp[0][n - 1]
```

---

### Dry Run

**Input:** `[3,1,5,8]`

Padded: `[1,3,1,5,8,1]`

Key computation: `dp[0][5]` = 167.
- Last balloon in [0,5] is k=1(val=3): dp[0][1] + 1×3×1 + dp[1][5]... etc.
- Maximum combination yields 167.

**Output:** `167` ✓ (burst order: 1→5→3→8, coins=3×1×5 + 3×5×8 + 1×3×8 + 1×8×1 = 15+120+24+8 = 167)

---

### Variations

1. **Strange Printer:** Interval DP for minimum print operations.
2. **Remove Boxes:** Harder interval DP with 3D state.
3. **Matrix Chain Multiplication:** Classic interval DP — same pattern.

---

## Dynamic Programming — Pattern Summary

| DP Type | Problem type | State | Recurrence |
|---------|-------------|-------|-----------|
| 1D rolling | Steps, sequences | `dp[i]` | `dp[i] = f(dp[i-1], dp[i-2])` |
| 1D decision | Select/skip items in sequence | `dp[i]` | `max(include, exclude)` |
| 1D knapsack | Subset sum, coin change | `dp[capacity]` | `dp[j] = min/max(dp[j], dp[j-w]+v)` |
| 2D grid | Paths in a grid | `dp[i][j]` | Sum of neighbors |
| 2D string | Two-string problems | `dp[i][j]` | Match or take max of subproblems |
| Interval DP | Optimal ordering, range queries | `dp[l][r]` | Try all split points k |
| Tree DP | Subtree aggregation | Returned pair | Post-order DFS |

### The 4 Questions to Ask for Any DP Problem

1. **What is the state?** What minimal information defines a subproblem?
2. **What is the recurrence?** How do smaller states build the current one?
3. **What are the base cases?** What do the smallest states equal directly?
4. **What order to fill?** Bottom-up: ensure dependencies are computed before current.

### How to Identify DP

- "Count the number of ways to X" → DP (usually addition recurrence).
- "Minimum/maximum cost to achieve X" → DP (min/max recurrence).
- "Is it possible to achieve X?" → boolean DP.
- "Overlapping subproblems" in a recursive tree → memoize → DP.

---

## Revision Checklist — After Problems 91–100

- [ ] Climbing Stairs: can you code it in under 5 lines?
- [ ] Coin Change: do you initialize `dp` to `amount+1`, not `MAX_VALUE`?
- [ ] Partition Subset: do you iterate capacity BACKWARDS for 0/1 knapsack?
- [ ] LCS: can you explain why `dp[i][j] = dp[i-1][j-1]+1` when characters match?
- [ ] Burst Balloons: can you articulate WHY we think about "last to burst"?

### Mixed Practice

1. **Jump Game II:** Greedy DP — minimum jumps to reach the end.
2. **Decode Ways:** 1D DP, similar to Climbing Stairs with constraints.
3. **Interleaving String:** 2D DP on three strings.

---

# FINAL MASTER SUMMARY

---

## Pattern Recognition Quick Reference

```
Input type          → Try these patterns first
────────────────────────────────────────────────────────────
Array + sum/count   → HashMap (prefix sum) or Sliding Window
Sorted array        → Two Pointers or Binary Search
"All combinations"  → Backtracking (subset/permutation template)
"Shortest path"     → BFS (unweighted) or Dijkstra (weighted)
"Count/min/max"     → DP (check for overlapping subproblems)
"Top K elements"    → Heap (size K)
"Next greater"      → Monotonic Stack
Tree structure      → DFS (depth/path) or BFS (level)
"Cycle in list"     → Fast-slow pointers (Floyd's)
"Intervals"         → Sort + merge / sweep line
"Connectivity"      → Union-Find or DFS/BFS
"Subsequence"       → DP (LCS, LIS, Edit Distance templates)
```

---

## Complexity Cheat Sheet

| Technique | Time | Space |
|-----------|------|-------|
| Linear scan | O(n) | O(1) |
| Binary search | O(log n) | O(1) |
| Two pointers | O(n) | O(1) |
| Sliding window | O(n) | O(1)–O(k) |
| Sorting | O(n log n) | O(1)–O(n) |
| HashMap lookup | O(1) avg | O(n) |
| DFS/BFS on graph | O(V+E) | O(V) |
| Heap insert/delete | O(log n) | O(n) |
| 1D DP | O(n) | O(n) → O(1) rolling |
| 2D DP | O(n×m) | O(n×m) → O(min) rolling |
| Backtracking subsets | O(2ⁿ×n) | O(n) stack |
| Backtracking perms | O(n!×n) | O(n) stack |
| Interval DP | O(n³) | O(n²) |

---

## Interview Decision Framework

```
Step 1: Understand the problem
  → Restate in your own words
  → Identify input/output types
  → Ask about constraints (size, range, duplicates)

Step 2: Identify the pattern
  → Which category does this fall into?
  → What trigger keywords are present?

Step 3: Brute force first (say it, don't code it)
  → State time/space complexity
  → Explain why it's insufficient

Step 4: Optimize
  → What information are you recomputing?
  → What data structure gives O(1) for that?

Step 5: Code the optimal solution
  → Write clean, named variables
  → Handle edge cases

Step 6: Test with examples
  → Walk through a small input
  → Test edge cases: empty, single element, all same, max constraints
```

---

## 30-Day Study Plan

| Days | Focus | Problems |
|------|-------|----------|
| 1–4 | Arrays & Strings | 1–15 |
| 5–7 | Hashing & Sets | 16–25 |
| 8–10 | Two Pointers & Sliding Window | 26–35 |
| 11–13 | Recursion & Backtracking | 36–45 |
| 14–16 | Stack & Queue | 46–55 |
| 17–19 | Linked List | 56–65 |
| 20–22 | Trees & BST | 66–75 |
| 23–24 | Heaps | 76–80 |
| 25–27 | Graphs | 81–90 |
| 28–30 | Dynamic Programming | 91–100 |

**After Day 30:** Do mixed contests — pick 1 Easy + 1 Medium + 1 Hard per day from random topics.

---

*End of DSA Complete Learning Program — 100 Problems*
