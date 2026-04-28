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

