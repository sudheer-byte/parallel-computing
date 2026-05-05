Parallel Molecular Dynamics Particle Simulation Using OpenMP
Student Name: Sudheer Sunkara
Project Type: Individual Project
Course: Parallel Programming

------------------------------------------------------------
Project Overview
------------------------------------------------------------

This project implements a simplified 2D molecular dynamics particle simulation.
Particles move inside a bounded box and interact using a Lennard-Jones style force.

The goal of the project is to compare three implementations:

1. Serial molecular dynamics baseline
2. OpenMP parallel molecular dynamics
3. OpenMP molecular dynamics with cell-list cutoff optimization

The main computational bottleneck is the force calculation between particle pairs.
Because many particle-pair force calculations can be done independently, this project
is suitable for parallel programming.

------------------------------------------------------------
Files Included
------------------------------------------------------------

serial_md.cpp
    Baseline serial molecular dynamics simulation.

parallel_md.cpp
    OpenMP parallel version.
    Uses thread-private force buffers to avoid race conditions.

parallel_md_cutoff.cpp
    OpenMP optimized version using a cell-list cutoff method.
    This reduces unnecessary force calculations by checking only nearby particles.

report.pdf or report.docx
    Final project report with explanation, results, tables, graphs, and analysis.

graph1_serial_runtime_vs_particles.png
    Serial runtime vs number of particles.

graph2_openmp_runtime_vs_threads.png
    OpenMP runtime vs number of threads.

graph3_cell_list_runtime_vs_threads.png
    Cell-list cutoff runtime vs number of threads.

graph4_best_runtime_comparison.png
    Best runtime comparison between serial, OpenMP, and cell-list versions.

------------------------------------------------------------
Compilation Commands
------------------------------------------------------------

Run these commands inside the project folder:

g++ -O3 serial_md.cpp -o serial_md
g++ -O3 -fopenmp parallel_md.cpp -o parallel_md
g++ -O3 -fopenmp parallel_md_cutoff.cpp -o parallel_md_cutoff

The -O3 flag enables compiler optimization.
The -fopenmp flag enables OpenMP support for the parallel programs.

------------------------------------------------------------
Command Line Arguments
------------------------------------------------------------

Serial version:

./serial_md <number_of_particles> <steps>

OpenMP parallel version:

./parallel_md <number_of_particles> <steps> <threads>

Cell-list cutoff version:

./parallel_md_cutoff <number_of_particles> <steps> <threads>

Example:

./parallel_md 1000 500 4

This means:
    1000 particles
    500 simulation steps
    4 OpenMP threads

------------------------------------------------------------
Commands Used for Report Data
------------------------------------------------------------

The following commands were used to collect the stable performance data shown in the report.

------------------------------------------------------------
1. Serial Particle Scaling Data
------------------------------------------------------------

These runs were used to measure how the serial runtime changes as the number of particles increases.

./serial_md 200 500
./serial_md 500 500
./serial_md 1000 500
./serial_md 2000 500

These runs produced the serial particle-scaling table:

Particles    Steps    Serial Time (seconds)
200          500      0.071945
500          500      0.406064
1000         500      1.64871
2000         500      6.49316

------------------------------------------------------------
2. OpenMP Thread Scaling Data
------------------------------------------------------------

These runs were used to measure how the OpenMP version scales with different thread counts.

./parallel_md 1000 500 1
./parallel_md 1000 500 2
./parallel_md 1000 500 4
./parallel_md 1000 500 8
./parallel_md 1000 500 16

These runs produced the OpenMP thread-scaling table:

Threads    Time (seconds)    Speedup vs 1 Thread
1          1.63353           1.00
2          0.933859          1.75
4          0.559991          2.92
8          0.375281          4.35
16         0.441211          3.70

------------------------------------------------------------
3. Cell-List Cutoff Thread Scaling Data
------------------------------------------------------------

These runs were used to measure how the optimized cell-list cutoff version scales with threads.

./parallel_md_cutoff 1000 500 1
./parallel_md_cutoff 1000 500 2
./parallel_md_cutoff 1000 500 4
./parallel_md_cutoff 1000 500 8
./parallel_md_cutoff 1000 500 16

These runs produced the cell-list cutoff table:

Threads    Time (seconds)    Speedup vs 1 Thread
1          0.160692          1.00
2          0.108000          1.49
4          0.108337          1.48
8          0.117653          1.37
16         0.251378          0.64

------------------------------------------------------------
4. Best Runtime Comparison
------------------------------------------------------------

The best runtime from each implementation was used for the final comparison.

Version              Best Time (seconds)
Serial               1.64871
OpenMP Parallel      0.375281
Cell-List Cutoff     0.108000

------------------------------------------------------------
Additional Quick Correctness Check
------------------------------------------------------------

I also used these smaller commands to quickly compare serial and parallel output values:

./serial_md 1000 100
./parallel_md 1000 100 4

These runs helped confirm that the serial and OpenMP versions produced matching
energy and temperature values for the same particle count and number of steps.

------------------------------------------------------------
Particle Scaling with Parallel and Cutoff Versions
------------------------------------------------------------

These commands were also tested to compare smaller particle sizes using 4 threads:

./parallel_md 200 500 4
./parallel_md 500 500 4

./parallel_md_cutoff 200 500 4
./parallel_md_cutoff 500 500 4

These are useful for checking that the parallel and cutoff versions run correctly
for smaller particle counts.

------------------------------------------------------------
Stress Tests Not Used in Main Graphs
------------------------------------------------------------

The following larger tests were attempted:

./parallel_md_cutoff 2000 500 4
./parallel_md_cutoff 5000 200 4
./parallel_md_cutoff 10000 100 4

The 2000-particle cutoff run was usable, but the 5000 and 10000 particle tests became
numerically unstable with the current timestep and density. Because of that, I did not
use the 5000 and 10000 particle runs in the main report graphs.

This is mentioned in the report as a limitation.

------------------------------------------------------------
Important Mathematical Formulas
------------------------------------------------------------

1. Speedup

Speedup measures how many times faster a version is compared to a baseline.

Speedup = Baseline Time / Parallel Time

For thread scaling, I used the 1-thread runtime as the baseline:

Speedup vs 1 Thread = Time with 1 Thread / Time with N Threads

Example:

OpenMP 8-thread speedup:

Speedup = 1.63353 / 0.375281 = 4.35

This means the 8-thread OpenMP version was about 4.35 times faster than
the 1-thread OpenMP version.

------------------------------------------------------------

2. Kinetic Energy

Kinetic energy measures the motion of particles.

KE = 1/2 * m * v^2

In this simplified simulation, mass is assumed to be 1.

For 2D velocity:

v^2 = vx^2 + vy^2

So the code calculates:

KE = 1/2 * (vx^2 + vy^2)

for each particle and sums it over all particles.

------------------------------------------------------------

3. Potential Energy

Potential energy comes from Lennard-Jones particle interactions.

The simplified Lennard-Jones potential energy form is:

PE = 4 * epsilon * [(1/r)^12 - (1/r)^6]

In the code, this is computed using inverse squared distance for efficiency.

------------------------------------------------------------

4. Total Energy

Total energy is the sum of kinetic energy and potential energy.

Total Energy = Kinetic Energy + Potential Energy

The output prints total energy to help check simulation behavior.

------------------------------------------------------------

5. Reduced Temperature

Temperature is reported in reduced units.

Temperature (reduced) = Kinetic Energy / Number of Particles

This is not Celsius or Kelvin.
It is a unitless value used to track the relative motion level of the particles.

------------------------------------------------------------

6. Simulation Time

Each simulation step advances the system by one small timestep.

Total Simulated Time = Number of Steps * DT

In this project:

DT = 0.0005

For 500 steps:

Total Simulated Time = 500 * 0.0005 = 0.25 reduced time units

------------------------------------------------------------
Output Explanation
------------------------------------------------------------

Each program prints values such as:

Kinetic Energy:
    Energy from particle motion.

Potential Energy:
    Energy from particle interactions.

Total Energy:
    Kinetic Energy + Potential Energy.

Temperature (reduced):
    Unitless temperature based on kinetic energy.
    It is not Celsius or Kelvin.

Total wall time:
    Runtime of the simulation.

Threads used:
    Number of OpenMP threads used in the run.

------------------------------------------------------------
Implementation Notes
------------------------------------------------------------

1. Grid Initialization

Particles are placed on a grid instead of fully random positions.
This avoids particles starting too close together and helps keep the simulation stable.

2. Fixed Random Seed

The program uses a fixed random seed for velocity initialization.
This makes the runs repeatable and fair for comparison.

3. Lennard-Jones Force

Particles interact using a Lennard-Jones style force.
This creates repulsion at short distances and weaker interaction at longer distances.

4. Newton's Third Law

Each particle pair is calculated once.
Equal and opposite forces are applied to both particles.

5. Thread-Private Force Buffers

The OpenMP version avoids race conditions by giving each thread its own force arrays.
After the force loop finishes, the private arrays are combined into the final force arrays.

6. Dynamic Scheduling

The OpenMP force loop uses dynamic scheduling because some loop iterations have more
particle-pair work than others.

7. Cell-List Cutoff

The cutoff version divides the simulation box into cells.
Only particles in the same or nearby cells are checked.
This reduces unnecessary comparisons and improves runtime.

------------------------------------------------------------
Final Notes
------------------------------------------------------------

The main report results are based on stable runs with 1000 particles and 500 steps
for thread scaling.

The project is a simplified 2D molecular dynamics simulation.
It is not a full real-world 3D molecular dynamics package, but it demonstrates the
main parallel programming idea clearly: pairwise force calculation is expensive, and
parallelism plus algorithmic optimization can reduce runtime.
