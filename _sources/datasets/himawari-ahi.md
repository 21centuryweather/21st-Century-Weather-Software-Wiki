---
title: "Cloud Type from Himawari"
location: "/g/data/su28/himawari-ahi"
maintainer: "TBD"
short_description: "Cloud type product is derived from Himawari AHI"
---
# Himawari-AHI cloud type data

The cloud type product is derived from Himawari AHI data using NWC SAF algorithms included in NWC/GEO software package. The CT product contains information on the major cloud classes: fractional clouds, semitransparent clouds, high, medium and low clouds (including fog) for all the pixels identified as cloudy in a scene. 

The dataset derived from the Bureau of Meteorology Satellite Derived Products under various NCI data collection, check the [documentation](https://opus.nci.org.au/spaces/NDP/pages/206110970/Himawari-AHI+Cloud+Type+CT) for more details.

This dataset have been post-processed to include the timestamp as a dimension and converted nx/ny to lat/lon coordinates. The domain is the Australian regional domain. Please cite/awknowledge the following people if this data has been used for publications/presentations: Samuel Green - ORCID 0000-0003-1129-4676; Mat Lipson - ORCID 0000-0001-5322-1796; Kimberley Reid - ORCID 0000-0001-5972-6015.

### Data location

```
/g/data/su28/himawari-ahi
```

### Data format and variables

The data is available in zarr format, to read it follow the example:

```python
import xarray as xr

ds = xr.open_zarr("/g/data/su28/himawari-ahi/cloud/ct/aus_regional_domain/S_NWC_CT_HIMA08_HIMA-N-NR.zarr/")
ds
```

Variables:

* ct
* ct_conditions
* ct_cumuliform
* ct_multilayer
* ct_quality
* ct_status_flag
