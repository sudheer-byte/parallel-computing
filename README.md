# ⚛️ Parallel Molecular Dynamics Particle Simulation

> A 2D Lennard-Jones particle simulator parallelized with OpenMP — featuring serial, parallel, and cell-list cutoff implementations.

**Course:** Concurrent and Parallel Programming (Spring 2026)  
**Author:** Sudheer Sunkara  
**Type:** Individual Final Project

---

## 📁 Project Structure

```
.
├── serial_md.cpp                                        # Baseline serial implementation
├── parallel_md.cpp                                      # OpenMP parallel version
├── parallel_md_cutoff.cpp                               # OpenMP + cell-list optimization
├── Parallel_Molecular_Dynamics_Particle_Simulation.docx # Final report
└── README.md
```

---

## 🔬 Overview

This project implements a **2D Molecular Dynamics simulator** using the Lennard-Jones interaction model to study how particles move and interact inside a bounded box — and how parallel programming improves simulation performance.

Three implementations are provided:

| Implementation | Description |
|---|---|
| `serial_md.cpp` | Baseline serial. Uses Newton's 3rd law for efficient pairwise force computation. Single-threaded. |
| `parallel_md.cpp` | OpenMP parallelization with thread-private force buffers, dynamic scheduling for forces, static scheduling for particle updates. |
| `parallel_md_cutoff.cpp` | OpenMP + 8×8 cell-list grid. Only checks particle pairs within the cutoff radius, reducing complexity from **O(N²) → O(N)**. |

---

## ⚙️ Simulation Parameters

| Parameter | Value | Description |
|---|---|---|
| `BOX_SIZE` | 80.0 | Side length of the 2D simulation box |
| `DT` | 0.0005 | Time step size |
| `EPSILON` | 1.0 | Lennard-Jones energy parameter |
| `MIN_R2` | 0.0001 | Minimum r² to prevent division by zero |
| `CUTOFF` | 10.0 | Force cutoff radius (cutoff version only) |
| Particle Mass | 1.0 | Implicit, reduced units |
| Random Seed | 42 | Fixed for reproducibility across all versions |

---

## 🛠️ Build

> **Requirements:** Linux or WSL · g++ (C++11+) · OpenMP · 4–8 CPU cores recommended

```bash
# Serial
g++ -O3 -o serial_md serial_md.cpp

# OpenMP Parallel
g++ -O3 -fopenmp -o parallel_md parallel_md.cpp

# OpenMP + Cell-List Cutoff
g++ -O3 -fopenmp -o parallel_md_cutoff parallel_md_cutoff.cpp
```

---

## 🚀 Usage

```bash
./serial_md          <num_particles> <num_steps>
./parallel_md        <num_particles> <num_steps> <num_threads>
./parallel_md_cutoff <num_particles> <num_steps> <num_threads>
```

**Defaults** (if no arguments given): `num_particles=1000`, `num_steps=100`, `num_threads=4`

**Stable particle counts:** 200, 500, 1000, 2000  
**⚠️ Unstable (do not use):** N ≥ 5000 — forces diverge at current timestep/density settings.

### Example Runs

```bash
# Serial baseline
./serial_md 1000 500

# Parallel — vary threads (N=1000)
./parallel_md 1000 500 1
./parallel_md 1000 500 2
./parallel_md 1000 500 4
./parallel_md 1000 500 8
./parallel_md 1000 500 16

# Cell-list cutoff — vary threads (N=1000)
./parallel_md_cutoff 1000 500 1
./parallel_md_cutoff 1000 500 2
./parallel_md_cutoff 1000 500 4
./parallel_md_cutoff 1000 500 8
./parallel_md_cutoff 1000 500 16

# Parallel — vary particles (4 threads)
./parallel_md 200  500 4
./parallel_md 500  500 4
./parallel_md 1000 500 4
./parallel_md 2000 500 4
```

---

## 📊 Performance Results

> Benchmarked at N=1000 particles, 500 steps

| Version | Threads | Time (s) | Speedup vs Serial |
|---|---|---|---|
| Serial Baseline | — | 1.64871 | 1.00× |
| OpenMP Parallel | 1 | 1.63353 | 1.01× |
| OpenMP Parallel | 2 | 0.93386 | 1.77× |
| OpenMP Parallel | 4 | 0.55999 | 2.95× |
| OpenMP Parallel | 8 | 0.37528 | **4.39×** |
| OpenMP Parallel | 16 | 0.44121 | 3.74× |
| Cell-List Cutoff | 1 | 0.16069 | 10.26× |
| Cell-List Cutoff | 2 | 0.10800 | **15.27× ✅ Best** |
| Cell-List Cutoff | 4 | 0.10834 | 15.22× |
| Cell-List Cutoff | 8 | 0.11765 | 14.01× |
| Cell-List Cutoff | 16 | 0.25138 | 6.56× |

**Key Findings:**
- OpenMP alone peaks at **8 threads (4.39×)**
- Cell-list peaks at **2 threads (15.27×)** — the algorithmic improvement dominates
- 16 threads underperforms 8 in both versions due to thread management overhead
- **Algorithmic optimization outperforms raw parallelism**

---

## 🧪 Output Format

Every 10 simulation steps:
```
Step <N>  Kinetic Energy=<KE>  Potential Energy=<PE>  Total Energy=<KE+PE>  Temperature (reduced)=<T>
```

At the end of each run:
```
Total wall time: <seconds>
Threads used: <N>
```

---

## 🧲 Physical Model

**Force:** Lennard-Jones pairwise interaction
```
F = 24 × ε × (2·r⁻¹² − r⁻⁶) / r²
```

**Integration:** Euler integration with fixed time step `DT = 0.0005`

**Boundaries:** Reflective walls (particles bounce)

**Energy tracking:**
- Kinetic Energy = `0.5 × mass × v²` (summed over all particles)
- Potential Energy = `4 × ε × (r⁻¹² − r⁻⁶)` (summed over pairs)
- Temperature = `KE / N` (reduced units, not Kelvin)

**Initialization:** Grid-based positions to prevent overlap; random velocities via Mersenne Twister (`seed=42`)

---

## 🔒 Race Condition Solution

Newton's 3rd law requires updating **two particles per pair interaction**, which causes data races in naive parallel implementations.

**Solution:** Each OpenMP thread writes forces into its own **private buffer**. After the parallel loop, buffers are summed into the main force array via a second parallel reduction loop — no locks, no atomics, no races.

---

## ⚠️ Known Limitations

- 2D simulation only (not 3D)
- Euler integration (not Velocity-Verlet)
- No periodic boundary conditions (reflective walls only)
- Temperature in reduced units (not physical Kelvin)
- Cutoff version uses a fixed 8×8 cell grid regardless of N
- N ≥ 5000 is numerically unstable at current settings — reduce `DT` or increase `BOX_SIZE` for larger systems
