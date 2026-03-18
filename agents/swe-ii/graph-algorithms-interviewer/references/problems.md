# Problem Bank: Graph Algorithms

## Medium Problems

### 1. Number of Islands

**Statement**: Given an `m x n` 2D grid of `'1'`s (land) and `'0'`s (water), count the number of islands. An island is formed by connecting adjacent lands horizontally or vertically.

**Example**:
```
Input:
  grid = [
    ["1","1","0","0","0"],
    ["1","1","0","0","0"],
    ["0","0","1","0","0"],
    ["0","0","0","1","1"]
  ]
Output: 3
```

**Optimal**: BFS or DFS flood fill, O(m*n) time, O(m*n) space
**Alternative**: Union-Find, O(m*n * alpha(m*n)) time

**Walkthrough**:
```
Grid (mark visited with '#'):

Step 1: Find first '1' at (0,0). Island count = 1.
  BFS from (0,0): visit (0,0), (0,1), (1,0), (1,1)
  Grid after:
    # # 0 0 0
    # # 0 0 0
    0 0 1 0 0
    0 0 0 1 1

Step 2: Skip '0's and '#'s. Find '1' at (2,2). Island count = 2.
  BFS from (2,2): visit (2,2)
  Grid after:
    # # 0 0 0
    # # 0 0 0
    0 0 # 0 0
    0 0 0 1 1

Step 3: Find '1' at (3,3). Island count = 3.
  BFS from (3,3): visit (3,3), (3,4)
  Grid after:
    # # 0 0 0
    # # 0 0 0
    0 0 # 0 0
    0 0 0 # #

No more '1's. Return 3.
```

**Follow-ups**:
- What if the grid wraps around (toroidal)? How does that change adjacency?
- Can you solve it with Union-Find instead of BFS/DFS? What are the trade-offs?
- What if diagonals also count as connected?

---

### 2. Clone Graph

**Statement**: Given a reference to a node in a connected undirected graph, return a deep copy of the graph. Each node contains a value and a list of neighbors.

**Example**:
```
Input: adjList = [[2,4],[1,3],[2,4],[1,3]]
  Node 1 -- Node 2
  |              |
  Node 4 -- Node 3

Output: Deep copy of the same graph with new node objects
```

**Optimal**: BFS or DFS with HashMap (old node -> new node), O(V+E) time, O(V) space

**Walkthrough**:
```
DFS approach with cloned = {} (maps original -> clone)

Visit node 1:
  Create clone of 1. cloned = {1: clone_1}
  Process neighbor 2:
    Create clone of 2. cloned = {1: clone_1, 2: clone_2}
    Process neighbor 1: already cloned, skip
    Process neighbor 3:
      Create clone of 3. cloned = {1: c1, 2: c2, 3: c3}
      Process neighbor 2: already cloned, skip
      Process neighbor 4:
        Create clone of 4. cloned = {1: c1, 2: c2, 3: c3, 4: c4}
        Process neighbor 1: already cloned, skip
        Process neighbor 3: already cloned, skip
      Add c4 to c3.neighbors
    Add c3 to c2.neighbors
  Add c2 to c1.neighbors
  Process neighbor 4: already cloned
  Add c4 to c1.neighbors

Return clone_1
```

**Follow-ups**:
- What if the graph is directed?
- What if nodes can have self-loops?
- How would you verify the clone is correct (same structure, different objects)?

---

### 3. Course Schedule (Cycle Detection / Topological Sort)

**Statement**: There are `numCourses` courses labeled `0` to `numCourses-1`. Given an array `prerequisites` where `prerequisites[i] = [a, b]` means you must take course `b` before course `a`, determine if you can finish all courses.

**Example**:
```
Input: numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]

  0 --> 1 --> 3
  |           ^
  +--> 2 -----+

Output: true (order: 0, 1, 2, 3 or 0, 2, 1, 3)
```

**Optimal (Kahn's Algorithm)**: BFS-based topological sort, O(V+E) time, O(V+E) space
**Alternative**: DFS with 3-color marking for cycle detection

**Walkthrough (Kahn's Algorithm)**:
```
Build adjacency list and in-degree array:
  adj = {0: [1,2], 1: [3], 2: [3], 3: []}
  in_degree = {0:0, 1:1, 2:1, 3:2}

Queue nodes with in-degree 0: [0]
processed = 0

Step 1: Dequeue 0. processed = 1.
  Decrement neighbors: in_degree[1] = 0, in_degree[2] = 0
  Queue: [1, 2]

Step 2: Dequeue 1. processed = 2.
  Decrement neighbors: in_degree[3] = 1
  Queue: [2]

Step 3: Dequeue 2. processed = 3.
  Decrement neighbors: in_degree[3] = 0
  Queue: [3]

Step 4: Dequeue 3. processed = 4.
  No neighbors.
  Queue: []

processed (4) == numCourses (4) -> return true
```

**Cycle example**:
```
Input: numCourses = 2, prerequisites = [[0,1],[1,0]]

  0 --> 1
  ^     |
  +-----+

in_degree = {0:1, 1:1}
No node has in-degree 0 -> Queue is empty from the start
processed (0) != numCourses (2) -> return false (cycle detected)
```

**Follow-ups**:
- Return one valid course ordering (Course Schedule II)
- What if you need all valid topological orderings?
- How would you handle the case where some courses have no prerequisites?

---

### 4. Pacific Atlantic Water Flow

**Statement**: Given an `m x n` grid of heights, find all cells where water can flow to both the Pacific (top/left edges) and Atlantic (bottom/right edges) oceans. Water flows from a cell to an adjacent cell with equal or lower height.

**Example**:
```
Input:
  heights = [
    [1, 2, 2, 3, 5],
    [3, 2, 3, 4, 4],
    [2, 4, 5, 3, 1],
    [6, 7, 1, 4, 5],
    [5, 1, 1, 2, 4]
  ]
Output: [[0,4],[1,3],[1,4],[2,2],[3,0],[3,1],[4,0]]
```

**Optimal**: Reverse BFS/DFS from ocean edges, O(m*n) time, O(m*n) space

**Walkthrough**:
```
Key insight: Instead of flowing water downhill from every cell,
reverse the flow -- start from ocean edges and go UPHILL.

Step 1: BFS from all Pacific-bordering cells (top row + left column)
  Mark all cells reachable by flowing uphill from Pacific edge.
  pacific_reachable = set of cells

Step 2: BFS from all Atlantic-bordering cells (bottom row + right column)
  Mark all cells reachable by flowing uphill from Atlantic edge.
  atlantic_reachable = set of cells

Step 3: Answer = pacific_reachable INTERSECT atlantic_reachable

Why reverse? Forward approach checks every cell -> O(m*n) BFS each = O((m*n)^2).
Reverse approach: only 2 BFS passes = O(m*n) total.
```

**Follow-ups**:
- What if water could also flow diagonally?
- Can you solve it with Union-Find?
- What if heights could be negative?

---

### 5. Graph Valid Tree

**Statement**: Given `n` nodes labeled `0` to `n-1` and a list of undirected edges, check whether these edges form a valid tree. A valid tree has exactly `n-1` edges, is connected, and has no cycles.

**Example**:
```
Input: n = 5, edges = [[0,1],[0,2],[0,3],[1,4]]

    0
   /|\
  1 2 3
  |
  4

Output: true (connected, n-1 = 4 edges, no cycles)
```

**Optimal**: Union-Find, O(E * alpha(V)) time, O(V) space
**Alternative**: BFS/DFS checking for cycles and connectivity

**Walkthrough (Union-Find)**:
```
Initialize: parent = [0, 1, 2, 3, 4], rank = [0, 0, 0, 0, 0]

Check: edges count must be n-1 = 4. We have 4 edges. OK.

Process edge [0,1]:
  find(0) = 0, find(1) = 1 -> different sets, union them
  parent = [0, 0, 2, 3, 4]

Process edge [0,2]:
  find(0) = 0, find(2) = 2 -> different sets, union them
  parent = [0, 0, 0, 3, 4]

Process edge [0,3]:
  find(0) = 0, find(3) = 3 -> different sets, union them
  parent = [0, 0, 0, 0, 4]

Process edge [1,4]:
  find(1) = 0, find(4) = 4 -> different sets, union them
  parent = [0, 0, 0, 0, 0]

No edge caused a cycle (same-set union). All nodes connected. -> true
```

**Cycle detection example**:
```
Input: n = 4, edges = [[0,1],[1,2],[2,3],[3,0]]

Process edges [0,1], [1,2], [2,3] -> all fine, unions succeed.
Process edge [3,0]:
  find(3) = 0, find(0) = 0 -> SAME SET -> cycle detected -> false
```

**Follow-ups**:
- What if the graph is directed -- how does the definition of a tree change?
- Can you detect which edge causes the cycle?
- How does path compression improve Union-Find performance?

---

### 6. Network Delay Time (Dijkstra)

**Statement**: Given a network of `n` nodes and directed weighted edges `times[i] = [u, v, w]`, find the time it takes for a signal from node `k` to reach all nodes. Return -1 if not all nodes are reachable.

**Example**:
```
Input: times = [[2,1,1],[2,3,1],[3,4,1]], n = 4, k = 2

  2 --1--> 1
  |
  1
  |
  v
  3 --1--> 4

Output: 2 (signal reaches 1 in 1, 3 in 1, 4 in 2. Max = 2)
```

**Optimal**: Dijkstra with min-heap, O(E log V) time, O(V+E) space

**Walkthrough**:
```
Adjacency list: {2: [(1,1), (3,1)], 3: [(4,1)]}
dist = {1: inf, 2: 0, 3: inf, 4: inf}
Min-heap: [(0, 2)]

Step 1: Pop (0, 2). Process node 2.
  Neighbor 1: dist[1] = min(inf, 0+1) = 1. Push (1, 1).
  Neighbor 3: dist[3] = min(inf, 0+1) = 1. Push (1, 3).
  dist = {1: 1, 2: 0, 3: 1, 4: inf}
  Heap: [(1, 1), (1, 3)]

Step 2: Pop (1, 1). Process node 1.
  No neighbors.
  dist = {1: 1, 2: 0, 3: 1, 4: inf}
  Heap: [(1, 3)]

Step 3: Pop (1, 3). Process node 3.
  Neighbor 4: dist[4] = min(inf, 1+1) = 2. Push (2, 4).
  dist = {1: 1, 2: 0, 3: 1, 4: 2}
  Heap: [(2, 4)]

Step 4: Pop (2, 4). Process node 4.
  No neighbors.
  dist = {1: 1, 2: 0, 3: 1, 4: 2}
  Heap: []

All nodes reachable. Answer = max(dist values) = 2.
```

**Follow-ups**:
- What if some edges have negative weights? (Dijkstra fails -- use Bellman-Ford)
- What if you only need the shortest path to one specific target?
- How would you reconstruct the actual shortest path, not just the distance?

---

## Advanced Problems (For candidates who ace everything)

### 7. Alien Dictionary

**Statement**: Given a sorted list of words in an alien language, derive the order of characters in the alien alphabet. Return the order as a string. If the order is invalid (cycle), return "". If multiple valid orderings exist, return any.

**Example**:
```
Input: words = ["wrt", "wrf", "er", "ett", "rftt"]

Compare adjacent words to extract ordering rules:
  "wrt" vs "wrf" -> t < f  (first difference at index 2)
  "wrf" vs "er"  -> w < e  (first difference at index 0)
  "er" vs "ett"  -> r < t  (first difference at index 1)
  "ett" vs "rftt"-> e < r  (first difference at index 0)

Directed edges: t->f, w->e, r->t, e->r

Graph:
  w -> e -> r -> t -> f

Topological sort: "wertf"
Output: "wertf"
```

**Optimal**: Topological sort (Kahn's or DFS), O(C) time where C = total characters across all words

**Walkthrough**:
```
Step 1: Extract edges from adjacent word comparisons
  edges = [(t,f), (w,e), (r,t), (e,r)]
  All unique chars: {w, r, t, e, f}

Step 2: Build adjacency list and in-degree
  adj = {t: [f], w: [e], r: [t], e: [r], f: []}
  in_degree = {w:0, e:1, r:1, t:1, f:1}

Step 3: Kahn's algorithm
  Queue: [w]  (in-degree 0)
  result = []

  Process w -> decrement e. Queue: [e]. result: [w]
  Process e -> decrement r. Queue: [r]. result: [w, e]
  Process r -> decrement t. Queue: [t]. result: [w, e, r]
  Process t -> decrement f. Queue: [f]. result: [w, e, r, t]
  Process f -> no neighbors.  result: [w, e, r, t, f]

  len(result) == len(unique chars) -> valid ordering
  Return "wertf"
```

**Edge cases**:
- `["abc", "ab"]` -> invalid (prefix appears after longer word)
- `["z", "z"]` -> only character is "z", return "z"
- Cycle in derived edges -> return ""

**Follow-ups**:
- How many valid orderings could exist? How would you enumerate them all?
- What if two words are identical? Does that affect the result?
- How do you handle the prefix edge case efficiently?

---

## Sample Session Flows

### Session Flow 1: Candidate Struggles (Easy Path)

**Interviewer**: "When would you represent a graph as an adjacency list vs an adjacency matrix?"
**Candidate**: "I'm not sure... I think adjacency list uses less memory?"
**Interviewer**: "That's correct for sparse graphs. An adjacency list uses O(V+E) space while a matrix uses O(V^2). For most interview problems, graphs are sparse, so we use lists. Let's start with something concrete."

*[Switch to Number of Islands -- grid-based, intuitive]*

**Interviewer**: "Given this grid of 1s and 0s, how many islands are there? Think of each cell as a node."
**Candidate**: "I'd check each cell..."
**Interviewer**: "Good start. When you find a '1', how do you find the entire island it belongs to?"
**Candidate**: "Maybe... visit all the connected 1s?"
**Interviewer**: "Exactly. That's BFS or DFS. Let's walk through it step by step on this small grid."

*[Guide through BFS flood fill with ASCII visualization]*

### Session Flow 2: Candidate Excels (Hard Path)

**Interviewer**: "When would you use an adjacency list vs matrix?"
**Candidate**: "List for sparse graphs -- O(V+E) space, O(degree) edge lookup. Matrix for dense graphs -- O(V^2) space but O(1) edge lookup. Most interview problems are sparse."
**Interviewer**: "Solid. Tell me the difference between what BFS and DFS guarantee."
**Candidate**: "BFS gives shortest path in unweighted graphs due to level-order traversal. DFS is useful for topological sort, cycle detection, and path-finding but doesn't guarantee shortest path."

*[Skip to Network Delay Time -- weighted shortest path]*

**Interviewer**: "Given a weighted directed graph, find the time for a signal to reach all nodes from a source. What algorithm would you use?"
**Candidate**: *[Immediately identifies Dijkstra, implements with min-heap]*
**Interviewer**: "Good. Follow-up: what if some edges could have negative weights?"

*[Challenge with Alien Dictionary if time permits]*

### Session Flow 3: Average Candidate (Standard Path)

**Interviewer**: "When would you choose an adjacency list over an adjacency matrix?"
**Candidate**: "When the graph doesn't have too many edges, I think."
**Interviewer**: "Right -- sparse graphs. Let's talk about BFS vs DFS. When would you use each?"
**Candidate**: "BFS uses a queue, DFS uses a stack or recursion."

*[Start with Course Schedule -- topological sort]*

**Interviewer**: "You have courses with prerequisites. How do you determine if you can complete all courses?"
**Candidate**: "I think... if there's a cycle you can't?"
**Interviewer**: "Exactly. So this is a cycle detection problem on a directed graph. How would you detect cycles?"
**Candidate**: "Maybe track what I've visited?"
**Interviewer**: "You're on the right track. There's a classic algorithm called topological sort that solves this. Let me show you the in-degree approach..."

*[Guide through Kahn's algorithm with the ASCII visualization, then if time remains, move to Number of Islands]*
