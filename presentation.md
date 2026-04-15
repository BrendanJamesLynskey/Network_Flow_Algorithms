# Network Flow Algorithms

**Computer Science Fundamentals Series**

Max-flow min-cut · Ford-Fulkerson · Edmonds-Karp · Dinic's · Bipartite matching · Cost flow

*Mid-level software engineer track -- 20 slides*

---

## Table of Contents

1. [Flow Networks -- Foundations](#slide-02--flow-networks--foundations)
2. [Capacity, Flow & Conservation](#slide-03--capacity-flow--conservation)
3. [Max-Flow Min-Cut Theorem](#slide-04--max-flow-min-cut-theorem)
4. [Residual Graphs & Augmenting Paths](#slide-05--residual-graphs--augmenting-paths)
5. [Ford-Fulkerson Method](#slide-06--ford-fulkerson-method)
6. [Edmonds-Karp Algorithm](#slide-07--edmonds-karp-algorithm)
7. [Dinic's Algorithm](#slide-08--dinics-algorithm)
8. [Blocking Flows & Level Graphs](#slide-09--blocking-flows--level-graphs)
9. [Push-Relabel Algorithm](#slide-10--push-relabel-algorithm)
10. [Push-Relabel -- Discharge & Heuristics](#slide-11--push-relabel--discharge--heuristics)
11. [Minimum Cost Flow](#slide-12--minimum-cost-flow)
12. [Minimum Cost Flow -- Algorithms](#slide-13--minimum-cost-flow--algorithms)
13. [Maximum Bipartite Matching](#slide-14--maximum-bipartite-matching)
14. [Hungarian Algorithm](#slide-15--hungarian-algorithm)
15. [Applications -- Transportation & Assignment](#slide-16--applications--transportation--assignment)
16. [Applications -- Image Segmentation & More](#slide-17--applications--image-segmentation--more)
17. [Network Flow Modelling Techniques](#slide-18--network-flow-modelling-techniques)
18. [Complexity Comparison](#slide-19--complexity-comparison)
19. [Summary & Further Reading](#slide-20--summary--further-reading)

---

## Slide 02 -- Flow Networks -- Foundations

### What is a flow network?

A directed graph `G = (V, E)` with two distinguished vertices and a capacity function:

- **Source (s)** -- where flow originates; no incoming flow constraint
- **Sink (t)** -- where flow is consumed; no outgoing flow constraint
- **Capacity c(u,v)** -- maximum flow allowed on edge (u,v); always non-negative
- **Flow f(u,v)** -- the actual flow on edge (u,v)

### Formal definition

A flow network is a tuple `(G, s, t, c)` where:

```
G = (V, E)    directed graph
s ∈ V         source vertex
t ∈ V         sink vertex
c: E → R⁺     capacity function
```

> Flow networks model any system where something moves through a constrained network -- water in pipes, data in networks, goods through supply chains.

---

## Slide 03 -- Capacity, Flow & Conservation

### Flow constraints

Every valid flow must satisfy two properties:

| Constraint | Formal | Meaning |
|-----------|--------|---------|
| **Capacity** | `0 ≤ f(u,v) ≤ c(u,v)` | Flow on an edge cannot exceed its capacity |
| **Conservation** | `Σ f(u,v) = Σ f(v,w)` for all v ≠ s,t | Flow into a node equals flow out of that node |

### Value of a flow

The total flow leaving the source (equivalently, arriving at the sink):

```
|f| = Σ f(s,v) - Σ f(v,s)
    = Σ f(v,t) - Σ f(t,v)
```

### Maximum flow problem

Find a flow `f` that maximises `|f|` while satisfying capacity and conservation constraints. This is a linear programming problem with efficient combinatorial solutions.

---

## Slide 04 -- Max-Flow Min-Cut Theorem

### s-t Cut

A partition of vertices into sets `S` and `T` where `s ∈ S` and `t ∈ T`. The capacity of the cut is:

```
c(S, T) = Σ c(u,v)  for all u ∈ S, v ∈ T
```

Only edges crossing from S to T count -- not edges from T to S.

### The theorem (Ford & Fulkerson, 1956)

In any flow network, the maximum flow from s to t equals the minimum capacity of any s-t cut:

```
max |f| = min c(S, T)
```

### Why it matters

- Provides a certificate of optimality -- if you find a flow and a cut with equal value, both are optimal
- Connects two seemingly different problems -- maximising flow and minimising cuts
- Foundation for duality in network optimisation

> The max-flow min-cut theorem is a special case of LP strong duality. Every max-flow algorithm implicitly finds a minimum cut as well.

---

## Slide 05 -- Residual Graphs & Augmenting Paths

### Residual graph

Given a flow `f`, the residual graph `G_f` shows remaining capacity on each edge:

```
Forward edge:  c_f(u,v) = c(u,v) - f(u,v)   (room to push more)
Backward edge: c_f(v,u) = f(u,v)              (room to cancel flow)
```

### Augmenting path

A path from s to t in the residual graph `G_f`. The bottleneck of the path is the minimum residual capacity along it.

### Augmenting Path Theorem

A flow `f` is maximum if and only if there is no augmenting path from s to t in `G_f`.

> Backward edges are the key insight -- they allow algorithms to "undo" earlier flow decisions. Without them, greedy approaches can get stuck at suboptimal solutions.

---

## Slide 06 -- Ford-Fulkerson Method

### The method (1956)

```
Ford-Fulkerson(G, s, t):
    initialise f(u,v) = 0 for all edges
    while there exists an augmenting path p in G_f:
        cf(p) = min residual capacity along p
        for each edge (u,v) in p:
            f(u,v) += cf(p)
            f(v,u) -= cf(p)
    return f
```

### Key properties

- **Correctness** -- terminates with max flow when capacities are integers
- **Integer capacities** -- runtime `O(E · |f*|)` where `|f*|` is max flow value
- **Irrational capacities** -- may not terminate; can converge to wrong answer
- **Path choice matters** -- poor choices lead to exponential iterations

### The bad example

With integer capacities, a poorly chosen path strategy can require `|f*|` augmentations. If the max flow is 2,000,000, that is 2,000,000 iterations -- each augmenting by 1.

> Ford-Fulkerson is a *method*, not an algorithm -- it does not specify how to find the augmenting path. The choice of path-finding strategy determines the actual complexity.

---

## Slide 07 -- Edmonds-Karp Algorithm

### BFS-based Ford-Fulkerson (1972)

Always choose the **shortest augmenting path** (fewest edges) using breadth-first search.

```
Edmonds-Karp(G, s, t):
    initialise f = 0
    while BFS finds a path p from s to t in G_f:
        cf(p) = bottleneck of p
        augment flow along p by cf(p)
    return f
```

### Complexity

| Metric | Bound |
|--------|-------|
| **Augmentations** | `O(VE)` -- at most VE augmenting path iterations |
| **Each BFS** | `O(E)` |
| **Total** | `O(V · E²)` |

### Why BFS helps

- Shortest paths in the residual graph are monotonically non-decreasing
- After at most `O(E)` augmentations, some shortest path distance increases
- At most `O(V)` distance increases possible -- hence `O(VE)` total augmentations

> Edmonds-Karp is the first polynomial-time max-flow algorithm. Independent of flow value, always `O(VE²)`.

---

## Slide 08 -- Dinic's Algorithm

### Level graph and blocking flow (1970)

Dinic's algorithm builds a *level graph* using BFS and finds *blocking flows* using DFS.

```
Dinic(G, s, t):
    while BFS builds level graph L of G_f:
        while there exists a blocking flow f' in L:
            augment f by f'
        rebuild level graph
    return f
```

### Complexity

| Metric | Bound |
|--------|-------|
| **Phases** | `O(V)` -- level graph depth increases each phase |
| **Blocking flow** | `O(VE)` per phase |
| **Total** | `O(V² · E)` |

### Special cases

- **Unit-capacity graphs:** `O(E · √V)` -- applies to bipartite matching
- **Unit networks:** `O(E · √V)` -- every vertex has in-degree or out-degree 1

> Dinic's algorithm is strictly faster than Edmonds-Karp. The improvement comes from finding multiple augmenting paths per BFS phase using blocking flows.

---

## Slide 09 -- Blocking Flows & Level Graphs

### Level graph construction

1. Run BFS from source `s` in `G_f`
2. Assign level `d(v)` = shortest distance from s
3. Keep only edges `(u,v)` where `d(v) = d(u) + 1`
4. Remove dead-end vertices with no path to `t`

### What is a blocking flow?

A flow in the level graph such that every s-t path contains at least one saturated edge. Not necessarily a maximum flow in the level graph, but simpler to find.

### Finding blocking flows

```
DFS from s:
    advance along unsaturated edges in level graph
    if reach t: augment along the path, retreat
    if stuck at v: delete v from level graph, retreat
```

- Each advance or retreat takes `O(1)` amortised
- At most `O(E)` advances and `O(V)` retreats per blocking flow

> The level graph acts as a "pruned" version of the residual graph, keeping only edges that make forward progress toward the sink.

---

## Slide 10 -- Push-Relabel Algorithm

### A different paradigm (Goldberg & Tarjan, 1988)

Instead of finding augmenting paths, push-relabel maintains a *preflow* -- conservation is relaxed, allowing excess at vertices.

### Core concepts

| Concept | Description |
|---------|------------|
| **Preflow** | Flow where inflow can exceed outflow (excess allowed) |
| **Excess e(v)** | `Σ f(u,v) - Σ f(v,w)` -- flow "pooling" at vertex v |
| **Height h(v)** | Label for each vertex; s starts at `|V|`, t at 0 |
| **Active vertex** | A vertex with `e(v) > 0` and `v ≠ s, t` |

### Operations

- **Push(u,v)** -- send `min(e(u), c_f(u,v))` flow from u to v; requires `h(u) = h(v) + 1`
- **Relabel(u)** -- increase `h(u)` to `min(h(v) + 1)` over neighbours v with `c_f(u,v) > 0`

```
Push-Relabel(G, s, t):
    initialise: h(s) = |V|, saturate all edges from s
    while there exists an active vertex u:
        if u has admissible edge (u,v):
            push(u, v)
        else:
            relabel(u)
```

---

## Slide 11 -- Push-Relabel -- Discharge & Heuristics

### Complexity

| Variant | Time |
|---------|------|
| **Generic push-relabel** | `O(V² · E)` |
| **FIFO selection** | `O(V³)` |
| **Highest-label** | `O(V² · √E)` |

### Gap heuristic

If no vertex has height `h`, then any vertex with height above `h` cannot reach the sink. Relabel all such vertices to `|V| + 1` -- they will drain back to source.

### Global relabelling

Periodically recompute exact heights using reverse BFS from `t`. Dramatically reduces wasted pushes in practice.

### Why push-relabel matters

- Asymptotically fastest known algorithms for max flow
- Excellent practical performance with heuristics
- Naturally parallel -- multiple active vertices can be processed simultaneously
- Foundation for minimum cut algorithms

> In practice, push-relabel with highest-label selection and gap/global relabelling heuristics is the fastest max-flow algorithm for dense graphs.

---

## Slide 12 -- Minimum Cost Flow

### Problem statement

Given a flow network where each edge has both a **capacity** and a **cost per unit of flow**, find a flow of a given value `d` that minimises total cost.

```
Minimise:   Σ a(u,v) · f(u,v)    over all edges
Subject to: flow conservation, capacity constraints, |f| = d
```

### Relationship to max flow

| Problem | Objective |
|---------|----------|
| **Max flow** | Maximise flow value, no costs |
| **Min-cost flow** | Fix flow value, minimise cost |
| **Min-cost max flow** | Maximise flow, break ties by minimum cost |

### Special cases

- **Shortest path** -- min-cost flow with `d = 1`
- **Assignment problem** -- min-cost perfect matching in bipartite graph
- **Transportation problem** -- supply and demand at nodes, costs on edges

> Min-cost flow is one of the most powerful network optimisation models. Many seemingly different problems reduce to it.

---

## Slide 13 -- Minimum Cost Flow -- Algorithms

### Successive shortest paths

Repeatedly find the shortest (cheapest) augmenting path and send flow along it. Uses Bellman-Ford or Dijkstra with potentials.

```
Successive-Shortest-Paths(G, s, t, d):
    f = 0
    while |f| < d:
        find shortest path p in G_f (by cost)
        augment along p
    return f
```

**Complexity:** `O(d · V · E)` with Bellman-Ford; `O(d · (E + V log V))` with Johnson's potentials + Dijkstra.

### Cycle-cancelling

Start with any feasible flow. Find negative-cost cycles in the residual graph and push flow around them until none remain.

**Complexity:** `O(V · E² · C · U)` -- pseudo-polynomial.

### Network simplex

Specialised simplex method for network flow LPs. Maintains a spanning tree basis. Strongly polynomial in practice and often the fastest method.

> For competitive programming: successive shortest paths with Dijkstra + potentials (SPFA/Johnson) is the standard min-cost flow template.

---

## Slide 14 -- Maximum Bipartite Matching

### The problem

Given a bipartite graph `G = (L ∪ R, E)`, find the largest set of edges with no shared endpoints.

### Reduction to max flow

1. Add source `s` connected to all vertices in `L` (capacity 1)
2. Add sink `t` connected from all vertices in `R` (capacity 1)
3. All original edges have capacity 1
4. Max flow in this network = maximum matching size

### Koenig's theorem

In a bipartite graph, the size of the maximum matching equals the size of the minimum vertex cover.

```
max matching = min vertex cover    (bipartite only)
```

### Hopcroft-Karp algorithm

Specialised for bipartite matching. Uses BFS to find maximal sets of shortest augmenting paths, then DFS to augment.

**Complexity:** `O(E · √V)` -- equivalent to Dinic's on unit networks.

> Any bipartite matching problem can be solved as max flow. For unweighted matching, Hopcroft-Karp is optimal. For weighted, use the Hungarian algorithm or min-cost flow.

---

## Slide 15 -- Hungarian Algorithm

### Weighted bipartite matching (Kuhn, 1955)

Find a perfect matching in a weighted bipartite graph that minimises (or maximises) total weight.

### Core idea

Maintain *feasible vertex labels* (potentials) such that:

```
l(x) + l(y) ≥ w(x, y)    for all x ∈ L, y ∈ R
```

The *equality subgraph* contains only edges where `l(x) + l(y) = w(x, y)`. A perfect matching in the equality subgraph is optimal.

### Algorithm outline

1. Initialise labels: `l(x) = max w(x,y)` for left vertices, `l(y) = 0` for right
2. For each unmatched left vertex, try to find an augmenting path in the equality subgraph
3. If no path exists, adjust labels to expand the equality subgraph
4. Repeat until all vertices matched

### Complexity

| Implementation | Time |
|---------------|------|
| **Naive** | `O(n⁴)` |
| **With shortest-path optimisation** | `O(n³)` |

> The Hungarian algorithm solves the assignment problem optimally. It can also be viewed as a primal-dual method for the assignment LP.

---

## Slide 16 -- Applications -- Transportation & Assignment

### Transportation problem

Ship goods from `m` suppliers to `n` consumers at minimum cost, respecting supply/demand constraints.

| Component | Network model |
|-----------|--------------|
| **Suppliers** | Source nodes with supply `s_i` |
| **Consumers** | Sink nodes with demand `d_j` |
| **Routes** | Edges with capacity and cost per unit |
| **Objective** | Min-cost flow satisfying all demand |

### Assignment problem

Assign `n` workers to `n` jobs, one-to-one, minimising total cost. Special case of transportation where all supplies and demands are 1.

### Airline scheduling

- Aircraft routing: flow through time-expanded flight network
- Crew scheduling: min-cost flow to cover all flights with minimum crews
- Gate assignment: matching flights to gates over time

> The simplex method was originally developed (Dantzig, 1947) partly to solve military transportation and logistics problems -- network flow was one of the first LP applications.

---

## Slide 17 -- Applications -- Image Segmentation & More

### Image segmentation via min-cut

Partition image pixels into foreground (F) and background (B):

1. Create a node per pixel
2. Source `s` = foreground label; sink `t` = background label
3. Edge `(s, pixel)` = likelihood pixel is foreground
4. Edge `(pixel, t)` = likelihood pixel is background
5. Edge `(pixel_i, pixel_j)` = smoothness penalty for adjacent pixels having different labels
6. Minimum s-t cut gives optimal binary segmentation

### Network reliability

Given a network with failure-prone edges, the maximum number of edge-disjoint paths between two nodes equals the minimum edge cut (Menger's theorem). Max flow computes this directly.

### Baseball elimination

Determine if a team is mathematically eliminated from winning the league. Formulated as a max-flow problem by Schwartz (1966).

### Project selection

Choose a subset of projects to maximise profit, respecting prerequisite dependencies. Equivalent to min-cut in a suitably constructed network.

> The power of network flow lies in modelling: once a problem is cast as a flow network, efficient algorithms immediately apply.

---

## Slide 18 -- Network Flow Modelling Techniques

### Vertex capacities

To limit flow through a vertex `v`, split it into `v_in` and `v_out` with an edge of the desired capacity:

```
Original:  u → v → w
Split:     u → v_in → (cap) → v_out → w
```

### Multiple sources/sinks

Add a super-source `S` connected to all sources and a super-sink `T` connected from all sinks. Edge capacities encode supply/demand.

### Lower bounds on flow

If edge `(u,v)` must carry at least `l(u,v)` flow:

1. Send `l(u,v)` flow on every lower-bounded edge
2. Adjust supply/demand to compensate
3. Solve the resulting standard flow problem

### Undirected edges

Replace each undirected edge with two directed edges (one in each direction), each with the same capacity.

### Time-expanded networks

Replicate the graph for each time step. Edges connect nodes across time to model temporal constraints (scheduling, routing over time).

> Mastering these transformations is the key skill for network flow in practice and competitions. The algorithm is the easy part -- the modelling is the hard part.

---

## Slide 19 -- Complexity Comparison

### Max-flow algorithms

| Algorithm | Time complexity | Notes |
|-----------|----------------|-------|
| **Ford-Fulkerson** | `O(E · |f*|)` | Pseudo-polynomial; depends on flow value |
| **Edmonds-Karp** | `O(V · E²)` | BFS shortest path; first polynomial bound |
| **Dinic's** | `O(V² · E)` | Blocking flows; `O(E√V)` on unit graphs |
| **Push-Relabel** | `O(V² · E)` | Generic; `O(V²√E)` with highest-label |
| **King-Rao-Tarjan** | `O(VE log(V²/E))` | Theoretical; rarely implemented |
| **Orlin** | `O(VE)` | Optimal for dense graphs (2013) |

### Related problems

| Problem | Best known |
|---------|-----------|
| **Min-cost flow** | `O(V · E · log V · log(VC))` network simplex |
| **Bipartite matching** | `O(E · √V)` Hopcroft-Karp |
| **Weighted matching** | `O(n³)` Hungarian |
| **General matching** | `O(V · E)` Edmonds' blossom |

> In practice, push-relabel with heuristics dominates for large dense graphs. Dinic's is preferred for sparse graphs and competitive programming.

---

## Slide 20 -- Summary & Further Reading

### Key takeaways

- Flow networks model constrained movement through a graph: capacity, conservation, and source-to-sink flow
- The max-flow min-cut theorem connects flow maximisation to cut minimisation via LP duality
- Ford-Fulkerson is the foundational method; Edmonds-Karp, Dinic's, and push-relabel improve it to polynomial time
- Dinic's algorithm achieves `O(V²E)` and is optimal for unit-capacity graphs at `O(E√V)`
- Push-relabel with heuristics is the fastest in practice for dense graphs
- Minimum cost flow generalises max flow with edge costs -- solved via successive shortest paths or network simplex
- Bipartite matching reduces to max flow; the Hungarian algorithm solves weighted assignment in `O(n³)`
- Applications span transportation, scheduling, image segmentation, network reliability, and project selection
- Modelling techniques (vertex splitting, super-sources, lower bounds) are the key practical skill

### Recommended reading

| Source | Description |
|--------|------------|
| **Cormen et al.** | *Introduction to Algorithms* (CLRS), Chapters 24-26 -- max flow and matching |
| **Ahuja, Magnanti, Orlin** | *Network Flows: Theory, Algorithms, and Applications* -- the definitive reference |
| **Kleinberg & Tardos** | *Algorithm Design*, Chapters 7-8 -- network flow and applications |
| **Schrijver** | *Combinatorial Optimization* -- rigorous treatment of flow, matching, and polyhedral theory |
| **Jeff Erickson** | [Algorithms](https://jeffe.cs.illinois.edu/teaching/algorithms/) -- free textbook with excellent flow chapters |
| **CP-Algorithms** | [cp-algorithms.com/graph/flow](https://cp-algorithms.com/graph/) -- implementation-focused flow tutorials |
