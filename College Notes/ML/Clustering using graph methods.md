Explain clustering with minimal spanning tree with reference to Graph based clustering

# Clustering Using a Minimum Spanning Tree

## 1. Graph-based clustering

Graph-based clustering represents a dataset as a graph and identifies clusters using the connections between data points.

In a distance-based representation:

- Each data point becomes a vertex or node.
- An edge connects a pair of points.
- The edge weight represents the distance or dissimilarity between the points.

A small edge weight means the points are close, while a large weight means they are far apart. In the basic formulation, we construct a complete, undirected, weighted graph, so every pair of points is connected.[1]

The central idea is:

> Points within a cluster should be connected through relatively short edges, while different clusters should be separated by relatively long edges.

A **minimum spanning tree** provides a compact way to capture these connections.[2]

## 2. What is a minimum spanning tree?

A spanning tree of a connected, undirected graph is a subgraph that:

- Includes every vertex.
- Connects all vertices.
- Contains no cycles.
- Has exactly $$n-1$$ edges for $$n$$ vertices.

A minimum spanning tree, or MST, is a spanning tree whose total edge weight is as small as possible.[1][2]

Mathematically:

$$
T^*=\underset{T\text{ is a spanning tree of }G}{\arg\min}
\sum_{e\in T}w(e)
$$

where $$w(e)$$ is the weight of edge $$e$$.

An MST minimizes the total weight of the tree; it does not necessarily provide the shortest path between every pair of vertices. Its purpose here is to preserve an economical network of connections between all observations.

## 3. How is an MST used for clustering?

After constructing an MST, remove selected long edges. Each remaining connected component becomes a cluster.

For the standard single-linkage version, to obtain $$k$$ clusters:

$$
\boxed{\text{Remove the }k-1\text{ longest edges from the MST}}
$$

Removing one edge from a tree splits one connected component into two. Therefore, removing $$k-1$$ edges produces exactly $$k$$ connected components.[2]

| Required clusters | MST edges to remove |
|---|---:|
| 2 | 1 longest edge |
| 3 | 2 longest edges |
| 4 | 3 longest edges |
| $$k$$ | $$k-1$$ longest edges |

Alternatively, remove every MST edge whose weight exceeds a chosen threshold. In that case, the number of clusters emerges from the threshold rather than being specified directly.[1]

## 4. Algorithm

### Input

- Dataset containing $$n$$ points.
- A distance measure.
- Desired number of clusters $$k$$, or an edge-weight threshold.

### Procedure

1. Represent each data point as a vertex.
2. Calculate pairwise distances and use them as edge weights.
3. Construct an MST using an algorithm such as Kruskal’s or Prim’s.
4. Sort the MST edges in decreasing order of weight.
5. Remove the $$k-1$$ longest edges.
6. Identify the remaining connected components.
7. Assign each connected component a separate cluster label.[2][3][4]

### Pseudocode

```text
MST_Clustering(data, k):

    Create a weighted graph G from the data
    T = MinimumSpanningTree(G)

    Sort the edges of T by decreasing weight
    Remove the first k - 1 edges

    Return the connected components of T as clusters
```

## 5. Worked example

Consider six points on a number line:

| Point | Position |
|---|---:|
| A | 1 |
| B | 2 |
| C | 3 |
| D | 10 |
| E | 11 |
| F | 12 |

For one-dimensional data, use absolute distance:

$$
d(p,q)=|x_p-x_q|
$$

For example:

$$
d(A,B)=|1-2|=1
$$

$$
d(C,D)=|3-10|=7
$$

### Step 1: Construct the weighted graph

The pairwise distance matrix is:

| Point | A | B | C | D | E | F |
|---|---:|---:|---:|---:|---:|---:|
| A | 0 | 1 | 2 | 9 | 10 | 11 |
| B | 1 | 0 | 1 | 8 | 9 | 10 |
| C | 2 | 1 | 0 | 7 | 8 | 9 |
| D | 9 | 8 | 7 | 0 | 1 | 2 |
| E | 10 | 9 | 8 | 1 | 0 | 1 |
| F | 11 | 10 | 9 | 2 | 1 | 0 |

Each off-diagonal value gives the edge weight between two points.

### Step 2: Construct the MST

The MST is:

```text
A --1-- B --1-- C -------7------- D --1-- E --1-- F
```

Its edges are:

| Edge | Weight |
|---|---:|
| A–B | 1 |
| B–C | 1 |
| C–D | 7 |
| D–E | 1 |
| E–F | 1 |

There are six vertices and five edges:

$$
n-1=6-1=5
$$

The total MST weight is:

$$
1+1+7+1+1=11
$$

Even though C and D are relatively far apart, the MST must connect them because it must connect all six vertices.

### Step 3: Remove the longest edge

Suppose we want two clusters:

$$
k=2
$$

The number of edges to remove is:

$$
k-1=1
$$

The longest edge is C–D, with weight 7. Remove it:

```text
A --1-- B --1-- C          D --1-- E --1-- F

     Cluster 1                  Cluster 2
```

### Step 4: Identify the clusters

$$
C_1=\{A,B,C\}
$$

$$
C_2=\{D,E,F\}
$$

| Point | Cluster |
|---|---|
| A | 1 |
| B | 1 |
| C | 1 |
| D | 2 |
| E | 2 |
| F | 2 |

The large gap between positions 3 and 10 naturally separates the two groups.

### Threshold-based alternative

Instead of specifying $$k=2$$, choose a threshold of 3 and remove all MST edges with weight greater than 3.

Only C–D has weight above this threshold, so the same two clusters result.

If several candidate edges have equal weights, requesting a fixed number of clusters can require a tie-breaking rule; equally valid cuts may produce different partitions.[2]

## 6. How Kruskal’s algorithm helps

Kruskal’s algorithm builds the MST by considering edges in increasing order of weight and adding an edge only when it does not create a cycle.

For clustering, it can stop early:

1. Start with every vertex as a separate cluster.
2. Process edges from smallest to largest.
3. Join two components when an edge connects them.
4. Stop when exactly $$k$$ components remain.

This is equivalent to building the full MST and then removing its $$k-1$$ longest edges, with consistent handling of ties.[3]

For the example:

| Action | Number of components |
|---|---:|
| Start: $$\{A\},\{B\},\{C\},\{D\},\{E\},\{F\}$$ | 6 |
| Add A–B | 5 |
| Add B–C | 4 |
| Add D–E | 3 |
| Add E–F | 2 |
| Stop before adding C–D | 2 |

## 7. Relationship with hierarchical clustering

MST clustering is closely related to **single linkage** hierarchical clustering.

In single linkage, the distance between two clusters is the smallest distance between any point in one cluster and any point in the other:

$$
d(C_i,C_j)=
\min_{x\in C_i,\;y\in C_j}d(x,y)
$$

Single linkage repeatedly merges the closest clusters. The MST represents the connections needed to recover this hierarchy.[2][5]

There are two equivalent viewpoints:

| Viewpoint | Operation |
|---|---|
| Agglomerative, or bottom-up | Add short edges to merge components |
| Divisive, or top-down | Remove long MST edges to split components |

Thus, repeatedly cutting the longest remaining MST edges gives progressively finer single-linkage partitions.[2][3]

## 8. Advantages

- Compact representation: after construction, the MST stores only $$n-1$$ edges.
- Non-spherical clusters: clusters are based on connectivity rather than distance from a centroid.
- Hierarchical structure: different cuts produce different levels of grouping.
- Flexible stopping rule: choose a required number of clusters or an edge-weight threshold.
- Clear interpretation: removed long edges indicate the connections used to separate groups.[1][2]

## 9. Limitations

### A. Chaining effect

A sequence of short connections can join two otherwise distinct groups. This is a characteristic weakness of single-linkage clustering: points need not all be close to one another if a chain connects them.[6]

### B. Sensitivity to outliers

An isolated point may be connected by a very long edge. Removing the longest edge can therefore create a singleton cluster rather than separate the main groups. Basic MST cutting does not automatically label such points as noise; additional rules are needed.[1]

### C. Choosing the cut

The MST alone does not determine which clustering is most useful. A cluster count, distance threshold, or more advanced edge-removal criterion is still required.[1][2]

### D. Cost of graph construction

Although the MST has only $$n-1$$ edges, explicitly constructing a complete graph requires:

$$
|E|=\frac{n(n-1)}{2}
$$

Therefore, storing all pairwise edges requires $$O(n^2)$$ space. Constructing the MST can be the expensive part; specialized methods can avoid explicitly building the full graph in some settings.[7]

