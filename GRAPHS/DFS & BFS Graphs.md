# DFS & BFS — Ground-Up Beginner Guide

Two ways to visit every reachable vertex in a graph, starting from one vertex. Same graph class as before, two new functions added on top.

---

## 1. The Graph We're Using

Same 5-vertex graph from before, built with:

```cpp
Graph g(5);
g.addEdge(0, 1);
g.addEdge(1, 2);
g.addEdge(1, 3);
g.addEdge(2, 3);
g.addEdge(2, 4);
```

Which gives:
- `l[0] = {1}`
- `l[1] = {0, 2, 3}`
- `l[2] = {1, 3, 4}`
- `l[3] = {1, 2}`
- `l[4] = {2}`

Both DFS and BFS answer the same question — **"starting from one vertex, visit every vertex you can reach"** — but they do it in a different order.

---

## 2. Why We Need a `visited` Array First

Before either traversal, there's one problem to solve: the graph has cycles (e.g. `1 → 2 → 3 → 1`). Without tracking who's already been visited, the code would go in circles forever.

That's what `vector<bool> vis(V, false)` is for — one `true`/`false` flag per vertex, starting all `false`. Every time you visit a vertex, you flip its flag to `true`, and you check that flag before visiting again.

This one array is shared by both DFS and BFS below — it's not specific to either one.

---

## 3. DFS — Depth First Search

**The idea:** go as deep as you can down one path first. Only when you hit a dead end (no unvisited neighbors left) do you come back and try a different path.

**How it's built:** the function calls *itself* on each unvisited neighbor. No extra data structure needed — the function call stack does the "remembering where to come back to" for you automatically.

```cpp
void dfsHelper(int u, vector<bool> &vis) {
    cout << u << " ";
    vis[u] = true;

    for (int v : l[u]) {
        if (!vis[v]) {
            dfsHelper(v, vis);
        }
    }
}

void dfs(int src) {
    vector<bool> vis(V, false);
    dfsHelper(src, vis);
}
```

### Line by line
- `cout << u << " ";` — print/visit `u` the moment we arrive at it.
- `vis[u] = true;` — mark it visited immediately, so nobody comes back to it.
- `for (int v : l[u])` — look at every neighbor of `u`.
- `if (!vis[v]) dfsHelper(v, vis);` — if that neighbor hasn't been visited, go there *right now*, deeper — don't finish checking `u`'s other neighbors first.

### Dry run from vertex 0
```
Call dfsHelper(0) → print 0, visited = {0}
  neighbor 1 not visited → dfsHelper(1) → print 1, visited = {0,1}
    neighbor 0 visited → skip
    neighbor 2 not visited → dfsHelper(2) → print 2, visited = {0,1,2}
      neighbor 1 visited → skip
      neighbor 3 not visited → dfsHelper(3) → print 3, visited = {0,1,2,3}
        neighbor 1 visited → skip
        neighbor 2 visited → skip
        (no more neighbors, dead end — go back up)
      neighbor 4 not visited → dfsHelper(4) → print 4, visited = {0,1,2,3,4}
        neighbor 2 visited → skip
        (dead end — go back up, and up, and up — nothing left to do)
```
**Output:** `0 1 2 3 4`

Notice it went 0 → 1 → 2 → 3 first (all the way down), and only reached 4 after backing all the way out to vertex 2.

---

## 4. BFS — Breadth First Search

**The idea:** visit `src`, then visit ALL of its direct neighbors, then ALL of their neighbors — one full ring outward at a time, like ripples in a pond.

**How it's built:** uses a `queue`, not recursion. A queue is first-in-first-out — whoever gets added first, gets processed first — which is exactly what keeps the "ring by ring" order correct.

```cpp
void bfs(int src) {
    vector<bool> vis(V, false);
    queue<int> q;

    vis[src] = true;
    q.push(src);

    while (!q.empty()) {
        int u = q.front();
        q.pop();
        cout << u << " ";

        for (int v : l[u]) {
            if (!vis[v]) {
                vis[v] = true;
                q.push(v);
            }
        }
    }
}
```

### Line by line
- `vis[src] = true; q.push(src);` — mark the start visited and put it in the queue first.
- `while (!q.empty())` — keep going as long as there's someone waiting in line.
- `int u = q.front(); q.pop();` — take the person at the front of the queue out, and process them.
- `cout << u << " ";` — visit/print `u`.
- `for (int v : l[u])` — check `u`'s neighbors.
- `if (!vis[v]) { vis[v] = true; q.push(v); }` — if not visited, mark it visited **immediately** (not later) and add it to the back of the queue, to be processed after everyone already waiting.

**Important beginner trap:** in BFS you mark `vis[v] = true` the moment you *push* it into the queue — not when you later pop and print it. If you wait until popping to mark it visited, the same vertex can get pushed into the queue multiple times by different neighbors before it's ever processed.

### Dry run from vertex 0
```
Start: vis[0]=true, queue = [0]

Pop 0 → print 0
  neighbor 1 not visited → mark visited, push → queue = [1]

Pop 1 → print 1
  neighbor 0 visited → skip
  neighbor 2 not visited → mark visited, push → queue = [2]
  neighbor 3 not visited → mark visited, push → queue = [2, 3]

Pop 2 → print 2
  neighbor 1 visited → skip
  neighbor 3 visited → skip
  neighbor 4 not visited → mark visited, push → queue = [3, 4]

Pop 3 → print 3
  neighbor 1 visited → skip
  neighbor 2 visited → skip
  queue = [4]

Pop 4 → print 4
  neighbor 2 visited → skip
  queue = [] → done
```
**Output:** `0 1 2 3 4`

Same output for this particular graph, but for a different-shaped graph the orders would clearly differ — DFS commits to one path, BFS spreads outward evenly.

---

## 5. The Core Difference (memorize this)

| | DFS | BFS |
|---|---|---|
| **Strategy** | Go deep first, backtrack on dead end | Go wide first, one layer at a time |
| **Data structure** | Function call stack (via recursion) | Explicit `queue` |
| **When is a vertex marked visited?** | Right when you enter it | Right when you push it into the queue |
| **Real-life analogy** | Solving a maze — commit to one path until it's a dead end | Ripples in a pond — nearest things reached first |
| **Good for** | Exploring structure, detecting cycles, exhausting all paths | Shortest path in an unweighted graph (fewest edges to reach a vertex) |

**One sentence to remember it by:** DFS asks "where does this path lead?", BFS asks "who's closest to me right now?"

---

## 6. Full Working Code

```cpp
#include <iostream>
#include <list>
#include <vector>
#include <queue>
using namespace std;

class Graph {
    int V;
    list<int> *l;

public:
    Graph(int V) {
        this->V = V;
        l = new list<int>[V];
    }

    void addEdge(int u, int v) {
        l[u].push_back(v);
        l[v].push_back(u);
    }

    // DFS Traversal
    void dfsHelper(int u, vector<bool> &vis) {
        cout << u << " ";
        vis[u] = true;

        for (int v : l[u]) {
            if (!vis[v]) {
                dfsHelper(v, vis);
            }
        }
    }

    void dfs(int src) {
        vector<bool> vis(V, false);
        dfsHelper(src, vis);
    }

    // BFS Traversal
    void bfs(int src) {
        vector<bool> vis(V, false);
        queue<int> q;

        vis[src] = true;
        q.push(src);

        while (!q.empty()) {
            int u = q.front();
            q.pop();
            cout << u << " ";

            for (int v : l[u]) {
                if (!vis[v]) {
                    vis[v] = true;
                    q.push(v);
                }
            }
        }
    }
};

int main() {
    Graph g(5);
    g.addEdge(0, 1);
    g.addEdge(1, 2);
    g.addEdge(1, 3);
    g.addEdge(2, 3);
    g.addEdge(2, 4);

    cout << "DFS: ";
    g.dfs(0);
    cout << endl;

    cout << "BFS: ";
    g.bfs(0);
    cout << endl;

    return 0;
}
```

**Expected output:**
```
DFS: 0 1 2 3 4
BFS: 0 1 2 3 4
```
<center>
    
<img width="1536" height="1024" alt="3D4F709D-559B-46C7-986B-D4D7E324744B" src="https://github.com/user-attachments/assets/06846d25-a6b8-423e-81b6-76ad95eb85d3" />

</center>
