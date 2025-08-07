# Porting MATLAB to Python

Porting MATLAB code to Python offers access to open-source tools, improved integration with modern software stacks, and greater flexibility. Python's scientific ecosystem—NumPy, SciPy, Matplotlib, and others—provides powerful alternatives to MATLAB's built-in functions.

This guide outlines key considerations and provides examples using `pytest` to validate the correctness of translated code.

## Key Differences to Consider

- **Indexing**: MATLAB uses 1-based indexing; Python uses 0-based.
- **Array Operations**: MATLAB operations are often vectorized; Python requires explicit broadcasting or use of NumPy.
- **Plotting**: MATLAB has built-in plotting; Python uses libraries like Matplotlib or Seaborn.

## Modules

### Installing modules:

In shell type `pip install <package>`

### Key Modules you may want to use:
- **Xarray** Access datasets 
- **NumPy** Core matrix operations
- **SciPy** Replicate built in MATLAB functions
- **Matplotlib** Display plot, figure, etc.