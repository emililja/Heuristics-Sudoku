# Heuristic Sudoku Solver: VNS + Simulated Annealing

## 🧩 Project Overview
An optimization engine developed for **SUTD Heuristics and Systems Theory (40.018)**. This project implements a hybrid metaheuristic approach to solve NxN Sudoku puzzles, balancing exploration and exploitation to navigate complex solution spaces.

## 💡 The Hybrid Approach
* **Simulated Annealing (SA):** Uses a probabilistic acceptance criterion (Boltzmann distribution) to escape local minima by occasionally accepting "worse" moves.
* **Variable Neighborhood Search (VNS):** Implements dynamic perturbation logic to jump to new neighborhoods when the SA process reaches a plateau.
* **ILP Benchmarking:** Includes a comparative analysis against an **Integer Linear Programming** solver to evaluate the efficiency and accuracy of the heuristic vs. exact methods.

## 🛠️ Key Technical Skills
* **Optimization Theory:** Cost function design, temperature cooling schedules, and move-set definitions.
* **Performance Analysis:** Benchmarking heuristic convergence speed against traditional solvers.
* **Python (Scientific Stack):** NumPy for matrix operations and Matplotlib for visualizing cost decay.

## 📈 Development Evolution & Benchmarking
This project followed an iterative research-led development process, benchmarking against the 9x9 Kaggle Sudoku dataset (10k puzzles).

### Version History
* **v1.0.0 (Baseline):** Initial SA implementation inspired by Lewis (2006). Achieved 99.96% accuracy.
* **v1.1.0 (Iteration):** Attempted nxn expansion; identified scaling issues with 16x16 grids (documented and pivoted).
* **v1.2.0 (Refinement):** Reworked temperature schedules based on Abramson et al. (1996) for geometric cost-based reheating.
* **v1.3.0 (Final - Current):** Integrated **VNS-style swapping** and candidate-based constructive heuristics. 
    * **Result:** 100% Success Rate (10,000/10,000 solved).
    * **Performance:** Avg time 0.062s (10% improvement over baseline).