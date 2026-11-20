# Heuristic Sudoku Solver: VNS + Simulated Annealing

An optimization engine developed for **SUTD Heuristics and Systems Theory (40.018)** that solves generalized $N \times N$ Sudoku grids (from $9 \times 9$ up to $49 \times 49$) by combining **Simulated Annealing (SA)**, **Variable Neighborhood Search (VNS)**, and **candidate-based constructive initialization**, benchmarked against an exact **Gurobi Integer Linear Programming (ILP)** model.

---

## 📐 Mathematical Formulation

For a generalized $N \times N$ Sudoku grid where $N = n^2$, let $F \subset \{1, \dots, N\}^2$ denote the set of fixed clue cells.

### 1. Subgrid-Invariant Search Space
Rather than penalizing all three Sudoku rules in the objective, every $n \times n$ subgrid (box) is initialized as a valid permutation of $\{1, \dots, N\}$ that respects all clues in $F$. Because every neighborhood move only swaps non-fixed cells **within the same $n \times n$ subgrid**, box constraints remain strictly satisfied at every iteration.

### 2. Cost (Objective) Function
With subgrid validity guaranteed by construction, the objective function $f(X)$ measures the total number of duplicate conflicts across all rows and columns:

$$f(X) = \sum_{r=1}^{N} \left( N - \left|\{X_{r,c} : c = 1,\dots,N\}\right| \right) + \sum_{c=1}^{N} \left( N - \left|\{X_{r,c} : r = 1,\dots,N\}\right| \right)$$

A grid $X$ is a valid global optimum if and only if $f(X) = 0$.

---

## ⚙️ Hybrid Metaheuristic Architecture

### 1. Candidate-Domain Constructive Initialization
Before local search begins, each empty cell $(r,c) \notin F$ is assigned a valid candidate domain based on initial clues:

$$C(r,c) = \{1, \dots, N\} \setminus \big(\text{Row}(r) \cup \text{Col}(c) \cup \text{Box}(r,c)\big)$$

Empty cells within each subgrid are populated in randomized order by prioritizing values in $C(r,c)$ before falling back to remaining box symbols, substantially lowering initial conflict cost $f(X_0)$.

### 2. Adaptive VNS + SA Neighborhood Swaps
At temperature $T$, the solver dynamically balances broad exploration and targeted conflict resolution via an adaptive probability:

$$p_{\text{conflict}}(T) = \min\left(0.9, \max\left(0.1, 1 - \frac{T}{T_0}\right)\right)$$

* **Exploration Neighborhood $\mathcal{N}_1$ (Random Intra-Box Swap):** Selected with probability $1 - p_{\text{conflict}}(T)$. Picks a random mutable cell $u \notin F$ and swaps it with a random mutable peer $v \neq u$ in the same $n \times n$ box.
* **Exploitation Neighborhood $\mathcal{N}_2$ (Conflict-Driven, Domain-Filtered VNS Swap):** Selected with probability $p_{\text{conflict}}(T)$. Identifies mutable cells actively causing row or column duplicates, selects a conflicted cell $u$, and filters intra-box mutable peers $v$ to those mutually compatible with initial clue domains:
  $$X_u \in C(v) \quad \text{and} \quad X_v \in C(u)$$
* **Metropolis Acceptance:** A candidate grid $X'$ with $\Delta = f(X') - f(X)$ is accepted if $\Delta \le 0$, or with probability $e^{-\Delta / T}$ when $\Delta > 0$.

### 3. Cost-Scaled Cooling & Dynamic Reheating
* **Initial Temperature:** Scaled to initial grid disorder via $T_0 = 0.5 \cdot f(X_0)$.
* **Geometric Cooling:** $T_{k+1} = \alpha T_k$ (with $\alpha = 0.99$).
* **Stagnation Reheating:** To escape deep local minima on larger boards, if the solver goes $\tau_{\text{reheat}} = N^2 \cdot f(X)^2$ iterations without improving the best-known cost $f(X^*)$, the temperature is reheated to $T \leftarrow P \cdot f(X^*)$ ($P = 0.55$).

---

## 📊 Exact ILP Benchmark & Ablation Suite

To evaluate heuristic efficiency against exact optimization, the notebook includes a **Gurobi (`gurobipy`) ILP formulation** over binary decision variables $x_{i,j,k} \in \{0,1\}$ (cell $(i,j)$ takes value $k+1$), enforcing single-assignment, row, column, subgrid, and clue constraints, alongside a 5-way ablation benchmark:

1. **Pure SA** (random box initialization + random swaps)
2. **SA + Greedy** (candidate-domain initialization + random swaps)
3. **VNS-SA Hybrid** (random initialization + conflict-driven swaps)
4. **VNS-SA Hybrid + Greedy** *(Final Model)*
5. **Exact ILP Solver** (`gurobipy`)

The repository also includes precomputed valid boards (`precomputed_9x9.csv` through `precomputed_49x49.csv`) for generating test puzzles across $3 \times 3$ to $7 \times 7$ subgrid dimensions.

---

## 📈 Development Evolution & Results (10,000 Kaggle 9×9 Puzzles)

* **v1.0.0 (Baseline):** Initial SA implementation inspired by Lewis (2006) — **99.96%** solve rate.
* **v1.1.0 (Scaling Exploration):** Extended formulation to generalized $N \times N$ grids; identified local-minima bottlenecks on $16 \times 16$ instances.
* **v1.2.0 (Schedule Refinement):** Implemented cost-scaled initial temperatures and geometric cost-based reheating inspired by Abramson et al. (1996).
* **v1.3.0 (Final Hybrid):** Integrated conflict-driven, domain-filtered VNS swaps and candidate-based constructive initialization:
  * **Success Rate:** **100%** (10,000 / 10,000 solved)
  * **Average Solve Time:** **0.062s** (~10% speedup over baseline)

---

### Comparison for Different Methods (1000 Kaggle 9×9 Puzzles)

<img width="1389" height="539" alt="image" src="https://github.com/user-attachments/assets/8f98cf01-466b-493b-bdb8-8b1b79995899" />

### Comparison for Different Methods (Current Solution Over Time for $49 \times 49$)

<img width="1014" height="776" alt="image" src="https://github.com/user-attachments/assets/2a735905-ef74-4e8a-8a1f-56252e1284da" />

