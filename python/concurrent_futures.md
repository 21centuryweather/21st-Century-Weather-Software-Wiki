# Parallelism in Python using concurrent futures

The large datasets we deal with in weather and climate research typically require cutting-edge hardware and software to be analysed efficiently.  Modern computing architecture relies on clusters of Central Processing Units (CPUs) with multiple processing pipelines, or cores. Each core may have separate threads. A single standard computational node on `gadi` for example, has 2x24 core CPUs attached to 190 Gb of RAM. Each core has two threads, so a `python` script can actually see 96 separate processing cores. 

You can check this for yourself by logging into an interactive `gadi` compute note, by following [these instructions](https://opus.nci.org.au/spaces/Help/pages/90308778/0.+Welcome+to+Gadi#id-0.WelcometoGadi-InteractiveJobs), and launching a simple `python` interpreter.
```python
$ python
Python 3.10.14 | packaged by conda-forge | (main, Mar 20 2024, 12:45:18) [GCC 12.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import os
>>> os.cpu_count()
96
```
So we can write `python` scripts to run 96 separate processes at once, provided we don't exceed the total shared memory of the node. The amount of memory at each `gadi` node can be found [here](https://opus.nci.org.au/spaces/Help/pages/90308823/Queue+Limits).

The `normal` queue has 190 Gb available, while the `hugemem` and `megamem` have 1.4 Tb and 3.0 Tb (!) respectively.

So clearly have a lot of scope to use shared-memory programming to speed up our data analysis in `python`.

:::{note}

There are two paradigms of distributed computing. In the 'shared memory' paradigm, multiple processes have access to a shared pool of memory. In the 'distributed memory' paradigm, processes are directed to separate pools of memory distributed across a network. Typically, this is how heavy computational tasks (e.g. running a CMIP model) are handled. Often, a combination of distributed and shared-memory techniques can be used. This tutorial is a 'shared memory' example, where we attempt to solve a problem in a single, unified memory space shared by multiple processes.
::

Often, users first encounter parallel processing via the `xarray` module via a `dask` Client. In this way, a single `xarray` data structure can be read, processed and written in parallel 'chunks' using `dask` arrays. 'Chunking' can occur along any dimension - spatial or temporal.

There are other ways to exploit parallelism in `python`. If our data arrays are relatively small, but there are many of them (e.g. decades worth a daily datasets) we can spawn multiple parallel processes to process each file separately. We can analyse each file in parallel and write temporary results to disk. We can then collate this data into a single data structure at the end.

A good way to spawn multiple parallel processes in `python` is to use the `concurrent futures` library. The official documentation is [here](https://docs.python.org/3/library/concurrent.futures.html).

## An example using concurrent futures

In this example, we are processing 10-minute geostationary satellite estimates of solar irradiance over Australia. Originally, the code loaded all 10-minute datasets valid for a single day into a single `xarray` dataset. The code then used a simple for loop to loop over a range of days. The `xarray` dataset valid for every day was appended to a list, which was  then concatenated into a larger array along the time dimension. Below is a combination of actual code combined with pseudo-code to save space.
```python
start_dt = datetime.strptime(start_date, "%d-%m-%Y")
end_dt = datetime.strptime(end_date, "%d-%m-%Y")

# Generate a list of dates
date_range = [start_dt + timedelta(days=i) for i in range((end_dt - start_dt).days + 1)]

NEM_performance = []

for dir_dt in date_range:

    dataset = utils_V2.get_irradiance_day(resolution='p1s', dir_dt=dir_dt)

    #( Extract three irradiance variables : global, diffuse and direct from this dataset, and apply spatial masks to extract only the gridded areas of interest. Then convert the 2-D masked arrays, containing NaNs (empty/missing values i.e. 'Not a Number') into a 1-D array)
        
    # calculate capacity factors using pvlib
    # the function defined in utils_V2 is essentially the same as the workflow in pv-output-tilting.ipynb
    actual_ideal_ratio = utils_V2.tilting_panel_pr(
        pv_model = 'Canadian_Solar_CS5P_220M___2009_',
        inverter_model = 'ABB__MICRO_0_25_I_OUTD_US_208__208V_',
        ghi=ghi_clean,
        dni=dni_clean,
        dhi=dhi_clean,
        time=time_1d_clean,
        lat=lat_1d_expanded_clean,
        lon=lon_1d_expanded_clean
    )  

    #( Convert the output of this PV function back into a 2-D array )
    # Then convert it into a dataset 

    ratio_da = xr.DataArray(reshaped, coords=mask_template.coords, dims=mask_template.dims)

    # Compute the spatial mean in the gridded areas of interest 

    mean_daily = ratio_da.mean(dim=["latitude", "longitude"], skipna=True)
    NEM_performance.append(mean_daily)

# Append it to the list of daily dataarrays
NEM_performance_timeseries = xr.concat(NEM_performance, dim='time')
```
It takes about three minutes to process a single day. Separate `qsub` scripts were generated for a given month, and submitted to the PBS queue on a single core.

:::{note}

In the `gadi` PBS scheduler, the number of CPUs that you request (i.e. `#PBS -l ncpus`) is the number of **cores**. Remember there are typically two 24 core CPUs located on each node. If want full access to every core on a node, we need to request `#PBS -l ncpus=48`)
:::

The solar irradiance dataset is relatively small. The spatial latitude-longitude dimensions are 1726 x 2214. The function `utils_V2.get_irradiance_day` uses `xarray.open_mfdataset` to collate 103 separate files (valid very 10 minutes). Clearly this can be processed quite quickly on a single core. So can we use `python` libraries to exploit the multiple cores on a single CPU?

The first step is to write a function that encapsulates all our daily computations. So a few function was written, which contained all the code listed above to compute the spatial mean across the gridded area of interest
```python
def compute_timeseries_clearsky_ratio(date,
                                      file_path):
    """
    Compute a timeseries of ratio b/w tilting panel irradiance
    relative to clear sky conditions for a given day
    """

    #( Contain all the above code that controls the compution of mean irradiance for a single day)#
    
    # In this fuction, we will write a seperate netCDF file for each day
    os.makedirs(file_path, exist_ok=True)
    LOG.info(f'Writing data for {date} to {file_path}/{date}.nc')
    mean_daily.to_netcdf(f'{file_path}/{date}.nc')

    return
```
I can save this function to a file `compute_data.py` so it can be imported into my main python script as a separate module. Now we can use the `concurrent.future` package so the main python script can send the computation of each day to a separate core in parallel. Let's compute 10 days in parallel, across 10 cores.
```python
import concurrent.futures
import os,sys
from datetime import datetime, timedelta
from pathlib import Path

import logger
from compute_data import compute_timeseries_clearsky_ratio

LOG = logger.get_logger(__name__)

TIMESERIES_DIR=Path('/g/data/gb02/pag548/solar_drought/REZ_tilting/ideal_ratio/NEM_timeseries')

HIMAWARI_SOLAR_DIR=Path('/g/data/rv74/satellite-products/arc/der/himawari-ahi/solar/')

if __name__ == "__main__":

    n_procs = os.cpu_count() # Find number of seperate CPUs
    worker_pool = concurrent.futures.ProcessPoolExecutor(max_workers=10)  # Create the number of parallel branches and fix it equal to the number of cores. Each core will have one job submitted

    futures = {}

    # Serial test
    start_date = '1-2-2022'
    end_date = '12-2-2022'
    
    start_dt = datetime.strptime(start_date, "%d-%m-%Y")
    end_dt = datetime.strptime(end_date, "%d-%m-%Y")

    # Generate a list of dates
    date_range = [start_dt + timedelta(days=i) for i in range((end_dt - start_dt).days + 1)]

    # Loop over the dates
    for date in date_range: 

        date_s = date.strftime('%d-%m-%Y')

        LOG.info(f'Computing ratio of tilting to clearsky irradiance for {date_s}')

        # Here we submit each function call to the worker pool. This loop will happen process instantly.
        future = worker_pool.submit(
            compute_timeseries_clearsky_ratio,
            date_s, 
            TIMESERIES_DIR)

        futures[future] = f"The job for {date}"

    LOG.info(f'Complete')
```
If this python job is submitted to the PBS job queue, it will take roughly 4:15 seconds to compute 10 days. The total memory usage is 153 Gb, which suggests each parallel process consumes about 15 Gb. So on a `normal` PBS node, we can only use about 13 cores before we risk running out of memory (190/15 = 12.6).

Using 13 cores, 28 days of data (i.e. all of February) can be computed in 10 minutes and consumes about 170 Gb. This is because we can only process 13 days at once. If you attempt to use more cores, you will exceed the shared memory capacity of a `normal` node.

Of course, you could run the job on a `hugemem` or `megamem` node. This could allow you to use all 96 threads in parallel (remember each `gadi` core contains two threads ). 96 threads times 15 Gb = 1.44 Tb. So the entire job could squeeze into the `hugemem` queue, but it would definitely fit in the `megamem` queue. 

Using this strategy could process an entire year of data (365 days) in a matter of minutes. i.e. 96 cores would process 96 days in parallel, so 365 days would take roughly (365/96) = 3.8 * 4 = 15 minutes. So a single job submitted to the `hugemem` or `megamem` queue could process a decade of data in under 3 hours.

To find out more about the `hugemem` and `megamem` queues, see [here](https://opus.nci.org.au/spaces/Help/pages/90308823/Queue+Limits). Note these queues are more expensive than the `normal` queues in terms of charge rates per hour, and you may have to wait longer in the queue for your job to process. But they are a useful options for tasks that aren't overly repetitive.

Another option is to reduce the memory overhead of a single day's computations, so we can squeeze more processes onto the `normal` node. Simple steps such as
- Only loading the required dataset variables from disk 
- Only loading the spatial subset of data from disk
will reduce the memory overhead of your process.

Reviewing Scott Wales' old CLEX posts on how to optimise `xarray.open_mfdataset` would be beneficial. See [here](https://opus.nci.org.au/spaces/Help/pages/90308823/Queue+Limits) on how to use the `preprocess` argument with `open_mfdataset`.

:::{warning}

When using `concurrent.futures` ensure you don't invoke the `parallel` option when using `open_mfdataset`. I'm unsure how the two libraries would compete for cores. You could always try a small experiment and see what happens?
:::

:::{note}

This kind of workflow - submitting individual tasks to a pool of 'workers' which then send the task to an individual core - is a gateway to `Dask` itself. `Dask` contains its own `futures` library which is derived from the python built-in library `concurrent.futures`. If you found this example interesting and would like to experiment more, examine this [tutorial](https://tutorial.dask.org/05_futures.html)
:::
