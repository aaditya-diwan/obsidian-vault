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

