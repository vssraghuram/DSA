<!-- GitHub Project Banner -->
<p align="center">
  <img src="https://github.com/user-attachments/assets/501c98ee-809c-4376-b32d-6d38ae07c489" alt="Project Banner" width="100%">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/VSR%20Expo-black?logo=cplusplus&logoColor=white&labelColor=black&color=black">
  <img src="https://img.shields.io/badge/Built%20by-RaghuRam-black?labelColor=black&color=black">
</p>

<p align="center">
  <a href="https://www.youtube.com/YOUR_VIDEO_LINK_HERE">
    <img src="https://img.shields.io/badge/Watch%20on-YouTube-black?logo=youtube&logoColor=FF0000&labelColor=black&color=black" alt="Watch on YouTube">
  </a>
</p>

# » VSR Expo – Graphs in C++

## » Table of Contents
- Introduction
- Why It Matters
- Real-World Implementation
- Alternative Ways to Implement It
- Most Efficient & Effective Approach
- Quick Reference Table
- Author Footer

## » Introduction
A **graph** is a data structure made of vertices (dots) connected by edges (lines), with no enforced hierarchy — unlike a tree, any vertex can connect to any other. Every graph is classified along two axes: **directed vs. undirected** (can you traverse an edge one way or both?) and **weighted vs. unweighted** (does the edge carry a cost?). This classification is the first decision a senior engineer makes, because it immediately narrows the algorithm family: BFS/DFS for structure and unweighted reachability, Dijkstra/Bellman-Ford/Floyd-Warshall for weighted shortest paths, Kruskal/Prim for spanning trees, Kahn's/DFS-topo for dependency ordering, and Kosaraju/Tarjan for strongly connected components in directed graphs. This doc consolidates the full toolkit — representation, traversal, shortest paths, MST, topological sort, cycle detection, SCCs, and bipartiteness — into one reference.

## » Why It Matters
Graphs model *relationships*, and relationships are everywhere in production systems: road networks (Google Maps, Uber), social graphs (follow/friend edges on Instagram/X), dependency graphs (build systems, package managers, task schedulers), network routing (BGP, OSPF), and even neural network topology. The reason this matters at a senior level isn't the definition — it's that picking the wrong representation or algorithm for the graph's shape (dense vs. sparse, directed vs. undirected, negative weights or not) silently blows up your complexity: an adjacency matrix on a sparse million-node social graph wastes `O(V²)` memory you'll never use, and running Dijkstra on a graph with negative edges gives you a *wrong* answer, not just a slow one.

## » Real-World Implementation
The adjacency list is the representation used in virtually all production graph code, because real-world graphs (road networks, social graphs, dependency graphs) are sparse — most vertices connect to a handful of others, not all of them.

```cpp
#include <list>
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
        l[v].push_back(u); // remove this line for a directed graph
    }
};
```

On top of this representation, shortest-path routing (e.g. a mapping service computing driving directions) is implemented with Dijkstra's algorithm using a min-heap, since road weights (distance/time) are never negative:

```cpp
vector<long long> dijkstra(int src, int n, vector<vector<pair<int,int>>>& adj) {
    vector<long long> dist(n, LLONG_MAX);
    priority_queue<pair<long long,int>, vector<pair<long long,int>>, greater<>> pq;
    dist[src] = 0;
    pq.push({0, src});

    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (d > dist[u]) continue; // stale entry, skip
        for (auto [v, w] : adj[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}
```

`dist[]` holds the best-known cost to each vertex; the heap always pops the currently-cheapest frontier vertex next, and stale heap entries (superseded by a shorter path found later) are skipped rather than removed, which is what keeps the implementation simple without needing a decrease-key operation.

## » Alternative Ways to Implement It

### 1. Bellman-Ford (instead of Dijkstra)
- **How it works:** Relaxes every edge `V - 1` times; a final pass detects negative cycles.
- **Best when:** The graph has negative edge weights (e.g. arbitrage detection in currency exchange graphs, or cost graphs where some transitions are "refunds").
- **Trade-off vs. primary approach:** `O(V · E)` instead of `O(E log V)` — much slower on large graphs, but it's the only single-source option that's *correct* in the presence of negative weights.

### 2. Floyd-Warshall (instead of running Dijkstra per source)
- **How it works:** DP over intermediate vertices `k`, computing all-pairs shortest paths in one pass: `dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])`.
- **Best when:** You need shortest paths between *every* pair of vertices (e.g. precomputed travel-time matrices between all warehouses) and `V` is small-to-moderate (roughly under a few thousand).
- **Trade-off vs. primary approach:** `O(V³)` time and `O(V²)` space — worse than running Dijkstra `V` times (`O(V·E log V)`) once the graph is sparse, but simpler to implement correctly and fine for dense, small graphs.

### 3. A* Search (instead of plain Dijkstra)
- **How it works:** Same min-heap structure as Dijkstra, but adds a heuristic `h(v)` (e.g. straight-line distance to the target) to prioritize exploring vertices likely to be on the shortest path.
- **Best when:** You have a single fixed target and a good admissible heuristic — this is what real routing engines (GPS navigation) use instead of plain Dijkstra.
- **Trade-off vs. primary approach:** Much faster in practice for point-to-point queries, but requires a domain-specific heuristic; a bad heuristic can make it incorrect (non-admissible) or no better than Dijkstra.

## » Most Efficient & Effective Approach
For the common case — sparse graph, non-negative weights, single source — **adjacency list + Dijkstra with a binary heap** is the right default: `O(E log V)` is fast enough for graphs with millions of edges, and it's the version every standard library and interview setting expects. Reach for **Bellman-Ford** only when negative weights are possible, **Floyd-Warshall** only when you genuinely need all-pairs results on a graph small enough for `O(V³)`, and **A\*** when you have a single fixed target and a real heuristic (e.g. geographic coordinates). For pure structure questions with no weights — reachability, cycle detection, bipartiteness, topological order, SCCs — BFS/DFS variants are always `O(V + E)` and are the correct default; there's rarely a reason to reach for anything heavier.

## » Quick Reference Table

| Approach | Best For | Trade-off |
|---|---|---|
| Dijkstra (min-heap) | Single-source shortest path, non-negative weights | Fails silently-wrong on negative edges |
| Bellman-Ford | Negative weights, negative-cycle detection | `O(V·E)` — much slower than Dijkstra |
| Floyd-Warshall | All-pairs shortest paths, small/dense graphs | `O(V³)` time and `O(V²)` space |
| A* Search | Single target + good heuristic (e.g. GPS routing) | Needs a domain-specific admissible heuristic |
| BFS / DFS | Unweighted reachability, cycle/bipartite/topo/SCC | No cost modeling — structure only |
| Kruskal's (DSU) | MST, edge list already available | `O(E log E)` sort dominates |
| Prim's (min-heap) | MST, adjacency list already available | Similar cost to Kruskal, different structure |

## » Author Footer

**Created By:** Vemparala Sri Satya RaghuRam
**License:** MIT
**Platform:** C++ (Graph Theory / DSA)

<p align="left">
  <img src="https://img.shields.io/badge/%23BeyondCertifications-black?logo=tag&logoColor=white&labelColor=black&color=black">
  <img src="https://img.shields.io/badge/%23IndustryOriented-black?logo=tag&logoColor=white&labelColor=black&color=black">
  <img src="https://img.shields.io/badge/%23CodeWithRaghuRam-black?logo=tag&logoColor=white&labelColor=black&color=black">
</p>
