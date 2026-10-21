# QAOA – Portfolio Optimization Demo

A technical demonstration mapping a classical financial optimization problem (Markowitz
mean-variance) to a combinatorial Quadratic Unconstrained Binary Optimization (QUBO) problem, solved
via the Quantum Approximate Optimization Algorithm (QAOA) using IBM Qiskit.

The project explores how a classical financial optimization problem can be transformed into a
combinatorial optimization problem and subsequently formulated for a quantum optimization algorithm.

## Overview

Portfolio optimization aims to find an allocation that balances expected return and risk.
Historical market data is obtained using [`yfinance`](https://github.com/ranaroussi/yfinance).

The optimization objective combines expected returns and portfolio risk and is formulated in a
Markowitz mean-variance model. The resulting Quadratic Unconstrained Binary Optimization (QUBO) is
then mapped to an Ising Hamiltonian and used as the cost Hamiltonian for QAOA.

The project therefore follows the workflow:

```text
Market Data (yfinance)
    ↓
Expected Returns & Covariance Matrix
    ↓
Markowitz Mean-Variance Model (Budget constraint)
    ↓
QUBO Formulation (Penalty factor α)
    ↓
Ising Hamiltonian (Cost operator H_C)
    ↓
QAOA Circuit (Statevector Simulation & COBYLA)
    ↓
Measurement & Evaluation
```

## Results

Instance: four assets (GOOG, AMZN, META, NVDA), 2025 daily prices (frozen in
`data/`), budget B = 2, risk/return trade-off λ = 0.5. Enumerating the six
feasible portfolios gives GOOG+NVDA as the optimum.

| QAOA depth                          | P(optimal portfolio) |
|-------------------------------------|----------------------|
| p = 1                               | 56.6 %               |
| p = 2                               | 62.7 %               |
| Uniform random (1 of 16 bitstrings) | 6.25 %               |

Probabilities are estimated from 10,000 shots (seed 42) sampled from the exact
statevector. Parameters are optimized on exact expectation values: grid search
at p = 1, then COBYLA. At p = 2 the p = 1 optimum is repeated in both layers as
the starting point. No noise model, no hardware.
Outputs: `results/results.json` (p = 1), `results/results_p2.json` (p = 2).

## Project Structure

The project is structured as a modular Python package (portfolio_qaoa).

- `data/`: Frozen historical market data (.csv and .npz) to guarantee deterministic execution.
- `src/portfolio_qaoa/`: The core package module containing the financial model, QUBO conversion,
  QAOA execution, and reporting logic.
- `src/run_data.py`: Script for data acquisition and annualization.
- `src/run_qaoa.py`: Main execution script for the quantum optimization pipeline.
- `results/`: Tracked JSON outputs of optimization runs (includes Git commit hashes for
  reproducibility).
- `images/`: Generated matplotlib figures (convergence, landscapes, circuit diagrams).

## Scope and Honest Limitations

This is purely an educational demonstration of encoding binary asset-selection into a QUBO and
sampling it via QAOA.

- No Quantum Advantage: The default instance handles 4 assets (16 total states) and is simulated
  classically. The exact feasible optimum is trivially found by enumeration.
- Simplified Financial Model: It uses historical estimates from a single year and does not model
  continuous asset weights, share counts, transaction costs, or trading constraints.
- Disclaimer: This project does not constitute financial advice or an investment recommendation.

## Installation and Run

Built with Python 3.14+ and Qiskit 2.5+.

Clone the repository:

```bash
# Clone the repository
git clone https://github.com/Wendo99/QAOA-Portfolio-Optimization.git

cd QAOA-Portfolio-Optimization

# Install dependencies via uv
uv sync

# Run the optimization pipeline
uv run python src/run_qaoa.py
```

## References

- J. S. Baker und S. K. Radha, „Wasserstein Solution Quality and the Quantum Approximate
  Optimization Algorithm: A Portfolio Optimization Case Study“, 14. Februar 2022, arXiv: arXiv:
  2202.06782. doi: 10.48550/arXiv.2202.06782.
- K. Blekos u. a., „A review on Quantum Approximate Optimization Algorithm and its variants“,
  Physics Reports, Bd. 1068, S. 1–66, Juni 2024, doi: 10.1016/j.physrep.2024.03.002.
- E. Farhi, J. Goldstone, und S. Gutmann, „A Quantum Approximate Optimization Algorithm“, 14.
  November 2014, arXiv: arXiv:1411.4028. doi: 10.48550/arXiv.1411.4028.
- M. Hodson, B. Ruck, H. Ong, D. Garvin, und S. Dulman, „Portfolio rebalancing experiments using the
  Quantum Alternating Operator Ansatz“, 13. November 2019, arXiv: arXiv:1911.05296. doi:
  10.48550/arXiv.1911.05296.
- J. Stein, M. Zorn, L. Sünkel, und T. Gabor, „Quantum Optimization Algorithms“, 15. November 2025,
  arXiv: arXiv:2511.12379. doi: 10.48550/arXiv.2511.12379.
