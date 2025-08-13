# C++ Simulation Benchmarking Tool

This project provides a fast, configurable command-line toolkit for benchmarking 1D finite-difference solvers on canonical PDEs: the heat equation and linear advection. It focuses on correctness and speed, with built-in stability checks (CFL), standardized output for analysis, and reproducible runs. Users can sweep grid sizes, time steps, and schemes to compare runtime, accuracy, and stability envelopes under consistent conditions.

---

## Abstract


This project provides a fast, configurable CLI tool for benchmarking classic 1D PDE solvers. It implements finite difference schemes for the heat and linear advection equations, with built-in stability checks (CFL), timing/throughput metrics, and accuracy reporting against reference solutions. The goal is to make it easy to compare schemes, timesteps, and grid resolutions while keeping the codebase small, modern C++, and ready for extension.

---

## Project Overview

This project demonstrates how to set up reliable and comparable PDE experiments in C++:

1. **Solvers** — Implement explicit (and optional implicit) finite-difference schemes for:
   - Heat equation $u_t = \alpha u_{xx}$
   - Advection equation $u_t + c$ , $u_x = 0$
2. **Stability Analysis** — Enforce and report CFL restrictions before each run.
3. **Benchmarking** — Record runtime, iteration counts, and (optionally) error metrics vs. known/analytic references.
4. **Configurable CLI** — Set `--equation`, `--nx`, `--dt`, `--tmax`, `--scheme`, and I/O options from the command line.
5. **Reproducibility & Output** — Emit structured results (CSV/JSON) and metadata for downstream plotting.

---

## Repository Structure

*(Planned — subject to change as implementation progresses)*

```text
simulation-bench/
├── cmake/                # CMake presets / toolchain files (optional)
├── data/                 # Example IC/BC configs & output samples
├── scripts/              # Helper scripts (plotting, sweeps)
├── src/
│   ├── core/             # Discretization kernels, CFL checks, timers
│   ├── equations/        # Heat, Advection equation-specific code
│   ├── io/               # CSV/JSON writers, config parsing
│   ├── cli/              # Argument parsing & run dispatch
│   └── main.cpp          # Entry point
├── tests/                # Unit/benchmark tests (GoogleTest)
├── CMakeLists.txt        # Build configuration
├── README.md
└── LICENSE
```

---

## Methods

- **Finite Difference Schemes**:
  - Heat: explicit forward-Euler in time, centered second-order in space (and optional implicit/backward-Euler or Crank-Nicolson).
  - Advection: explicit upwind (default) and optional Lax-Friedrichs.
- **Typical CFL Conditions**:
  - Heat (explicit FTCS): \
    $$\displaystyle{\Delta t \leq \frac{\Delta x^2}{2\alpha}}$$
  - Advection (upwind/LF): \
    $$\displaystyle{\Delta t \leq \frac{\Delta x}{|c|}}$$ \
    The tool computes and reports recommended 
- **Boundary/Initial Conditions**:
  - Dirichlet and periodic (planned options).
  - Built-in initial condition templates (e.g., Gaussian pulse, step, sine).
- **Error & Profiling**:
  - L2/L errors vs. analytic or reference solutions.
  - Timing via lightweight RAII timers; aggregated stats per run.

---

## Technology Stack

- **Language**: C++17 (or later)
- **Build**: CMake
- **CLI**: Single-header arg parser (e.g., argparse-like) or custom
- **Testing**: GoogleTest
- **Visualization**: Python (matplotlib) helper script in ```scripts/```

---

# Build & Install

```bash
# Clone
git clone https://github.com/<your-username>/simulation-bench.git
cd simulation-bench

# Configure & build (Release by default)
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j

# Binary
./build/simulation_bench --help
```

---

## Usage

```bash
./simulation_bench \
  --equation heat \
  --nx 400 \
  --lx 1.0 \
  --alpha 0.01 \
  --dt 2.5e-6 \
  --tmax 0.05 \
  --scheme explicit \
  --ic gaussian \
  --bc dirichlet \
  --out results/heat_run.csv \
  --report-cfl
```

### Key Arguments

- ```--equation```: ```heat``` | ```advection```
- ```--nx``` : number of spatial grid points
- ```lx```: domain length (default ```1.0```)
- ```--dt```, ```--tmax```: time step and final time
- ```--scheme```: ```explicit``` | ```implicit```| ```crank-nicolson``` | ```upwind``` | ```lax-friedrichs``` (subset depends on equation)
- ```--alpha```: diffusion coefficient (heat)
- ```--c```: advection speed (advection)
- ```--ic```: ```gaussian``` | ```sine```| ```step``` | ```file:<path>```
- ```--bc```: ```dirichlet``` | ```periodic```
- ```--out```: output CSV/JSON path
- ```report-cfi```: print CFL limits and safety factors
- ```--seed```: RNG seed (if stochastic ICs are added later)
- ```--quiet```: minimal console output

---

## Examples

```bash
# Heat (explicit, CFL reported)
./simulation_bench --equation heat --nx 200 --lx 1.0 --alpha 0.01 \
  --dt 2.0e-5 --tmax 0.05 --scheme explicit --ic gaussian --bc dirichlet --report-cfl

# Advection (upwind) with stable dt
./simulation_bench --equation advection --nx 400 --lx 2.0 --c 1.0 \
  --dt 0.0025 --tmax 2.0 --scheme upwind --ic step --bc periodic

# Write JSON for downstream plotting
./simulation_bench --equation heat --nx 300 --alpha 0.02 \
  --dt 1.0e-5 --tmax 0.02 --scheme explicit --out results/run.json
```

---

## Benchmarking Workflow

1. Fix domain/IC/BC and vary $\Delta x$ and $\Delta t$ to map stable vs. unstable regions.
2. For stable runs, record runtime and (if available) error vs. reference.
3. Export CSV/JSON; plot runtime vs. error in ```scripts/plot_benchmarks.py```.
4. Compare schemes (e.g., explicit vs. Crank-Nicolson) at equal accuracy targets.

---

## Tests

- Unit tests for CFL checks, stencil updates, and I/O
- Optional regression tests against known solutions

    ```bash
    cmake -S . -B build -DENABLE_TESTS=ON
    cmake --build build -j
    ctest --test-dir build
    ```

---

## Future work

- Add Crank–Nicolson (heat) and TVD schemes (advection)
- Adaptive time-stepping and automatic CFL selection
- Multi-right-hand-side parameter sweeps via config files
- 2D extensions and GPU backends

---

## License

MIT License




