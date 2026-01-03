# SAC_Spark

*A distributed system to compute minimal road distances between cities using Apache Spark RDDs via PySpark.*

Assignment for **Sistemes Actuals de Computació**.

---

## INTRODUCTION

This repository contains the implementation of a distributed algorithm to compute shortest paths in a weighted directed graph, serving as a practical activity for processing Resilient Distributed Datasets (RDDs) with *Apache Spark*.

The project implements an iterative Dijkstra-style relaxation algorithm to find the minimal distance paths from a source node (city) to all others. This practice explores the *Map-Reduce* paradigm and distributed graph processing using Apache Spark RDDs, without requiring a local installation of the cluster, instead relying on cloud-based notebooks (Google Colab in this case).

---

## SYSTEM STRUCTURE

The system is divided into three logical layers:

### 1. Data Modeling and Preprocessing

The application begins by generating graph structures using the `NetworkX` library. It supports both manually defined graphs (i.e.: the example graph) and randomly generated graphs using the $G(n, p)$ model via the implemented `generate_random_graph` function.

To make these graphs compatible with Spark's distributed computing model, they are transformed into a specific tuple-based RDD structure:

- **Node Representation:** `(Name, State)`
- **Node State:** `(Neighbors List, Weight, Visited/Processed, Path History)`

Where:
- `Neighbors_List`: A list of tuples `(Destination, Weight)` representing outgoing connections.
- `Weight`: The cost to reach the source node from the current node.
- `Visited/Processed`: A boolean tracking if the node has been already processed.
- `Path History`: A list accumulating the nodes visited to reach the current node (i.e.: the best known path from the source to the node).

### 2. Distributed Dijkstra Algorithm

The core logic is an iterative implementation of **Dijkstra's algorithm**, adapted for RDDs. Unlike the standard priority-queue implementation, this approach makes use of Spark's parallel transformations (`map`, `filter`, `reduce`).

The `solve_dijkstra` function executes the following cycle until all reachable nodes are visited:

1. **Selection (Reduce):**
    - The system filters (`filter`) for all unvisited nodes.
    - A `reduce` operation aggregates these nodes to identify the one with the minimum  weight. This node becomes the *current node* for the iteration.

2. **Relaxation (Map):**
    - A `map` transformation is applied to the entire graph RDD using a custom logic function (`update_node_logic`).
    - The function marks the *current node* as visited.
    - For every neighbor of the *current node*, it calculates a distance: `current_weight + edge_weight`.
    - If that distance is strictly smaller (better) than the neighbor's existing distance, the neighbor's state is updated with the new distance and the path history is appended, since it is now the best path to that node.

3. **Optimization (Caching):**
    - At the end of each iteration, `nodes.cache()` is called to materialize the RDD. This is critical for breaking the RDD history, preventing stack overflow errors and performance degradation inherent in long iterative Spark jobs.

### 3. Visualization and Validation

The application includes a final visualization using `Matplotlib`. It provides side-by-side comparisons of:
- The raw graph topology.
- The expected solution (computed via `NetworkX`'s standard Dijkstra).
- The distributed Spark solution.

This visual output validates the correctness of the distributed algorithm by highlighting the path edges and displaying computed costs.

---

## USAGE

The project was explicitly designed to be run on *Google Colab*. This cloud-based setup eliminates the need for complex local installations of the Apache Spark ecosystem (JVM, Hadoop binaries, Spark workers, etc.).

**How to Run:**
1.  Open the `SAC_Spark.ipynb` notebook in Google Colab.
2.  The notebook handles the environment setup automatically, and already has a JDK, Spark, Hadoop and the `pyspark` library installed.
3.  Execute the cells sequentially to observe the step-by-step processing of the graph.

---

## GENERATION AND VISUALIZATION ADDITIONS

To make the theoretical concepts more accessible and pleasant to work with, several extra features were implemented:

**Automatic Graph Generation:**
Instead of relying solely on the example precoded graph, a `generate_random_graph` function (based on the suggested resources) was implemented to create arbitrary directed graphs. This allows testing the algorithm against various topologies by simply adjusting parameters like node count and edge probability.

**Data Visualization:**
Extensive use of `Matplotlib` and `NetworkX` was made to visualize the results graphically. Rather than just staring at raw RDD output logs, the project plots the graphs, highlighting the computed shortest paths (in blue) and comparing them against standard NetworkX Dijkstra implementations (in red) to visually verify correctness.

---

## CONCLUSIONS

This project demonstrates the advantages of distributed computing over traditional sequential methods for large-scale graph processing. By utilizing Apache Spark RDDs, the system moves beyond the physical limitations of a single machine, enabling parallel execution across a cluster. This approach allows for the efficient processing of massive datasets that would overload non-distributed environments, effectively abstracting data partitioning and processing while maximizing throughput.

The primary value of this implementation lies in its scalability. While standard Dijkstra algorithms run in memory on a single machine, this RDD-based approach can theoretically scale to massive graphs that exceed the memory capacity of a single server, distributing the storage and computation across a cluster of workers.

A significant technical challenge encountered during development was the management of the Spark Caching and Lineage. Since Dijkstra is an iterative algorithm, each iteration creates a new RDD based on the previous one, causing the RDD lineage graph to grow indefinitely. Without intervention, Spark tries to recompute the entire chain of transformations from the root RDD in every single iteration, leading to exponential performance degradation and eventual stack overflow errors. Properly applying `.cache()` to cut this lineage and persist intermediate results was a crucial step, showing that in distributed systems, managing how data is stored is just as important as how it is computed.

---

## AUTHOR

**Adrià Serra Martín**