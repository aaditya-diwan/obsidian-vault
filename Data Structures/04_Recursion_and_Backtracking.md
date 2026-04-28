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

