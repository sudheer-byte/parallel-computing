# Parallel Molecular Dynamics Particle Simulation Using OpenMP

**Student:** Sudheer Sunkara  
**Course:** Concurrent and Parallel Programming (Spring 2026)  
**Project Type:** Individual Project  
**Submission:** Final Project

## 1. PROJECT OVERVIEW

This project implements a two-dimensional Molecular Dynamics (MD) particle simulator using the Lennard-Jones interaction model. The goal is to simulate how particles move and interact inside a bounded box, and to study how parallel programming improves simulation performance.

Three implementations are provided:

1. **serial_md.cpp**  
   - Baseline serial implementation  
   - Uses Newton's third law to compute pairwise forces efficiently  
   - All force, velocity, and position updates run on a single thread  
   - Used as the performance baseline for speedup calculations

2. **parallel_md.cpp**  
   - OpenMP parallel implementation  
   - Parallelizes the force calculation using #pragma omp parallel  
   - Uses thread-private force buffers to avoid data races  
   - Uses dynamic scheduling to balance uneven loop workloads  
   - Particle update step parallelized with static scheduling  
   - Best performance at 8 threads (4.39x speedup vs serial)

3. **parallel_md_cutoff.cpp**  
   - OpenMP parallel + cell-list cutoff optimization  
   - Divides the simulation box into a grid of cells (8x8)  
   - Only checks particle pairs in the same or neighboring cells  
   - Ignores pairs beyond the cutoff radius (CUTOFF = 10.0)  
   - Reduces algorithmic complexity from O(N²) to approximately O(N)  
   - Best overall performance: 15.27x speedup vs serial at 2 threads

## 2. FILES INCLUDED

- serial_md.cpp – Serial implementation  
- parallel_md.cpp – OpenMP parallel version  
- parallel_md_cutoff.cpp – OpenMP + cell-list version  
- Parallel_Molecular_Dynamics_Particle_Simulation.docx – Final report  
- README.md – This file

## 3. SIMULATION PARAMETERS

| Parameter       | Value   | Description                                      |
|-----------------|---------|--------------------------------------------------|
| BOX_SIZE        | 80.0    | Side length of the 2D simulation box             |
| DT              | 0.0005  | Time step size                                   |
| EPSILON         | 1.0     | Lennard-Jones energy parameter                   |
| MIN_R2          | 0.0001  | Minimum r² to prevent division by zero           |
| CUTOFF          | 10.0    | Force cutoff radius (cutoff version only)        |
| Particle Mass   | 1.0     | Implicit, reduced units                          |
| Random Seed     | 42      | Fixed for reproducibility across versions        |

## 4. SYSTEM REQUIREMENTS

- Linux or WSL (Windows Subsystem for Linux)  
- g++ compiler with C++11 or later  
- OpenMP support (included with most g++ installations)  
- Recommended: 4 to 8 CPU cores for best parallel performance

## 5. COMPILE COMMANDS

```bash
# Serial
g++ -O3 -o serial_md serial_md.cpp

# Parallel (OpenMP)
g++ -O3 -fopenmp -o parallel_md parallel_md.cpp

# Parallel with cell-list cutoff
g++ -O3 -fopenmp -o parallel_md_cutoff parallel_md_cutoff.cpp

6. RUN COMMANDS
Bash# Serial
./serial_md <num_particles> <num_steps>

# Parallel
./parallel_md <num_particles> <num_steps> <num_threads>

# Parallel with cutoff
./parallel_md_cutoff <num_particles> <num_steps> <num_threads>
Arguments:

num_particles: Number of particles (e.g. 200, 500, 1000, 2000)
num_steps: Number of simulation time steps (e.g. 100, 500)
num_threads: Number of OpenMP threads (e.g. 1, 2, 4, 8, 16)

Default values: 1000 particles, 100 steps, 4 threads.
7. EXAMPLE RUNS
Bash# Serial baseline
./serial_md 1000 500

# Serial particle scaling
./serial_md 200 500
./serial_md 500 500
./serial_md 1000 500
./serial_md 2000 500

# Parallel - vary threads (N=1000)
./parallel_md 1000 500 1
./parallel_md 1000 500 2
./parallel_md 1000 500 4
./parallel_md 1000 500 8
./parallel_md 1000 500 16

# Cell-list cutoff - vary threads (N=1000)
./parallel_md_cutoff 1000 500 1
./parallel_md_cutoff 1000 500 2
./parallel_md_cutoff 1000 500 4
./parallel_md_cutoff 1000 500 8
./parallel_md_cutoff 1000 500 16
8. OUTPUT FORMAT
Every 10 simulation steps, each program prints:
textStep <N>  Kinetic Energy=<KE>  Potential Energy=<PE>  Total Energy=<KE+PE>  Temperature (reduced)=<T>
At the end of each run:
textTotal wall time: <seconds>
Threads used: <N>   (parallel versions only)
9. PHYSICAL MODEL

Force Model: Lennard-Jones style force between particle pairs:
F = 24 * epsilon * (2*r^-12 - r^-6) / r^2
Integration: Euler integration with fixed time step DT = 0.0005
Boundaries: Reflective (particles bounce off walls)
Energy Tracking: Kinetic, Potential, Total Energy, and reduced Temperature (KE / N)
Initialization: Grid-based positions + random velocities (Mersenne Twister, seed = 42)

10. RACE CONDITION SOLUTION
Newton’s third law updates two particles per pair interaction.
Direct shared writes would cause data races.
Solution used in parallel_md.cpp and parallel_md_cutoff.cpp:

Each OpenMP thread writes forces into its own private buffer
After the parallel loop, private buffers are summed into the main force array
No locks, no atomics, no race conditions
