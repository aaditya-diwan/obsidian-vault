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
