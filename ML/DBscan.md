Write detailed  on following: DBSCAN algorithm

# DBSCAN Algorithm

## 1. Introduction

**DBSCAN** stands for Density-Based Spatial Clustering of Applications with Noise. It is an unsupervised machine-learning algorithm that groups closely packed data points into clusters and identifies points in sparse regions as noise or outliers. Unlike K-means, it does not require the number of clusters to be specified in advance and can discover clusters of irregular shapes.[1]

Its basic idea is:

> A cluster is a dense region of data points separated from other clusters by regions of lower density.

DBSCAN starts from a sufficiently dense neighbourhood and expands the cluster through nearby dense neighbourhoods.[1]

## 2. Important parameters

DBSCAN primarily depends on two parameters: epsilon and MinPts.[1]

### A. Epsilon $$\varepsilon$$

Epsilon defines the radius of the neighbourhood around a point. Two points are considered neighbours if their distance is less than or equal to $$\varepsilon$$.[1]

The epsilon-neighbourhood of a point $$p$$ is:

$$
N_{\varepsilon}(p)=\{q\in D\mid d(p,q)\leq\varepsilon\}
$$

where:

- $$D$$ is the dataset.
- $$d(p,q)$$ is the distance between points $$p$$ and $$q$$.
- $$N_{\varepsilon}(p)$$ contains all points within the specified radius, including $$p$$ itself.

For numerical data, Euclidean distance is commonly used. For two-dimensional points:

$$
d(p,q)=\sqrt{(x_p-x_q)^2+(y_p-y_q)^2}
$$

Other distance metrics can also be used.[1]

### B. MinPts

MinPts is the minimum number of points that must lie inside a point’s epsilon-neighbourhood for that point to be considered a core point. In scikit-learn, this parameter is called `min_samples`, and the count includes the point itself.[1]

For example:

$$
\varepsilon=2,\qquad \text{MinPts}=4
$$

A point is a core point if its neighbourhood contains at least four points in total: itself and at least three others.

MinPts is a local density requirement, not simply a rule about the total number of points in a finished cluster.[1]

## 3. Types of points

DBSCAN distinguishes three types of points: core, border, and noise.[2]

### A. Core point

A point $$p$$ is a core point if its neighbourhood contains at least MinPts points:

$$
|N_{\varepsilon}(p)|\geq\text{MinPts}
$$

Core points lie in sufficiently dense regions and can expand a cluster.[1]

### B. Border point

A point is a border point if:

- It has fewer than MinPts points in its own neighbourhood.
- It lies within the epsilon-neighbourhood of a core point.

A border point belongs to a cluster, but it does not expand the cluster further.[2]

### C. Noise point

A noise point is neither a core point nor a border point. It does not belong to any discovered cluster. In scikit-learn, noise points receive the label `-1`.[2][3]

| Point type | Enough points in its own neighbourhood? | Within $$\varepsilon$$ of a core point? | Role |
|---|---|---|---|
| Core | Yes | Not an additional requirement | Starts or expands a cluster |
| Border | No | Yes | Joins a cluster without expanding it |
| Noise | No | No | Remains outside clusters |

These categories depend on the chosen values of $$\varepsilon$$, MinPts, and the distance metric.[1][2]

## 4. How DBSCAN works

The algorithm proceeds as follows:[2]

1. Select an unvisited point $$p$$.
2. Mark it as visited and find its epsilon-neighbourhood.
3. If it has fewer than MinPts neighbours, temporarily mark it as noise.
4. Otherwise, $$p$$ is a core point. Create a new cluster and add $$p$$ and its neighbours.
5. Examine the neighbours:
   - If a neighbour is a core point, add its neighbours to the search.
   - If it is a border point, include it but do not expand through it.
6. Continue until no more points can be added.
7. Repeat with another unvisited point.
8. Points that remain outside all clusters are noise.

A point initially marked as noise can later become a border point if it is reached from a core point. Therefore, the first noise label is provisional.[2]

### Simplified pseudocode

```text
Initially, all points are unvisited and unassigned.

For each point p:

    If p is already visited:
        continue

    Mark p as visited
    Find neighbours of p within epsilon

    If neighbourhood size < MinPts:
        Mark p as noise for now
        continue

    Create a new cluster C
    Add p to C
    Put p's neighbours into a search list

    While the search list is not empty:
        Remove a point q from the list

        If q is unvisited:
            Mark q as visited
            Find q's neighbours

            If q is a core point:
                Add its neighbours to the search list

        If q is not already assigned to a cluster:
            Assign q to C
```

This expansion process is what allows DBSCAN to discover shapes that are not spherical.[1][2]

## 5. Worked example

Consider the following illustrative one-dimensional dataset. Using one dimension makes the distance calculations easy to follow.

| Point | Position |
|---|---:|
| A | 0 |
| B | 1 |
| C | 2 |
| D | 3 |
| E | 4 |
| F | 10 |
| G | 11 |
| H | 12 |
| I | 13 |
| J | 14 |
| K | 20 |

Choose:

$$
\varepsilon=1.1,\qquad\text{MinPts}=3
$$

In one dimension, the distance is simply:

$$
d(p,q)=|x_p-x_q|
$$

### Step 1: Find each neighbourhood

Remember to include the point itself.

| Point | Neighbours within $$1.1$$ | Count | Final type |
|---|---|---:|---|
| A | A, B | 2 | Border |
| B | A, B, C | 3 | Core |
| C | B, C, D | 3 | Core |
| D | C, D, E | 3 | Core |
| E | D, E | 2 | Border |
| F | F, G | 2 | Border |
| G | F, G, H | 3 | Core |
| H | G, H, I | 3 | Core |
| I | H, I, J | 3 | Core |
| J | I, J | 2 | Border |
| K | K | 1 | Noise |

For example:

$$
d(B,C)=|1-2|=1\leq1.1
$$

Therefore, B and C are neighbours.

But:

$$
d(B,D)=|1-3|=2>1.1
$$

Therefore, B and D are not direct neighbours.

### Step 2: Form the first cluster

Suppose A is examined first:

- A has only two points in its neighbourhood, so it is initially marked as noise.
- B has three points in its neighbourhood, so it starts a cluster.
- B adds A and C.
- C is a core point, so expansion reaches D.
- D is also a core point, so expansion reaches E.
- A and E belong to the cluster as border points.

Thus:

$$
C_1=\{A,B,C,D,E\}
$$

A’s initial noise label is replaced when B reaches it.

### Step 3: Form the second cluster

The same process around G, H, and I produces:

$$
C_2=\{F,G,H,I,J\}
$$

F and J are border points.

### Step 4: Identify noise

K has no nearby core point and remains noise:

$$
\text{Noise}=\{K\}
$$

### Final result

| Group | Points |
|---|---|
| Cluster 1 | A, B, C, D, E |
| Cluster 2 | F, G, H, I, J |
| Noise | K |

Notice that A and E are four units apart—greater than $$\varepsilon=1.1$$—but still belong to the same cluster through a chain of core points. Epsilon limits local neighbourhood distance, not the maximum distance between all points in a cluster.[1]

## 6. Effect of parameter selection

| Parameter choice | Possible effect |
|---|---|
| Epsilon too small | Clusters may fragment, and many points may become noise |
| Epsilon too large | Separate groups may merge into one cluster |
| MinPts too small | Small, sparse groups may qualify as clusters |
| MinPts too large | Only very dense regions qualify; smaller groups may disappear |

These effects interact, so changing a parameter does not guarantee a particular number of clusters.[1][2]

A common heuristic for choosing epsilon is a sorted k-distance plot: examine the distance to the neighbour corresponding to the chosen MinPts convention and look for a noticeable bend or “elbow.” This is a starting point rather than a universally reliable rule.[2]

## 7. Advantages

- No predefined cluster count: the number of clusters emerges from the data and parameters.
- Irregular cluster shapes: connected dense regions need not be circular or spherical.
- Explicit noise detection: isolated points can remain unassigned.
- Density-based grouping: clusters grow through local neighbourhoods rather than around a centroid.[1][4]

## 8. Limitations

- Parameter sensitivity: results depend strongly on epsilon and MinPts.
- Varying densities: one global parameter setting may not suit both a dense cluster and a sparse cluster.
- High-dimensional data: distance-based neighbourhoods can become less informative.
- Computational cost: neighbourhood searches can become expensive on large datasets.
- Memory usage: scikit-learn’s implementation can require $$O(n^2)$$ memory in the worst case, particularly when neighbourhoods are very large; this is not a universal memory requirement of the original algorithm.

