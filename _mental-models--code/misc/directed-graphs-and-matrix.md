# Representing a directed graph with a matrix

## 1. Adjacency Matrix Representation

An adjacency matrix is a common way to represent graphs, particularly when you have a dense graph (i.e., many edges). Each cell in the matrix represents the presence (or weight) of an edge between two nodes.

```ts
const graph: number[][] = [
  [0, 1, 0],
  [0, 0, 1],
  [1, 0, 0]
];
// graph[0][1] = 1 represents an edge from node 0 to node 1
```

**Pros**

- Easy to check if an edge exists between two nodes with constant time complexity, `O(1)`, for both edge existence and weight lookup
- Good for dense graphs

**Cons**

- Space complexity is `O(N^2)` where `N` is the number of nodes. For sparse graphs, using an adjacency matrix wastes a lot of space.
- Depending on the size of your graph, the matrix representation can be hard to read and visualize.

## 2. Adjacency List Representation

If the graph is sparse, an adjacency list might be a better choice. This is typically implemented as an array of lists, where each list contains the nodes that are directly reachable from the node corresponding to that index.

```ts
const graph: { [key: number]: number[] } = {
  0: [1], // Node 0 has an edge to Node 1
  1: [2], // Node 1 has an edge to Node 2
  2: [0] // Node 2 has an edge to Node 0
};
```

**Pros**

- Space efficiency, especially for sparse graphs
- Traversing neighbors of a node is simpler and more intuitive (readable) using an adjacency list

**Cons**

- Less efficient for dense graphs
- Checking the existence of a specific edge takes linear time relative to the number of edges originating from that node
