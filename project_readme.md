# Comparative Analysis of Bidirectional BFS and Bidirectional Dijkstra

## Shortest-Path Search on Large-Scale Social Networks (Stanford SNAP Facebook Dataset)

### Overview
This project presents an empirical and theoretical performance evaluation of **Bidirectional Breadth-First Search (BFS)** versus **Bidirectional Dijkstra's Algorithm** for state-space shortest-path discovery on large-scale social graphs. Using the **Stanford SNAP Facebook Network**, the notebook benchmarks both search paradigms across latency, state-space exploration, memory allocation, and path optimality.

---

### Dataset
* **Source:** [Stanford SNAP Social Circles: Facebook](https://snap.stanford.edu/data/egonets-Facebook.html) (`facebook_combined.txt.gz`)
* **Graph Type:** Undirected, unweighted (uniform edge weight $w = 1$)
* **Nodes:** 4,039
* **Edges:** 88,234
* **Degree Statistics:**
  * Average Degree: `43.69`
  * Minimum Degree: `1`
  * Maximum Degree: `1045`

---

### Algorithms Evaluated
1. **Bidirectional Breadth-First Search (BFS):**
   * Uses dual FIFO queues (`collections.deque`) expanding alternating wavefronts from the source and target nodes.
   * Terminates immediately upon the first frontier collision (`meeting_node`).
2. **Bidirectional Dijkstra's Algorithm:**
   * Uses dual min-priority heaps (`heapq`) tracking shortest-path estimates from both frontiers.
   * Terminates when the sum of the minimum frontier keys meets or exceeds the best path cost discovered so far ($\mu_{f} + \mu_{b} \ge d_{\text{best}}$), guaranteeing 100% path optimality.

---

### Evaluation Metrics & Profiling
Both algorithms are instrumented to record:
* **Execution Time (Latency):** Measured in seconds using `time.perf_counter()`.
* **Search Space Expansion:** Total number of popped/expanded nodes across both frontiers.
* **Peak Memory Consumption:** Measured in Kilobytes (KB) using Python's `tracemalloc` module.
* **Solution Depth & Optimality:** Path length comparison and agreement rate across randomized queries.

---

### Key Benchmark Results (50 Random Queries)
Across 50 randomly sampled source-target node pairs (`random.seed(42)`):

| Metric | Bidirectional BFS | Bidirectional Dijkstra | Comparison / Ratio |
| :--- | :---: | :---: | :---: |
| **Average Execution Time** | `0.000503 s` | `0.005635 s` | **11.20x faster** (BFS) |
| **Average Nodes Expanded** | `17.48` | `209.88` | **12.01x fewer nodes** (BFS) |
| **Average Peak Memory** | `52.95 KB` | `191.58 KB` | **3.62x less memory** (BFS) |
| **Average Path Distance** | `3.68` | `3.62` | **94.0% agreement** (47/50) |

#### Key Takeaway on Optimality
While Bidirectional BFS achieves an order-of-magnitude speedup and significantly lower memory overhead, terminating on the *first* neighbor intersection without exhausting the current depth level can occasionally yield a suboptimal path of length $d + 1$ instead of $d$ (observed in 3 out of 50 queries). Bidirectional Dijkstra avoids this via its stopping condition, guaranteeing true shortest paths at the cost of priority-queue overhead and wider frontier expansion.

---

### Visualizations Included
* **Degree Distribution Histogram:** Node connectivity distribution across the Facebook graph.
* **Sample Subgraph Topology:** Spring-layout visualization of the first 100 nodes.
* **Single-Query Bar Charts:** Side-by-side comparison of latency, node expansions, and peak memory.
* **State-Space Wavefront Visualization:** Graph illustration highlighting Source, Target, Meeting Node, and reconstructed optimal path edges.
* **50-Query Benchmark Curves & Aggregate Summaries:** Line plots and bar charts comparing execution time, nodes expanded, and peak heap memory across all 50 experiments.

---

### Requirements & Setup

#### Dependencies
Ensure Python 3.7+ is installed along with the following libraries:
```bash
pip install numpy pandas matplotlib networkx jupyter
```

#### Running the Notebook
1. Clone or download the repository containing the Jupyter Notebook (`.ipynb`).
2. Launch Jupyter Notebook or Google Colab:
   ```bash
   jupyter notebook
   ```
3. Run all cells sequentially. The notebook automatically downloads and extracts `facebook_combined.txt.gz` from Stanford SNAP if it is not already present in the working directory.

---

### Output Artifacts
Running the notebook generates two CSV files for reproducibility and further analysis:
* `algorithm_comparison_results.csv`: Detailed per-query metrics across all 50 random source-target pairs.
* `algorithm_summary.csv`: Summary table from the illustrative single-query run (`Node 0` $\rightarrow$ `Node 100`).