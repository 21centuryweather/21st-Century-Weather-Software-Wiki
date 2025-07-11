# Using Zarr with Xarray for Scalable Scientific Data

### 1. What is Zarr and Why Use It?
Zarr is a format for storing N-dimensional arrays in a **chunked, compressed, and lazily-loadable** manner. Unlike NetCDF:
- Zarr stores data as **directory structures** with many small `.npy`-like chunks.
- Designed for parallel and cloud-friendly access.
- Works seamlessly with **Xarray + Dask** for large out-of-core datasets.

Use Zarr if:
- You’re working with > memory datasets.
- You want to scale across HPC or cloud storage (e.g., S3).
- You need faster writes and flexible chunking.

---

### 2. Creating a Zarr Store from NetCDF
```python
import xarray as xr

# Open NetCDF file
ds = xr.open_dataset("example.nc", chunks={"time": 100})  # use dask chunks!

# Save to Zarr
ds.to_zarr("example.zarr", mode="w")
```

To use compression:
```python
import numcodecs

compressor = numcodecs.Blosc(cname="zstd", clevel=5, shuffle=numcodecs.Blosc.SHUFFLE)
encoding = {var: {"compressor": compressor} for var in ds.data_vars}

ds.to_zarr("compressed.zarr", encoding=encoding, mode="w")
```

---

### 3. Opening Zarr Stores with `open_zarr`
```python
ds = xr.open_zarr("example.zarr", chunks={})  # Auto chunk
print(ds)
```
You can control chunks:
```python
ds = xr.open_zarr("example.zarr", chunks={"time": 500, "lat": 100, "lon": 100})
```
> `open_zarr()` is **lazy** and uses Dask under the hood for parallelism.

---

### 4. Writing Data to Zarr in Pieces (Streaming / Sharded)
To write data over time slices:
```python
ds_1980 = xr.open_dataset("data_1980.nc", chunks={"time": 10})
ds_1980.to_zarr("multi_year.zarr", append_dim="time", mode="w")

ds_1981 = xr.open_dataset("data_1981.nc", chunks={"time": 10})
ds_1981.to_zarr("multi_year.zarr", append_dim="time", mode="a")  # append
```
This avoids loading full datasets into memory.

---

### 5. Opening and Combining Multiple Zarr Stores
If you’ve partitioned your dataset into multiple Zarrs (e.g., by year):
```python
import xarray as xr
from glob import glob

paths = sorted(glob("data/*.zarr"))

datasets = [xr.open_zarr(p, chunks={"time": 100}) for p in paths]
combined = xr.combine_by_coords(datasets)
```
For consistent time ordering, sort paths and check for duplicate coordinates.

---

### 6. Performance Tips (HPC / Cloud)
- Always chunk your data! Use `ds.chunk()` and match to your access pattern.
- Use `numcodecs.Blosc` for fast compression; Zstd is a good default.
- Avoid too many small variables/files.
- If running on cloud (e.g., S3), use `fsspec` and `gcsfs`, `s3fs`, etc.

```python
fs_map = fsspec.get_mapper("s3://my-bucket/data.zarr", anon=False)
ds = xr.open_zarr(fs_map, consolidated=True)
```

---

### 7. Advanced Tools: Kerchunk + Zarr v3
**Kerchunk**: Virtual Zarr index from NetCDF/HDF5 files — great for cloud or read-only stores:
```python
import fsspec
import xarray as xr

reference_path = "reference.json"  # created by kerchunk
fs = fsspec.filesystem("reference", fo=reference_path)
mapper = fs.get_mapper("")

ds = xr.open_dataset(mapper, engine="zarr", backend_kwargs={"consolidated": False})
```

**Zarr v3** (experimental): Enables **sharding** and **better cloud compatibility**. You can enable it in recent versions of `zarr-python` and `xarray`.

---

### Summary: Zarr with Xarray
| Task | Method |
|------|--------|
| Open Zarr | `xr.open_zarr()` |
| Save Zarr | `ds.to_zarr()` |
| Compress | Use `numcodecs` in `encoding` |
| Combine Zarrs | `xr.combine_by_coords()` |
| Use with Cloud | Use `fsspec.get_mapper()` |
| Virtual Zarr from NetCDF | Use `kerchunk` |
| Parallel I/O | Use Dask with chunks |
