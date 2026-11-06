# Quantum Portfolio Optimization with QAOA

An educational implementation of the Quantum Approximate Optimization Algorithm (QAOA) for binary portfolio selection. The project transforms a Markowitz mean–variance objective with a budget constraint into a quadratic unconstrained binary optimization (QUBO) problem and then an Ising cost Hamiltonian. It uses IBM Qiskit for noiseless statevector simulation, not quantum hardware.

The emphasis is on a transparent end-to-end experiment: data preparation, mathematical formulation, hybrid optimization, measurement analysis and reproducible reporting.

## Quick start

**Requirements**: Python 3.14 or newer and [uv](https://docs.astral.sh/uv/). The dependency constraints, including Qiskit, are specified in [`pyproject.toml`](pyproject.toml), with versions resolved in [`uv.lock`](uv.lock).

```bash
git clone https://github.com/Wendo99/QAOA-Portfolio-Optimization.git
cd QAOA-Portfolio-Optimization
uv sync
uv run python src/run_qaoa.py
```

The default run reads the included [`data/portfolio_data.npz`](data/portfolio_data.npz), prints the optimized parameters and portfolio probabilities, and writes a structured report and plots. It does not require a new market-data download.

To regenerate market data rather than use the frozen instance, run the following separately (requires internet access and depends on the historical data returned by the provider):

```bash
uv run python src/run_data.py
```

That script downloads price history using `yfinance` and updates `data/prices_2025.csv` and `data/portfolio_data.npz`. Regenerating data can change the optimization instance; use the committed files for comparison with the documented experiment.

## Architecture and workflow

```text
Historical market prices (yfinance, optional refresh)
                        |
                        v
                src/run_data.py
              annualized returns
              + covariance matrix
                        |
                        v
             data/portfolio_data.npz
                        |
                        v
         Markowitz objective + budget
                        |
                        v
              QUBO -> Ising model
                        |
                        v
          p=1 grid search -> COBYLA
                        |
                        v
       statevector sampling + analysis
                        |
                        v
        results/*.json + images/*.png
```

| Component | Responsibility |
| --- | --- |
| [`data.py`](src/portfolio_qaoa/data.py) | Acquire adjusted price history, compute annualized moments, save frozen inputs |
| [`problem.py`](src/portfolio_qaoa/problem.py) | Construct the binary portfolio model, QUBO and Ising representation; enumerate feasible outcomes |
| [`qaoa.py`](src/portfolio_qaoa/qaoa.py) | Build QAOA circuits, evaluate expected cost, perform grid search and COBYLA optimization, sample outcomes |
| [`report.py`](src/portfolio_qaoa/report.py) | Compute result summaries and save traceable JSON reports |
| [`plots.py`](src/portfolio_qaoa/plots.py) | Produce landscapes, convergence, distribution and circuit visualizations |
| [`config.py`](src/portfolio_qaoa/config.py) | Define paths and experiment parameters |
| [`run_data.py`](src/run_data.py) | Data-generation entry point |
| [`run_qaoa.py`](src/run_qaoa.py) | Optimization and reporting entry point |

The Python package is named `portfolio_qaoa` and uses a `src/` layout with packaging metadata in `pyproject.toml`.

## Experimental setup and results

The reported experiment selects exactly two assets out of four (GOOG, AMZN, META and NVDA) using frozen 2025 daily price data. The risk–return trade-off parameter is λ = 0.5, with a two-asset budget. Exhaustive enumeration identifies GOOG + NVDA as the optimal feasible portfolio for the chosen model and inputs.

| Configuration | Estimated probability of optimal portfolio |
| --- | ---: |
| QAOA depth `p = 1` | 56.6% |
| QAOA depth `p = 2` | 62.7% |
| Uniform random sampling across 16 bit strings | 6.25% |

The probabilities are estimated from 10,000 shots (seed 42) sampled from the exact statevector. A grid search over the `p = 1` energy landscape provides a starting point for COBYLA. For `p = 2`, the single-layer starting parameters are repeated across both layers. Parameter optimization uses exact expectation values; the reported probabilities are based on sampling. The experiment includes no noise model or hardware execution.

The example's result artifacts are retained as [`results/results.json`](results/results.json) (`p = 1`) and [`results/results_p2.json`](results/results_p2.json) (`p = 2`). Figures are stored in [`images/`](images/).

## Reproducibility and validation

Reproducibility is supported through several concrete design choices:

- **Frozen inputs:** `data/prices_2025.csv` and `data/portfolio_data.npz` make it possible to rerun the saved instance without depending on later market-data downloads.
- **Explicit configuration:** [`Config`](src/portfolio_qaoa/config.py) records the asset budget, λ, circuit depth, grid resolution, COBYLA parameters, shot count and random seed.
- **Structured reporting:** The JSON reports capture model and optimization parameters, optimization history, feasible and infeasible portfolios, sampled probabilities, timestamps, Python/library versions, Git commit SHA and whether the working tree was modified (when Git is available).
- **Independent check:** With only four assets, the program can enumerate the feasible portfolios and compare the sampled distribution against the exact classical optimum. This is a correctness reference for the selected small instance, not evidence of quantum advantage.
- **Non-overwriting output:** [`run_qaoa.py`](src/run_qaoa.py) appends a variant/counter to new report and image names when the intended paths already exist, preserving previous runs.

For example, after running the project, inspect the newly printed results filename and compare the `result` and `versions` sections of the corresponding JSON report:

```python
import json
from pathlib import Path

report = json.loads(Path("results/results_p2.json").read_text())
print(report["result"]["P_opt"])
print(report["versions"]["python"])
print(report["versions"]["git_commit"])
```

The example reads the committed `p = 2` report. New runs generally create a different filename to avoid overwriting existing artifacts.

Testing status: This repository does not currently contain a dedicated automated test suite. The implementation has a small-instance exhaustive comparison and retained experimental results, but these are not a substitute for systematic unit and integration tests. Useful future tests would cover QUBO/Ising energy consistency, budget feasibility, bitstring mappings, report schema and seeded sampling behaviour.

## Project structure

```text
data/                  # Frozen prices and portfolio parameters
src/portfolio_qaoa/    # Reusable model, optimization, reporting and plotting code
src/run_data.py        # Refresh market data and compute moments
src/run_qaoa.py        # Run the QAOA experiment
results/               # JSON reports including experiment metadata
images/                # Generated plots and circuit diagrams
pyproject.toml         # Package and dependency metadata
uv.lock                # Resolved dependencies
```

## Scope and limitations

- **No claim of quantum advantage:** Four assets produce only 16 binary states, so exhaustive classical enumeration is straightforward. All simulations are classical statevector computations.
- **No hardware/noise analysis:** Results do not predict performance on noisy quantum hardware.
- **Simplified financial setting:** The model uses one historical year, binary asset selection and a fixed budget. It excludes transaction costs, changing markets, continuous asset weights and additional constraints.
- **Reproducibility scope:** Frozen data and explicit metadata improve traceability, but package versions, seeds, operating environments and software updates can still influence numerical behaviour. Re-downloading data may create a different instance.
- **Testing:** The project has no dedicated automated test suite at present.
- **Financial disclaimer:** This is an educational experiment, not financial advice or an investment recommendation.

## References

- Baker, J. S., & Radha, S. K. (2022). *Wasserstein Solution Quality and the Quantum Approximate Optimization Algorithm: A Portfolio Optimization Case Study*. [arXiv:2202.06782](https://doi.org/10.48550/arXiv.2202.06782).
- Blekos, K., et al. (2024). *A review on Quantum Approximate Optimization Algorithm and its variants*. *Physics Reports, 1068*, 1–66. [DOI](https://doi.org/10.1016/j.physrep.2024.03.002).
- Farhi, E., Goldstone, J., & Gutmann, S. (2014). *A Quantum Approximate Optimization Algorithm*. [arXiv:1411.4028](https://doi.org/10.48550/arXiv.1411.4028).
- Hodson, M., Ruck, B., Ong, H., Garvin, D., & Dulman, S. (2019). *Portfolio rebalancing experiments using the Quantum Alternating Operator Ansatz*. [arXiv:1911.05296](https://doi.org/10.48550/arXiv.1911.05296).
- Stein, J., Zorn, M., Sünkel, L., & Gabor, T. (2025). *Quantum Optimization Algorithms*. [arXiv:2511.12379](https://doi.org/10.48550/arXiv.2511.12379).
