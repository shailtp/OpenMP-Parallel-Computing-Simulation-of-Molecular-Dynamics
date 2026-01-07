# Molecular Dynamics Simulation - Performance Testing

This repository contains both sequential and parallel (OpenMP) implementations of a molecular dynamics simulation. This document summarizes the performance testing and results.

## Quick Start

### Running the Sequential Version
```bash
./pdc_sequential [ND] [NP] [STEP_NUM] [DT]
```

### Running the Parallel Version
```bash
./pdc_parallel [ND] [NP] [STEP_NUM] [DT]
```

**Parameters:**
- `ND`: Spatial dimension (2 or 3)
- `NP`: Number of particles
- `STEP_NUM`: Number of time steps
- `DT`: Time step size (e.g., 0.0001)

**Example:**
```bash
./pdc_sequential 3 500 100 0.0001
./pdc_parallel 3 500 100 0.0001
```

## Test Results

All tests were run on a Mac with 8 CPU cores. The parallel version uses OpenMP with 8 threads.

### Test 1: Small Scale Simulation
**Configuration:** 3D, 500 particles, 100 time steps, dt = 0.0001

| Version   | Execution Time | Speedup |
|-----------|----------------|---------|
| Sequential| 0.771 seconds  | 1.0x    |
| Parallel  | 0.243 seconds  | **3.17x** |

The parallel version completed in about 31% of the sequential time. Both versions produced identical results:
- Initial potential energy: 124406.486423
- Final energy error: ~1.5e-11 (excellent energy conservation)

### Test 2: Large Scale Simulation
**Configuration:** 3D, 1000 particles, 400 time steps, dt = 0.0001

| Version   | Execution Time | CPU Usage | Speedup |
|-----------|----------------|-----------|---------|
| Sequential| 12.46 seconds  | 99% (1 core) | 1.0x |
| Parallel  | 3.91 seconds   | 507% (8 cores) | **3.19x** |

For larger simulations, the speedup remains consistent at around 3.2x. The parallel version effectively utilizes all 8 cores, as shown by the 507% CPU usage (multiple cores working simultaneously).

## Performance Analysis

### What We Observed

1. **Consistent Speedup**: Both small and large tests show approximately 3.2x speedup on an 8-core system. This is good performance, though not perfect linear scaling (which would be 8x).

2. **Parallel Efficiency**: The efficiency is around 40% (3.2x speedup / 8 cores). This is typical for parallel programs due to:
   - Overhead from thread management
   - Memory bandwidth limitations
   - Load balancing issues
   - Parts of the code that can't be parallelized

3. **Scaling Behavior**: The O(N²) force computation loop benefits significantly from parallelization. Each particle's force calculation is independent, making it ideal for parallel execution.

4. **Energy Conservation**: Both versions maintain excellent energy conservation with relative errors on the order of 10^-11, confirming that parallelization doesn't introduce numerical errors.

### Why Not 8x Speedup?

Perfect linear scaling (8x speedup on 8 cores) is rarely achieved in practice. The 3.2x speedup we're seeing is reasonable because:

- The `compute()` function has nested loops where the outer loop is parallelized, but there's still sequential work in initialization and I/O
- Memory access patterns can create contention
- OpenMP overhead for thread creation and synchronization
- Some parts of the code (like `update()`) are parallelized but may have less work per iteration

## Correctness Verification

Both implementations produce numerically identical results:
- Initial energies match exactly
- Energy conservation errors are within floating-point precision
- The physics simulation behaves correctly in both versions

The parallel version uses OpenMP reduction clauses to safely accumulate energy values across threads, ensuring correctness.

## Compilation Notes

The sequential version compiles with standard gcc:
```bash
gcc -o pdc_sequential pdc_sequential.c -lm
```

The parallel version requires OpenMP support:
```bash
gcc -fopenmp -o pdc_parallel pdc_parallel.c -lm
```

**Note:** On macOS with clang, you may need to install libomp or use a different compiler that supports OpenMP.

## Recommendations

1. **For small simulations** (< 500 particles): The sequential version might be sufficient due to OpenMP overhead.

2. **For larger simulations** (> 1000 particles): The parallel version shows clear benefits and is recommended.

3. **Thread count tuning**: You can control the number of threads using:
   ```bash
   export OMP_NUM_THREADS=4
   ./pdc_parallel 3 1000 400 0.0001
   ```
   Experiment with different thread counts to find optimal performance for your system.

4. **Scaling test**: Try running with 2000+ particles to see if the speedup improves further.

## Conclusion

The OpenMP parallelization provides a solid 3.2x speedup on an 8-core system while maintaining numerical accuracy. The implementation successfully parallelizes the computationally expensive force calculations, which dominate the runtime for larger simulations.

