(content:intro-python)=
# Intro to Python for Climate Science

[Python](https://www.python.org) provides a powerful yet accessible toolkit for analyzing vast datasets, performing complex modeling, and visualizing climate patterns. In climate science, data often involves multiple dimensions—like time, latitude, longitude, and depth—making Python’s specialized libraries invaluable for both efficiency and depth of analysis.

In this introductory post, we’ll walk through the core Python libraries frequently used in climate science: xarray, dask, matplotlib, and pandas. We’ll also touch on popular development environments, Jupyter Notebooks and VS Code, which support interactive, reproducible research workflows ideal for scientific research.

## Why Python is Ideal for Climate Science

Python’s simplicity and rich ecosystem make it an ideal choice for climate science research:
- **Multi-Dimensional Data Handling**: Climate data often has multiple dimensions, such as time, location, and variables (e.g., temperature, precipitation). Python’s libraries like xarray and dask can handle these complex data structures effortlessly.
- **Robust Data Analysis and Visualization Tools**: Libraries like pandas for data manipulation and matplotlib for visualization make it easy to process and visualize climate trends.
- **Community and Open Source**: Python has an active scientific community with extensive documentation and resources, allowing researchers to adopt reproducible, open-source methods.

Whether you’re analyzing atmospheric data, studying ocean temperature patterns, or modeling climate simulations, Python has the tools needed to support robust and scalable research.

## Essential Libraries for Climate Science

Here’s a closer look at the main libraries used in climate science research.

### Xarray: Multi-Dimensional Arrays for Climate Data

[Xarray](https://docs.xarray.dev/en/stable/) provides data structures that make it easy to work with multi-dimensional arrays, such as those found in NetCDF files commonly used in climate science. With xarray, you can work with data that has labeled dimensions, making it simpler to keep track of multiple variables, units, and dimensions.
- **Handles NetCDF and GRIB Formats**: Designed for climate science data formats.
- **Metadata Support**: Keeps track of labels for dimensions and coordinates.
- **Integration with Dask**: Easily loads large datasets by leveraging Dask for parallel processing.

Example:
```python
import xarray as xr

# Load a NetCDF climate dataset
ds = xr.open_mfdataset("climate_data.nc")
print(ds)
```

### Dask: Scalable Data Processing for Large Climate Datasets

[Dask](https://www.dask.org) extends Python’s data processing capabilities, allowing for parallel computation on large datasets. When working with datasets too big for memory, Dask processes data in chunks and manages parallel operations, allowing you to analyze data on a laptop, cloud server, or HPC.
- **Out-of-Core Computation**: Works with datasets that don’t fit in memory.
- **Parallel Processing**: Leverages multi-core processors or distributed systems.
- **Works Seamlessly with Xarray**: Enables efficient loading and computation for xarray DataArrays.

Example:
```python
from dask.distributed import Client
c = Client()

# Load dataset with Dask for lazy evaluation
ds = xr.open_mfdataset("climate_data.nc", chunks={"time": 100}, Parallel=True)
```

### Pandas: Data Manipulation and Time Series Analysis

[Pandas](https://pandas.pydata.org) is a powerful library for handling tabular and time series data. While not optimized for multi-dimensional data, pandas is still useful for climate science when handling metadata, summaries, and specific data manipulations.
- **Data Wrangling**: Effective for cleaning, merging, and reshaping datasets.
- **Time Series Support**: Built-in functionality for time series data.
- **Works with Xarray**: Easily converts between xarray and pandas structures.

Example:
```python
import pandas as pd

# Load time series climate data
df = pd.read_csv("temperature_data.csv")
print(df.head())
```

### Matplotlib: Visualizing Climate Data

[matplotlib](https://matplotlib.org) is the primary library for visualizing climate data in Python, enabling high-quality, customizable visualizations, from simple line plots to complex spatial maps.

- **Wide Range of Plot Types**: Line plots, histograms, 2D heatmaps, and more.
- **Highly Customizable**: Perfect for publication-quality figures.
- **Integrates with Xarray and Pandas**: Visualizes data directly from data structures.

Example:
```python
import matplotlib.pyplot as plt

# Plot a temperature time series
plt.plot(df["date"], df["temperature"])
plt.xlabel("Date")
plt.ylabel("Temperature")
plt.title("Temperature Over Time")
plt.show()
```

## Development Environments: Jupyter Notebooks and VS Code

**Jupyter Notebooks**

[Jupyter Notebooks](https://jupyter.org) are highly popular in scientific computing because they allow researchers to combine code, results, and documentation in a single environment. With their interactive nature, Jupyter Notebooks are ideal for data exploration and visualization.
- **Interactive Cells**: Run code cell-by-cell, seeing results instantly.
- **Markdown Support**: Add notes, equations, and documentation.
- **Rich Visualization**: View plots and outputs inline, great for iterative analysis.

**VS Code**
[Visual Studio Code (VS Code)](https://code.visualstudio.com) is a versatile code editor that supports both traditional coding and interactive notebooks, making it a great alternative to Jupyter Notebooks for climate science.
- **Notebook Support**: The Jupyter extension allows you to run notebooks directly in VS Code.
- **Advanced Editing and Debugging**: Ideal for larger projects that need a more comprehensive IDE.
- **Extensions for Data Science**: Customize your environment with extensions for Python, Dask, and more.

Both environments offer unique benefits, allowing researchers to choose based on preference and project requirements.

## Conda Python Environments - analysis3

An [Anaconda](https://anaconda.org/anaconda/conda) is maintained at NCI (created and currently maintaned by the CLEX CMS team), with a wide variety of climate and weather related libraries.

You can find the most recent list of libraries at the [github repository](https://github.com/coecms/conda-envs/blob/analysis3/environment.yml), or run conda list with an environment loaded.

To use any of the conda environments:

- Request access to hh5 (do once)
- You must first run (to do at each session, a qsub job is a new session):
```module use /g/data/hh5/public/modules```


If you need to use the conda environment in a PBS job you will need to add the [hh5 project](https://my.nci.org.au/mancini/login?next=/mancini/project/hh5) to your storage flags, e.g. `#PBS -l storage=gdata/hh5`. These conda environments will work on gadi login/compute nodes, and in the [ARE cloud environment](https://are-auth.nci.org.au). 

**Stable Environment**: 
The stable environment is updated once a quarter, around when NCI do their quarterly maintenance of Gadi. Otherwise everything in the environment stays fixed, packages arent intsalled or updated unless something is very broken.

`module load conda/analysis3`

**Unstable Environment**: 
The unstable environment gets updated more often, new packages and updated packages are installed here. If you ask for a new package it will be installed here.

`module load conda/analysis3-unstable`

When the quarterly update happens the unstable environment becomes the new stable environment.

**ARE**
To use CLEX Conda in [ARE cloud environment](https://are-auth.nci.org.au), start Jupyter with the advanced options:

```Module Directories: /g/data/hh5/public/modules```

```Modules: conda/analysis3```