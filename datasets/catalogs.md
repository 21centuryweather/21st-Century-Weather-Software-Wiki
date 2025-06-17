# Data catalogs

An intake data catalog is a easy way to access a dataset (usually composed of many files). It is also useful to discover new datasets available and list variables, date ranges and other attributes. If you ever tried to work with CMIP6 data, you know how hard it can be to find the file (or files) you need to read. The catalog will also alow you to load the data to start working.

## How to use a catalog

The workflow using a catalog will require to:

1. Read a catalog and list the datasets available in that catalog. For example:

```python
import intake

data_catalog = intake.open_catalog("/g/data/su28/tools/su28_catalog/catalog/su28_catalog.yaml")
list(data_catalog)
```
```
['daily_era5', 'drought_era5', 'MMLEAv2', 'GSMaP', 'weather_objects']
```

2. Explore the metadata and the parameters for a specific dataset, these can be variables included, frequency, resolution, date, model, etc.

```python
daily_era5 = data_catalog['daily_era5'] # data_catalog.daily_era5 gives the same result
daily_era5.unique()
```
```
variable                [2t, mer, msl, mslhf, msshf, mtpr, mvimd, q, t...
dataset                                                            [era5]
level                                                           [sfc, pl]
resolution                                                         [1deg]
frequency                                                         [daily]
year                    [1950, 1951, 1952, 1953, 1954, 1955, 1956, 195...
name_in_file            [t2m, mer, msl, mslhf, msshf, mtpr, mvimd, q, ...
long_name               [2 metre temperature, Mean evaporation rate, M...
units                   [K, kg m**-2 s**-1, Pa, W m**-2, kg kg**-1, m,...
path                    [/g/data/su28/ERA5/daily/2t/2t_era5_oper_sfc_m...
derived_name_in_file                                                   []
dtype: object
```
3. Search the dataset using one or more parameters.

```python
daily_era5.search(variable = "2t")
```

4. Read the data

```python
data = daily_era5.search(variable = "2t").to_dask()
```

This is an example using the catalog for the datasets hosted on `su28`. Not all catalogs will work exactly the same way but the idea is the same. Also, each dataset can have different parameters. For that, it is very important to no skip step 2 and take your time to get familiar with each dataset.

## Available catalogs on Gadi

### `su28` catalog

(almost) all datasets hosted on `su28` and [documented in this wiki](datasets/index.html) are listed in the su28 catalog. The catalog can be access through the url or from Gadi:

```python
catalog_url = intake.open_catalog("https://raw.githubusercontent.com/21centuryweather/su28_catalog/refs/heads/main/catalog/su28_catalog.yaml")
catalog_path = intake.open_catalog("/g/data/su28/tools/su28_catalog/catalog/su28_catalog.yaml")
```

An example on how to use this catalog can be found [here](https://github.com/21centuryweather/su28_catalog/blob/main/docs/how_to_catalog.ipynb).

### NCI catalogs

Many of the datasets hosted by NCI have an associated intake catalog (don't confuse with the [online NCI Data Catalogue](https://geonetwork.nci.org.au/geonetwork/srv/eng/catalog.search#/home)). 

NCI catalogs can be access using `open_esm_datastore` as these are intake-esm catalogs. For example:

```python
catalog = intake.open_esm_datastore("/g/data/rt52/catalog/v2/esm/catalog.json")
```

#### NCI catalog (incomplete) list

Here is the list of catalog we know exist. If you find any catalog not listed here, please let us know by completing this [form](https://creatorapp.zohopublic.com.au/21centuryweather/w21c-clever/form-perma/Wiki_Feedback_Form/dNjCfQET7NjEgyE6GATZgRXhqOUEnHtsANkEZSJ54Cp8ry74xBHQQz6AvfOYhOXmJqdP35fBSjdbdnFx5ySvxFurqPPzBnnXjSw8).

| Dataset | path on Gadi |
|---------|--------------------------------------------|
| ERA5 Single and pressure-levels   | `/g/data/rt52/catalog/v2/esm/catalog.json` |
| CMIP6   | `/g/data/oi10/catalog/v2/esm/catalog.json` |
| CMIP6 Australian data | `/g/data/fs38/catalog/v2/esm/catalog.json` |
| CMIP5   | `/g/data/al33/catalog/v2/esm/catalog.json` |
| CMIP5 Australian data | `/g/data/rr3/catalog/v2/esm/catalog.json` |
| BARPA   | `/g/data/py18/catalog/v2/esm/catalog.json` |
| BARRA2  | `/g/data/ob53/catalog/v2/esm/catalog.json` |
| AI/DL reference data | `/g/data/w00/catalog/v2/esm/catalog.json` |
| CMIP forcing | `/g/data/qv56/catalog/v2/esm/catalog.json` |

> Note: you may find that NCI has 2 versions of the same catalog, one uses intake-esm (your example here) and the other uses intake-spark that was specifically developed for very big datasets. 

### ACCESS-NRI catalog

The ACCESS-NRI Intake catalog provides a way for Python users to peruse, search and load ACCESS and ACCESS-related climate data available on Gadi. If you have access to `xp65` you already have access to the catalog:

```python
catalog = intake.cat.access_nri
```

You can find a comprehensive tutorial [here](https://access-nri-intake-catalog.readthedocs.io/en/latest/usage/quickstart.html).

Also, if you've run an ACCESS model ad you want to make the simulations available to people, contributing to the ACCESS-NRI catalog is a great way to do it. [This section of the documentation explains how to do it](https://access-nri-intake-catalog.readthedocs.io/en/latest/datastores/index.html#datastore-create) and you can contact the RSE team if you need help. 

## Cataloging your own datasets

Catalogs are not only used to share datasets with others. If you've produced a dataset, it may be useful to create a catalog to easily access the data for your personal use. This is particularly important if is it includes many files. 

The process of creating a catalog is very easy, the only challenge will be to list all the files and extract the necessary information to create a table with attributes or parameters. If you need support doing this, contact the RSE team!