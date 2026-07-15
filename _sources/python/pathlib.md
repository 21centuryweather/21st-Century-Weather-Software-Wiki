# Using the Python pathlib module

When using Python, we often fall into the trap of specifying filepaths as a single string, e.g. 
```python
path='/some_folder_name/a_random_subdirectory_name/another_subdirectory/filename.txt'
```
While this may suffice when writing simple test scripts, such practices create several problems. 

1. Such strings are **platform dependent!** Windows machines use backslashes "\\" while Mac/Linux uses forwardslashes "/". So a user would have to edit every filepath string when using code across different platforms.

2. Even if you are migrating between common platforms, the filestructure could be completely different from one machine to the next. So data could be sitting on a Linux directory at `gadi` for example at:
```
/g/data/xx09/schmoopy/rainfall
```
and on a Mac directory at:
```
/Users/schmoopy/rainfall
```
Again, a user would have to edit every filepath throughout the code in order to run it.

Surely, you say, there must be a better way!?

Thankfully, there is.

## The pathlib module

The Python `pathlib` module treats filepaths not as strings, but as python **objects**. The eliminates cross-platform bugs and makes project configuration very clean and inherently portable.

The following code will produce the same pathname on any platform.
```python
from pathlib import Path
import os

home = os.environ['HOME']
dummy_path = Path(home) / "Documents" / "Projects"
print (dummy_path)
```
On `gadi`, this will create the `PosixPath` object
```python
/home/<id>/<user_id>/Documents/Projects
```
On a Mac, it will generate 
```python
/Users/<username/Documents/Projects
```
and on Windows
```python
C:\Users\<username>\Documents\Projects
```
Anytime your pass a filestring to the `Path` function, you will generate a `PosixPath` (on Linux/Mac) or `WindowsPath` object. e.g.
```python
>>> Path('/g/data/gb02/')
PosixPath('/g/data/gb02')
```
We can then use the '/' operator to join strings onto this Path object quickly and efficiently, e.g.
```python
>>> DATA = Path('/g/data/gb02/')
>>> file = DATA / 'file.nc'
>>> data = xr.open_dataarray (file)
# Or you can type
>>> data = xr.open_dataarray ( DATA / 'file.nc' )
```

## Dynamically defining repository locations

Using `pathlib` avoids hardcoding absolute paths (like "C:/Users/Name/Projects/data") and allows us to anchor your project dynamically relative to a configuration file or script execution point. We do this using the `__file__` attribute.

Let's say we are following best practise and using the Centre's project template as described [here](../gadi/project-template.md), and you clone the project in a directory `/Users/schmoopy/rainfall_project/`.

You could then create a configuration file `config.py` saved to `/Users/schmoopy/rainfall_project/config.py` with the following entries.

```python
# config.py (Located inside your repo at: my_project/config.py)
from pathlib import Path

# 1. Get the absolute path of this specific config file
CONFIG_PATH = Path(__file__).resolve()

# 2. Go up one level to hit the project root directory (i.e. where you clone the project repository)
PROJECT_ROOT = CONFIG_PATH.parents[0]

# 3. Define important subdirectories relative to the root
DATA_DIR = PROJECT_ROOT / "data"
DERIVED_DIR = DATA_DIR / "derived"
TEMP_DIR = DATA_DIR / "temp"
ANALYSIS_DIR = PROJECT_ROOT / "analysis"

# Ensure directories exist before your scripts try to use them
DATA_DIR.mkdir(parents=True, exist_ok=True)
OUTPUT_DIR.mkdir(parents=True, exist_ok=True)

print(f"Project Root is anchored at: {PROJECT_ROOT}")
```
So we have created python objects for every directory that exits in the project template. For all subsequent code written for the project, you would use these `pathlib` objects (i.e. `TEMP_DIR`,`ANALYSIS_DIR` etc.) instead of using text strings.

The values of these `pathlib` objects can be imported into any script via the command
```python
from config import *
```

:::{note}

You will need to set your `sys.path` inside any python script so it knows where to find `config.py` (unless you are working in the same directory)
:::

The use of configuration files is standard practise for python programming. See [here](./package-structure.md) for more discussion. In that example, a bash environment variable `ROOT` is used to define the location of the project directory instead of a `pathlib` `__file__` attribute.

Using a configuration file allows you to define all file locations relative to a central location.

### What about file paths outside of my project?

If you need to reference data on filesystems outside of your project directory, you can define them as a full path. Follow the example [here](https://github.com/21centuryweather/software_engineering_demos/blob/main/solar_example/config.py).

This still helps with portability. If you wanted to run the data analysis on another platform, you would just need to change a single filepath definition in `config.py`, rather than edit every textstring that points to a data file in your code base.

If your project grows to ingest data from multiple sources on multiple platforms, you can use **different configuration files!**. E.g. you could have a file called `config_gadi.py` which contains
```
HIMAWARI_SOLAR_DIR=Path('/g/data/rv74/satellite-products/arc/der/himawari-ahi/solar/')
```
and another file called `config_mac.py` which contains
```
HIMAWARI_SOLAR_DIR=Path('/Users/schmoopy/Downloads/himawari-ahi/solar')
```
You could commit both configuration files to the project repository, and then use [symbolic links](../gadi/project-template.md#organise-your-code) to create a link called `config.py` that points the relevant configuration file for your current platform.

## Searching, Globbing and Iterating files.

In addition to helping maintain portability, `pathlib` contains lots of inbuilt methods to help us search for files:

- `.glob()`: Searches the immediate directory.
- `.rglob()`: Searches recursively through all subdirectories (equivalent to `/*.ext`).

Both methods return a generator, meaning they are memory-efficient even when dealing with thousands of files.
```python
from pathlib import Path

# Assume we are looking inside our raw data directory
data_folder = Path("./data/raw")

# Example A: Iterate over only CSVs in the immediate directory
print("--- CSVs in Main Folder ---")
for csv_file in data_folder.glob("*.csv"):
    print(csv_file.name)  # .name yields just the filename (e.g., 'data.csv')

# Example B: Recursively find ALL JSON files across all subdirectories
print("\n--- JSONs in All Subfolders ---")
for json_file in data_folder.rglob("*.json"):
    print(json_file.relative_to(data_folder))  # Prints path relative to data_folder
```
Because we all the filenames we generate are python objects, we can working with the python `datetime` module to easily filter filenames according to specified date criteria. Suppose I have directory full of netCDF files. Each file contains a `<YYYYMMDD>` string and I want to find files valid between a given range of dates.

In this example, the filenames have the format `maq5_tasmax_<YYYYMMDD>_e01.nc`

```python
from datetime import datetime
from pathlib import Path

# Setup mock environment/directory
data_directory = Path("/g/data/xx00/some_data")

# Target Range: We only want files from March 2016 to May 2017
start_bound = datetime(2016, 3, 1)
end_bound = datetime(2017, 5, 31)

valid_paths = []

# We use rglob to catch them even if they are nested in subfolders
for file_path in data_directory.rglob("maq5*.nc"):
    
    # 1. Extract the filename without the full path or extension (.stem fetches '<filename_YYYYMMDD>')
    filename_stem = file_path.stem
    
    # 2. Extract the date portion string ('20260315'). Note the value of [1]
    #.   will depend on how many underscore '_' characters are in your actual files
    date_string = filename_stem.split("_")[2]
    
    try:
        # 3. Parse the string into a comparable datetime object
        file_date = datetime.strptime(date_string, "%Y%m%d")
        
        # 4. Check if the file fits neatly within our timeline boundaries
        if start_bound <= file_date <= end_bound:
            valid_paths.append(file_path)
            
    except ValueError:
        # Skips files that don't match the expected date formatting pattern
        continue

# Output the discovered target paths
print(f"Found {len(valid_paths)} valid files within the timeframe:")
for path in valid_paths:
    print(f" -> {path}")
```

## Using pathlib with generators and iterators

You can build on this example to create elegant logic to load a subset of ERA5 data. In this example the value of `ERA_PRESSURE_DIR` and `ERA_SURFACE_DIR` were defined in a `config.py` file as
```python
ERA_PRESSURE_DIR = Path('/g/data/rt52/era5/pressure-levels/reanalysis/')
ERA_SURFACE_DIR = Path('/g/data/rt52/era5/single-levels/reanalysis/')
```

```python
def load_era(ERA_PRESSURE_DIR,
             ERA_SURFACE_DIR, 
             start_date):

    """
    Load the ERA-5 files from ERA_DIRS. 
    """

    # Extract the year string from an input date string
    year = start_date[:4]

    # First add pressure-level data
    pressure_vars = ['u','v','vo','r']  # List of atmospheric variables on pressure levels 
    target_files = {} # Create dictionary for files opened

    # Construct matching patterns relative to ERA_DIR
    pressure_patterns = [f'{var}/{year}/*{start_date}*.nc' for var in pressure_vars ]

    # Extract list of required files for this start_date and ensemble
    matched = list(
        itertools.chain.from_iterable(
            ERA_PRESSURE_DIR.glob(pattern) for pattern in pressure_patterns
        )
    )

    # Append filename to dictionary
    for file in matched:
        var = file.name.split('_')[0]
        print (f' INFO : Assigning {file} for variable {var}')
        target_files[var] = file 

    # Now repeat for surface level data
    surface_vars = ['sst']

    surface_patterns = [f'{var}/{year}/*{start_date}*.nc' for var in surface_vars ]

    matched = list(
        itertools.chain.from_iterable(
            ERA_SURFACE_DIR.glob(pattern) for pattern in surface_patterns
        )
    )

    for file in matched:
        var = file.name.split('_')[0]
        print (f' INFO : Assigning {file} for variable {var}')
        target_files[var] = file 

    return target_files
```

## Further examples

You can find more information here:

https://realpython.com/python-pathlib/

https://www.w3schools.com/python/ref_module_pathlib.asp