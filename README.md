=============================================================================
  Parallel Molecular Dynamics Particle Simulation Using OpenMP
=============================================================================
  Student      : Sudheer Sunkara
  Course       : Concurrent and Parallel Programming (Spring 2026)
  Project Type : Individual Project
  Submission   : Final Project
=============================================================================

-----------------------------------------------------------------------------
1. PROJECT OVERVIEW
-----------------------------------------------------------------------------

This project implements a two-dimensional Molecular Dynamics (MD) particle
simulator using the Lennard-Jones interaction model. The goal is to simulate
how particles move and interact inside a bounded box, and to study how
parallel programming improves simulation performance.

Three implementations are provided:

  1. serial_md.cpp
     - Baseline serial implementation
     - Uses Newton's third law to compute pairwise forces efficiently
     - All force, velocity, and position updates run on a single thread
     - Used as the performance baseline for speedup calculations

  2. parallel_md.cpp
     - OpenMP parallel implementation
     - Parallelizes the force calculation using #pragma omp parallel
     - Uses thread-private force buffers to avoid data races
     - Uses dynamic scheduling to balance uneven loop workloads
     - Particle update step parallelized with static scheduling
     - Best performance at 8 threads (4.39x speedup vs serial)

  3. parallel_md_cutoff.cpp
     - OpenMP parallel + cell-list cutoff optimization
     - Divides the simulation box into a grid of cells (8x8)
     - Only checks particle pairs in the same or neighboring cells
     - Ignores pairs beyond the cutoff radius (CUTOFF = 10.0)
     - Reduces algorithmic complexity from O(N^2) to approximately O(N)
     - Best overall performance: 15.27x speedup vs serial at 2 threads

-----------------------------------------------------------------------------
2. FILES INCLUDED
-----------------------------------------------------------------------------

  serial_md.cpp                                         Serial implementation
  parallel_md.cpp                                       OpenMP parallel version
  parallel_md_cutoff.cpp                                OpenMP + cell-list version
  Parallel_Molecular_Dynamics_Particle_Simulation.docx  Final report
  README.txt                                            This file

-----------------------------------------------------------------------------
3. SIMULATION PARAMETERS
-----------------------------------------------------------------------------

  Parameter        Value      Description
  --------------------------------------------------------------------
  BOX_SIZE         80.0       Side length of the 2D simulation box
  DT               0.0005     Time step size
  EPSILON          1.0        Lennard-Jones energy parameter
  MIN_R2           0.0001     Minimum r^2 to prevent division by zero
  CUTOFF           10.0       Force cutoff radius (cutoff version only)
  Particle Mass    1.0        Implicit, reduced units
  Random Seed      42         Fixed for reproducibility across versions

-----------------------------------------------------------------------------
4. SYSTEM REQUIREMENTS
-----------------------------------------------------------------------------

  - Linux or WSL (Windows Subsystem for Linux)
  - g++ compiler with C++11 or later
  - OpenMP support (included with most g++ installations)
  - Recommended: 4 to 8 CPU cores for best parallel performance

-----------------------------------------------------------------------------
5. COMPILE COMMANDS
-----------------------------------------------------------------------------

  Serial:
    g++ -O3 -o serial_md serial_md.cpp

  Parallel (OpenMP):
    g++ -O3 -fopenmp -o parallel_md parallel_md.cpp

  Parallel with cell-list cutoff:
    g++ -O3 -fopenmp -o parallel_md_cutoff parallel_md_cutoff.cpp

  Note: -O3 enables compiler optimizations. All three versions use
  the same flag to keep the comparison fair.

-----------------------------------------------------------------------------
6. RUN COMMANDS
-----------------------------------------------------------------------------

  Serial:
    ./serial_md <num_particles> <num_steps>

  Parallel:
    ./parallel_md <num_particles> <num_steps> <num_threads>

  Parallel with cutoff:
    ./parallel_md_cutoff <num_particles> <num_steps> <num_threads>

  Arguments:
    num_particles   Number of particles (e.g. 200, 500, 1000, 2000)
    num_steps       Number of simulation time steps (e.g. 100, 500)
    num_threads     Number of OpenMP threads (e.g. 1, 2, 4, 8, 16)

  Default values if no arguments given:
    num_particles = 1000
    num_steps     = 100
    num_threads   = 4

-----------------------------------------------------------------------------
7. EXAMPLE RUNS
-----------------------------------------------------------------------------

  # Serial baseline
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

  # Parallel vary particles (4 threads)
  ./parallel_md 200 500 4
  ./parallel_md 500 500 4
  ./parallel_md 1000 500 4
  ./parallel_md 2000 500 4

-----------------------------------------------------------------------------
8. OUTPUT FORMAT
-----------------------------------------------------------------------------

  Every 10 simulation steps, each program prints:

    Step <N>  Kinetic Energy=<KE>  Potential Energy=<PE>
              Total Energy=<KE+PE>  Temperature (reduced)=<T>

  At the end of each run:

    Total wall time: <seconds>
    Threads used: <N>      (parallel versions only)

  Example (N=1000, steps=500, threads=8):

    Particles initialized with stable grid positions and random velocities.
    === Parallel MD Simulation (OpenMP) ===
    Particles: 1000, Steps: 500, Threads: 8
    Step 0   Kinetic Energy=730.818  Potential Energy=-62.515
             Total Energy=668.303   Temperature (reduced)=0.730818
    ...
    Total wall time: 0.375281 seconds
    Threads used: 8

-----------------------------------------------------------------------------
9. PHYSICAL MODEL
-----------------------------------------------------------------------------

  Force Model:
    Lennard-Jones style force between particle pairs.
    F = 24 * epsilon * (2*r^-12 - r^-6) / r^2

  Integration:
    Euler integration with fixed time step DT = 0.0005.
    Reflective boundary conditions (particles bounce off walls).

  Energy Tracking (printed every 10 steps):
    Kinetic Energy  = 0.5 * mass * v^2   (summed over all particles)
    Potential Energy= 4 * epsilon * (r^-12 - r^-6)  (summed over pairs)
    Total Energy    = KE + PE
    Temperature     = KE / N  (reduced units, not Kelvin or Celsius)

  Initialization:
    Grid-based positions to prevent particle overlap at startup.
    Random velocities via Mersenne Twister (seed = 42).
    Same seed used across all three versions for fair comparison.

-----------------------------------------------------------------------------
10. RACE CONDITION SOLUTION
-----------------------------------------------------------------------------

  Newton's third law updates TWO particles per pair interaction.
  Direct shared writes would cause data races in parallel code.

  Solution used in parallel_md.cpp and parallel_md_cutoff.cpp:
    - Each OpenMP thread writes forces into its own private buffer
    - After the parallel loop, private buffers are summed into the
      main force array using a second parallel reduction loop
    - No locks, no atomics, no race conditions

-----------------------------------------------------------------------------
11. PERFORMANCE RESULTS (N=1000, 500 steps)
-----------------------------------------------------------------------------

  Version                   Threads   Time (s)    Speedup vs Serial
  ------------------------------------------------------------------
  Serial Baseline           -         1.64871     1.00x
  OpenMP Parallel           1         1.63353     1.01x
  OpenMP Parallel           2         0.93386     1.77x
  OpenMP Parallel           4         0.55999     2.95x
  OpenMP Parallel           8         0.37528     4.39x
  OpenMP Parallel           16        0.44121     3.74x
  Cell-List Cutoff          1         0.16069     10.26x
  Cell-List Cutoff          2         0.10800     15.27x  <- Best
  Cell-List Cutoff          4         0.10834     15.22x
  Cell-List Cutoff          8         0.11765     14.01x
  Cell-List Cutoff          16        0.25138     6.56x

  Key findings:
    - OpenMP best at 8 threads (4.39x speedup)
    - Cell-list best at 2 threads (15.27x speedup)
    - 16 threads slower than 8 in both versions (thread overhead)
    - Algorithmic optimization (cell-list) outperforms raw parallelism

-----------------------------------------------------------------------------
12. KNOWN LIMITATIONS
-----------------------------------------------------------------------------

  - 2D simulation only (not 3D)
  - Euler integration (not Velocity-Verlet)
  - No periodic boundary conditions (reflective walls used)
  - Temperature is in reduced units (not physical Kelvin)
  - Large particle counts (N >= 5000) become numerically unstable
    with the current timestep and box density settings
  - Cutoff version uses a fixed 8x8 cell grid regardless of N

-----------------------------------------------------------------------------
13. NOTES ON NUMERICAL STABILITY
-----------------------------------------------------------------------------

  Stable configurations tested: N = 200, 500, 1000, 2000
  Unstable configurations (do not use): N = 5000, N = 10000

  If particles are too densely packed relative to the box size and
  time step, forces become extremely large and the simulation diverges.
  For larger N, reduce DT or increase BOX_SIZE before running.

=============================================================================
