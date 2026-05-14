# 🎲 Quantum Catan Optimizer

This project was developed during the **Qiskit Fall Fest 2025**, organized by **Freeyamind** in partnership with **IBM**, where it won an award during the hackathon.

The project explores the application of quantum computing and hybrid (quantum-classical) algorithms to solve complex optimization problems in the game of Catan. It utilizes the **Qiskit** framework to simulate quantum circuits that optimize settlement placement, road construction, and resource management.

## 🏆 Project Context
In the game of Catan, making optimal decisions requires evaluating a massive number of variables: dice roll probabilities, resource diversity, distance rules, and construction costs. This project demonstrates how quantum computing can tackle these combinatorial optimization problems more efficiently than classical "brute-force" methods.

## 🚀 Features and Algorithms

### 1. Settlement Placement Optimization (QAOA)
The first step in Catan is choosing the initial intersections. The algorithm uses the **Quantum Approximate Optimization Algorithm (QAOA)** to find the best locations.
* **QUBO Modeling**: The problem is formulated as a QUBO matrix.
* **Cost Function**: Rewards nodes at 3-hex intersections, resource diversity, and high-probability numbers (6, 8).
* **Constraints**: Penalizes adjacent placements (distance rule) and incorrect settlement counts.

### 2. Longest Road (Quantum Walk / DFS)
Finds the longest simple path (no branching) starting from an existing settlement.
* **Graph Generation**: Uses `NetworkX` to map the board.
* **Logic**: Explores paths that comply with Catan rules, ensuring no branching (validating node degrees).

### 3. Resource Trading and Allocation (Grover's Algorithm)
Determines the best way to spend resources to maximize victory points.
* **Amplitude Amplification**: Uses Grover-inspired concepts to find the optimal combination of purchases (Roads, Settlements, Cities, Dev Cards) that the player can afford.

## 🛠️ Visualization and Outputs
The script generates visual representations of the decisions:
* `settlement_placement.png`: Map showing the optimized settlement locations.
* `longest_road_optimized.png`: Map showing the longest valid road path.

## 💻 How to Run the Project

### Prerequisites
Make sure you have Python installed (Python 3.8+ recommended).

### Installing Dependencies
You will need the following Python packages:

```bash
pip install numpy matplotlib networkx scipy qiskit qiskit-aer
```

### Execution
Run the main file directly from your terminal:

```bash
python main.py
```

The program will prompt you in the terminal to enter the desired number of settlements (e.g., `3`) and the maximum number of roads (e.g., `10`). After the calculations, it will display a summary and save the images.

## ⚙️ Tech Stack
* **Language**: Python
* **Quantum Computing**: Qiskit, Qiskit AerSimulator
* **Classical Optimization**: SciPy (COBYLA)
* **Graph Theory**: NetworkX
* **Visual Rendering**: Matplotlib

---
*Project created for the Qiskit Fall Fest 2025 Hackathon.*
