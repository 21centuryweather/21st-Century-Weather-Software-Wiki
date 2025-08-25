# Porting MATLAB to Python

Porting MATLAB code to Python offers access to open-source tools, improved integration with modern software stacks, and greater flexibility. Python's scientific ecosystem—NumPy, SciPy, Matplotlib, and others—provides powerful alternatives to MATLAB's built-in functions.

This guide outlines key considerations and provides examples using `pytest` to validate the correctness of translated code.

## Key Differences to Consider

- **Indexing**: MATLAB uses 1-based indexing; Python uses 0-based.
- **Array Operations**: MATLAB operations are often vectorized; Python requires explicit broadcasting or use of NumPy.
- **Plotting**: MATLAB has built-in plotting; Python uses libraries like Matplotlib or Seaborn.

## Modules

### Key Modules you may want to use:
- **Xarray** Access netCDF datasets 
- **NumPy** Core matrix operations
- **SciPy** Replicate built in MATLAB functions
- **Matplotlib** Display plot, figure, etc.

## Case study
In this example we will port some Matlab code used to identify atmospheric rivers into python.

The original code can be found here : [GitHub link](https://github.com/Pirate-rr/REID_ARalgorithm)

The code can be broken into several steps:
1) Loading the netCDF data containing 'Integrated Water Vapour Transport' (IVT). Here is an example IVT data computed using ERA5 re-analysis : [UCAR Data catalogue](https://rda.ucar.edu/datasets/d651018/dataaccess/)

2) Masking the input array for given threshold value of IVT.

3) Identifying features within the array using image recognition algorithms.

4) Apply various constraints to the identified features.

5) Write the desired features to a lat/lon grid and save to netCDF.

Let's proceed through each step of this process in MATLAB and display the equivalent code in python.

### Loading netCDF data.

MATLAB uses its `ncread` library to load netCDF data from disk. Here is some example code
```
filename = '/<Path>/ERA5_hourlyIVT_19880201.nc';

ivt = ncread(filename,'ivt');
ERA5_lat = ncread(filename,'lat');
ERA5_lon = ncread(filename,'lon');
ERA5_time = ncread(filename,'time');
```
Note this shape of size of `ivt`:
```
>> size(ivt)

ans =

        1440         721         672
```
Loading a single slice of 2-D data for the first timestep can be done using
```
dataset=ivt(:,:,1)
```
In this example using data from February 1 1988, the first entries of `dataset` are equal to
```
>> dataset(1:5,1:5)

ans =

    1.4834   10.4450   11.2884   12.7587   14.3300
    1.4834   10.4450   11.2263   12.7285   14.2727
    1.4834   10.3842   11.2892   12.6691   14.2165
    1.4834   10.3842   11.2288   12.7326   14.1614
    1.4834   10.4473   11.2288   12.6747   14.2259
```

Let's use the python module `xarray` to do the same thing. We can run this from an 'interactive python' session (or `ipython`) to create similar look and feel to the MATLAB environment.
```python
$ ipython
In [1]: import xarray as xr

In [2]: from pathlib import Path

In [3]: DATA_FILE = Path('~/Downloads/ERA5_hourlyIVT_19880201.nc')

In [5]: dataset =xr.open_dataset(DATA_FILE)

In [6]: dataset
Out[6]: 
<xarray.Dataset> Size: 6GB
Dimensions:  (lon: 1440, lat: 721, time: 672)
Coordinates:
  * lon      (lon) float64 12kB -180.0 -179.8 -179.5 ... 179.2 179.5 179.8
  * lat      (lat) float64 6kB -90.0 -89.75 -89.5 -89.25 ... 89.5 89.75 90.0
  * time     (time) datetime64[ns] 5kB 1988-02-01 ... 1988-02-28T23:00:00
Data variables:
    ivt      (time, lat, lon) float64 6GB ...
```

To look at the values of this dataset, we must reference the data variable name `ivt`. But first, let's check the array shape.
```
In [49]: dataset.ivt.shape
Out[49]: (672, 721, 1440)
```
Note this are **NOT** the same as the MATLAB values! Python read the data array using time along the first dimension. MATLAB also uses a different row/column order when processing matrices to python. This may cause confusion when plotting the raw matrix values as they your x/y axes may be swapped. 

We can check that python is seeing the same data as MATLAB by using a `transpose` operator.

I can do that using the `numpy` library

```
In [39]: import numpy as np

In [40]: np.set_printoptions(precision=4) #To print floats with lower precision

In [41]: np.transpose(dataset.ivt.values)[0:5,0:5,0]
Out[41]: 
array([[ 1.4834, 10.445 , 11.2884, 12.7587, 14.33  ],
       [ 1.4834, 10.445 , 11.2263, 12.7285, 14.2727],
       [ 1.4834, 10.3842, 11.2892, 12.6691, 14.2165],
       [ 1.4834, 10.3842, 11.2288, 12.7326, 14.1614],
       [ 1.4834, 10.4473, 11.2288, 12.6747, 14.2259]])
```
Which matches the MATLAB outputs. We can also use the `numpy` shortcut `.T` 
```
In [55]: dataset.ivt[0].T[0:5,0:5].values
Out[55]: 
array([[ 1.4834, 10.445 , 11.2884, 12.7587, 14.33  ],
       [ 1.4834, 10.445 , 11.2263, 12.7285, 14.2727],
       [ 1.4834, 10.3842, 11.2892, 12.6691, 14.2165],
       [ 1.4834, 10.3842, 11.2288, 12.7326, 14.1614],
       [ 1.4834, 10.4473, 11.2288, 12.6747, 14.2259]])
```

### Masking the data.

The next task is to apply a user-input threshold value of IVT to the data. In MATLAB this is done using
```
