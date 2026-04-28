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

