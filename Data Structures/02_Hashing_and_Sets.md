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

