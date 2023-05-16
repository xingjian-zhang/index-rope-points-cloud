# Algorithm for Indexing Points Cloud of a Rope
- Author: Xingjian Zhang (jimmyzxj@umich.edu)

**Summary**
- Build a discrete graph from the geometric data.
- Simulate a heat diffusion on the graph from one end (src, high temperature)
to another end (dst, low temperature).
- The indexing are obtained by sorting the
"temperature" of each node.

Suppose we have $n$ points in total.

## Step 1: Graph Construction
1. Each point is a node in the graph.
2. The edge of two nodes $x,y$ is given by a function $\rho: \mathbb{R}^2\to
   [0, 1]$
   $$ \rho(x, y) = \exp\left(-\dfrac{\| x - y \|^2}{2\sigma^2}\right) $$
3. $\sigma$ is a parameter defined by
   $$ \sigma = \dfrac{1}{n}\sum_{i=1,\dots, n} \min_{j\neq i} \|x_i - x_j\| $$
   (Intuition: $\sigma$ is approximately the average distance between two adjacent points.)
4. Given step 2 and 3, we can construct the adjacency matrix $A$ of the graph,
   which should be a symmetric and sparse matrix (plotted below). Notice that
   the diagonal entries of $A$ are defined to be 0.

## Step 2: Heat Diffusion
1. We assume that the the beginning and ending point of the rope is given by
   $src$ and $dst$. (This is not necessary but it makes the algorithm easier to
   implement. We will show how to find $src$ and $dst$ in the next section.)
2. Compute the *diffusion matrix* (normalized adjacency matrix) $W$ by
   $$ W = D^{-1}A $$
   where $D$ is the diagonal matrix with $D_{ii} = \sum_j A_{ij}$.
3. Define the temperature distribution as a vector $f^t\in \mathbb{R}^n$ with respect to
   step/time $t$. The boundary condition is given by $f^t_{src} = 1$ and
   $f^t_{dst} = 0$.
4. The heat diffusion is given by
   $$ f^{t+1} = Wf^t $$
   and then set
   $$ f^{t+1}_{src} = 1, f^{t+1}_{dst} = 0 $$
5. Repeat step 4 until convergence, for example, $\|f^{t+1} - f^t\| < \epsilon$.

## Step 3: Indexing
1. Sort the nodes by the temperature $f^t$ in descending order.
2. The index of the nodes are given by the sorted order.

## What if we do not know the beginning and ending point?
We can find the beginning and ending point by performing two heat diffusion
simulations with a few steps. (not implemented yet)

1. Select a random point $i$ as the beginning point and perform the heat
   diffusion simulation for $k$ steps. (set $f^t_i = 1$ as the boundary
   condition) $k$ is a hyperparameter and can be set as $\sqrt{n}$ for
   simplicity.
2. Find the node $j$ with the lowest temperature and set it as the new beginning
   point $src$. i.e. Set $f^t_{src} = 1$ as the new boundary condition and
   perform another heat diffusion simulation for $k$ steps.
3. Find the node $j'$ with the lowest temperature and set it as the ending
   point $dst$.