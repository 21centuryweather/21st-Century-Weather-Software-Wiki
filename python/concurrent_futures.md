# Parallelism in Python using concurrent futures

The large datasets we deal with in weather and climate research can take considerable time to interrogate and process. Modern computing architecture uses Central Processing Units (CPUs) with multiple processing pipelines, or cores. Each core may have separate threads. A single standard computational node on `gadi` for example, has 2x24 core CPUs attached to 190 Gb of RAM. Each core has two threads, so a `python` script can actually see 96 separate processing cores. 

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

Typically, users first encounter with parallel processing occurs via the `xarray` module via a `dask` Client. In this way, a single `xarray` data structure can be read, processed and written in parallel 'chunks' using `dask` arrays. 'Chunking' can occur along any dimension - spatial or temporal.

There are other ways to exploit parallelism in `python`. If our data arrays are relatively small, but there are many of them (e.g. decades worth a daily datasets) we can spawn multiple parallel processes to process each file separately. We can analyse each file in parallel and write temporary results to disk. We can then collate this data into a single data structure at the end.

A good way to spawn multiple parallel processes in `python` is to use the `concurrent futures` library. The official documentation is [here](https://docs.python.org/3/library/concurrent.futures.html).

## An example using concurrent futures

In this example, we are processing 10-minute geostationary satellite estimates of solar irradiance over Australia. Originally, the code loaded all 10-minute datasets valid for a single day into a single `xarray` dataset.