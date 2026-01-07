# Molecular Dynamics Simulation with OpenMP

A C implementation of a molecular dynamics simulation comparing sequential and parallel (OpenMP) approaches. This project demonstrates how parallel computing can accelerate computationally intensive scientific simulations.

## What This Does

This simulates the motion of particles in 2D or 3D space using molecular dynamics. Each particle interacts with all other particles through a force field, and we track how the system evolves over time. The simulation uses a velocity Verlet integration scheme to update particle positions and velocities at each time step.

## The Physics

The particles interact through a sinusoidal potential energy function:

```
V(r) = sin²(min(r, π/2))
```

This creates a smooth, bounded interaction that saturates at π/2. The force between particles is the negative gradient of this potential. At each time step, the code:

1. Computes forces between all particle pairs (O(N²) complexity)
2. Calculates potential and kinetic energies
3. Updates positions and velocities using the velocity Verlet algorithm

Energy conservation is tracked throughout the simulation as a sanity check - the total energy (potential + kinetic) should remain constant.

## Project Structure

- `pdc_sequential.c` - Single-threaded implementation
- `pdc_parallel.c` - OpenMP parallelized version
- Compiled executables: `pdc_sequential`, `pdc_parallel`

## Building

**Sequential version:**
```bash
gcc -o pdc_sequential pdc_sequential.c -lm
```

**Parallel version (requires OpenMP):**
```bash
gcc -fopenmp -o pdc_parallel pdc_parallel.c -lm
```

On macOS, you may need to install libomp or use a compiler with OpenMP support.

## Running

Both programs take the same command-line arguments:

```bash
./pdc_sequential [ND] [NP] [STEP_NUM] [DT]
./pdc_parallel [ND] [NP] [STEP_NUM] [DT]
```

- `ND`: Spatial dimension (2 or 3)
- `NP`: Number of particles
- `STEP_NUM`: Number of time steps to simulate
- `DT`: Time step size (e.g., 0.0001)

**Example:**
```bash
./pdc_parallel 3 1000 400 0.0001
```

This runs a 3D simulation with 1000 particles for 400 time steps.

## Parallelization Strategy

The parallel version uses OpenMP to speed up the computationally expensive parts:

1. **Force computation loop**: The outer loop over particles is parallelized. Each thread handles a subset of particles, computing their interactions with all other particles. OpenMP reduction clauses safely accumulate the energy values across threads.

2. **Position/velocity updates**: The update loop is also parallelized since each particle's update is independent.

The key insight is that while computing forces is O(N²), each particle's force calculation is independent, making it perfect for parallel execution. The parallel version distributes particles across available CPU cores.

## Why This Matters

Molecular dynamics simulations are used extensively in chemistry, physics, and materials science. The force computation scales quadratically with particle count, so parallelization becomes essential for realistic system sizes. This project shows how a straightforward parallelization can provide significant speedup without changing the underlying physics.

## Performance

See `TESTING.md` for detailed performance comparisons and benchmarks. The parallel version typically achieves 3-4x speedup on multi-core systems while maintaining numerical accuracy.

## Notes

- Both versions produce identical results (within floating-point precision)
- Energy conservation errors are typically on the order of 10^-11
- The parallel version automatically detects available CPU cores
- You can control thread count with `export OMP_NUM_THREADS=N`
