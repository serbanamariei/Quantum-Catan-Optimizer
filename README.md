# 🎲 Quantum Catan Optimizer

A hybrid quantum-classical application that applies quantum computing algorithms to solve optimization problems in the board game Catan. Developed during the **Qiskit Fall Fest 2025** hackathon, organized by **Freeyamind** in partnership with **IBM**, where it won an award.

---

## Project Structure

```
.
├── main.py                      # Main application
├── settlement_placement.png     # Output: optimized settlement map
└── longest_road_optimized.png   # Output: longest road visualization
```

---

## Requirements

- Python 3.8+
- The following packages:

```bash
pip install numpy matplotlib networkx scipy qiskit qiskit-aer
```

---

## How to Run

```bash
python main.py
```

The program will prompt you for two inputs:
- **Number of settlements** to place
- **Maximum number of roads** to build

After optimization, results are printed to the terminal and two images are saved.

---

## How It Works

The project solves three interconnected optimization problems, each using a different quantum algorithm.

---

### Problem 1 — Settlement Placement (QAOA)

**Goal:** Find the best starting positions for settlements, maximizing resource diversity and dice roll probability, while enforcing the distance rule (no two settlements on adjacent vertices).

#### Algorithm: QAOA (Quantum Approximate Optimization Algorithm)

QAOA is a hybrid quantum-classical algorithm designed to solve combinatorial optimization problems. It works by encoding the problem into a cost Hamiltonian and using a parameterized quantum circuit to find the bitstring (i.e. the combination of vertices) that minimizes the energy of that Hamiltonian — which corresponds to the best solution.

**Step 1 — QUBO formulation.**
The placement problem is mapped to a **Quadratic Unconstrained Binary Optimization (QUBO)** matrix `Q`. Each vertex on the board is assigned a binary variable: `1` if a settlement is placed there, `0` otherwise. The diagonal entries `Q[i][i]` encode how good vertex `i` is (score based on resource diversity, dice probability, and how many hexes it touches — 3-hex vertices are strongly preferred). The off-diagonal entries `Q[i][j]` encode penalties: a large positive penalty is added for every pair of adjacent vertices `(i, j)` to enforce the distance rule, and additional penalties discourage solutions with the wrong number of settlements.

**Step 2 — QAOA circuit.**
A 2-layer QAOA circuit is constructed:
- All qubits start in superposition via Hadamard gates, representing all possible placements simultaneously.
- Each layer applies two operations: a **cost unitary** (RZZ and RZ gates parameterized by `γ`) that encodes the QUBO energy into the quantum state's phases, and a **mixer unitary** (RX gates parameterized by `β`) that explores the solution space by rotating between states.

**Step 3 — Classical optimization.**
The circuit is run on a simulated quantum backend (`AerSimulator`). A classical optimizer (**COBYLA**) iteratively adjusts the parameters `γ` and `β` to minimize the expected energy of the output measurements. This feedback loop between the quantum circuit and the classical optimizer is what makes QAOA a hybrid algorithm.

**Step 4 — Solution extraction.**
After optimization, the circuit is measured 1000 times. Each measurement produces a bitstring representing a candidate placement. Valid candidates (correct number of settlements, no adjacent pairs) are ranked by a combined score of base vertex quality, resource diversity, and average dice probability. The best valid solution is returned.

If no valid quantum solution is found among the measurements, a **classical greedy fallback** is used: vertices are ranked by their tile count and score, and selected greedily while respecting the distance rule.

**Output:** `settlement_placement.png`

![Settlement Placement](settlement_placement.png)

---

### Problem 2 — Longest Road (Quantum Walk Inspired DFS)

**Goal:** Starting from the settlements chosen in Problem 1, find the longest simple path through the road network — a path with no branching and no repeated edges.

#### Algorithm: Quantum Walk Inspired Depth-First Search

Finding the longest simple path in a graph is an **NP-Hard** problem. The approach here is inspired by **quantum walks**, which are the quantum analogue of classical random walks. In a classical random walk, a particle moves to a random neighbor at each step. In a quantum walk, the particle exists in a superposition of positions and explores multiple paths simultaneously through interference — making it naturally suited for graph traversal problems.

While this implementation runs on a classical simulator, the traversal strategy mirrors the structure of a quantum walk: the algorithm explores the graph from each settlement in a depth-first manner, maintaining a superposition-like branching of paths and pruning branches that violate the simple-path constraint.

**Step 1 — Graph construction.**
The Catan board is modeled as a graph `G(V, E)` where vertices are the intersection points of hexagonal tiles and edges are the roads between them.

**Step 2 — Constraint-enforced DFS.**
From each settlement, a recursive DFS explores all possible road extensions. At each step, the algorithm checks:
- The edge has not already been used in the current path.
- Adding the new edge does not create **branching** (no vertex in the path may have degree > 2, which would mean more than two roads meeting at a point — not a simple path).
- The path does not return to the starting settlement after leaving it.
- The path does not pass through another settlement.

Branches that violate any of these constraints are pruned immediately, keeping the search efficient.

**Step 3 — Best path selection.**
The algorithm runs from every settlement and keeps the globally longest valid path. If the user requests more roads than are physically achievable on the board, the maximum possible path is returned instead, with a warning.

**Output:** `longest_road_optimized.png`

![Longest Road](longest_road_optimized.png)

---

### Problem 3 — Resource Trading (Grover's Algorithm)

**Goal:** Given the player's current resources, find the optimal combination of actions (Build Road, Build Settlement, Build City, Buy Dev Card) that maximizes Victory Points.

#### Algorithm: Grover's Search

Grover's Algorithm is a quantum search algorithm that finds a marked item in an unsorted database of `N` items in `O(√N)` steps, compared to `O(N)` for a classical linear search. It works through two key components repeated for `π/4 · √N` iterations:

**The Oracle.**
The oracle is a quantum gate that identifies "good" solutions — in this case, combinations of actions that are affordable given the player's resources. It applies a phase flip (`Z` gate) to the qubits encoding valid states, marking them without measuring or collapsing the superposition.

**The Diffusion Operator (Amplitude Amplification).**
After the oracle marks valid states, the diffusion operator reflects the quantum state around the average amplitude. This systematically increases the probability amplitude of marked states and decreases it for unmarked ones. After enough iterations, measuring the circuit yields a marked (valid, high-value) state with high probability.

**Circuit structure.**
Each qubit represents a possible action. All qubits start in equal superposition via Hadamard gates, meaning all action combinations are considered simultaneously. The oracle and diffusion operator are applied for the optimal number of Grover iterations (`π/4 · √2^n` for `n` qubits). The circuit is then measured 100 times per batch, and each resulting bitstring is decoded into a set of actions and evaluated for total Victory Points.

The best solution found across all iterations is returned and compared against a classical greedy baseline (which ranks actions by points-per-resource-cost ratio).

---

## Tech Stack

| Component | Technology |
|---|---|
| Language | Python 3.8+ |
| Quantum Computing | Qiskit, Qiskit AerSimulator |
| Classical Optimization | SciPy (COBYLA) |
| Graph Theory | NetworkX |
| Visualization | Matplotlib |

---

## Configuration

At the top of `main.py` you can tune the following constants:

```python
MAP_SEED = None        # Set an integer for a reproducible map, None for random
DIVERSITY_WEIGHT = 2.0
PROBABILITY_WEIGHT = 1.5
QAOA_ITERATIONS = 30
ROAD_ITERATIONS = 500
TRADE_ITERATIONS = 1000
```

---

*Project created for the Qiskit Fall Fest 2025 Hackathon.*
