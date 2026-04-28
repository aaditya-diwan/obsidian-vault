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

