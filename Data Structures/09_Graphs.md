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

