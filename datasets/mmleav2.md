---
title: "MMLEAv2"
location: "/g/data/su28/MMLEAv2"
maintainer: "TBD"
short_description: "Multi-Model Large Ensemble Archive version 2 includes 18 models from the CMIP project in a 2.5 degree grid."
---

# MMLEAv2

The Multi-Model Large Ensemble Archive version 2 (MMLEAv2) was compiled as an extension of the original archive. It includes 18 models (12 CMIP6 and 6 CMIP5) and 15 monthly variables. While a consistent RCP8.5 future forcing scenario was available for the CMIP5 class models, the CMIP6 class models use one or both of the historical plus SSP370 or SSP585 scenarios. Observational reference products are also included in this archive. All data is provided remapped to a common 2.5 x 2.5 degree grid.

NetCDF output from the Climate Variability Diagnostics Package version 6 (CVDP v6) run on the model for the time periods 1950-2022 and 2027-2099 is provided for both detrended and non-detrended MMLEAv2 data. The CVDP output can be found on the [MMLEAv2 website](https://www.cesm.ucar.edu/community-projects/mmlea/v2).

### Temporal range:

1950-01 to 2099-12

### Variables:

- Geopotential Height
- Maximum/Minimum Temperature
- Ocean Mixed Layer
- Rain
- Salinity
- Sea Ice Concentration
- Sea Surface Height
- Sea Surface Temperature
- Wind Stress

### Location:

This dataset is located at:

```bash
/g/data/su28/MMLEAv2
```

The data is then divided up into sub-categories with the following options:

```bash
/g/data/su28/MMLEAv2/<domain>/<frequency>/<variable>/<model>
```

**domain** - atm, ocn, seaice

**frequency** - monthly, daily

**variable** - evspsbl, pr, psl, tas, tauu, tauv, zg500, ts, Rx1day, TNn, TXx, mlotst, sos, tos, zos, siconc, observations

**model** - ACCESS-ESM1-5, CanESM5, CESM2, E3SM-1-0, EC-Earth3, IPSL-CM6A-LR, MIROC-ES2L, UKESM1-0-LL, CanESM2, CESM1-CAM5, CSIRO-Mk3-6-0, E3SMv2, GFDL-ESM2M, MIROC6, MPI-ESM


Each domain type has thje following variables:

- atm_mon = ['evspsbl', 'pr', 'psl', 'tas', 'tauu', 'tauv', 'zg500', 'ts']
- atm_daily = ['Rx1day', 'TNn', 'TXx']
- ocn_mon = ['mlotst', 'sos', 'tos', 'zos']
- seaice_mon = ['siconc']
- obs = ['observations']

### Data Contributers:

UCAR/NCAR/CGD

### Related Resources:

[MMLEA version 2 community project website](https://www.cesm.ucar.edu/community-projects/mmlea/v2)

### Publication:

Maher, N., A. S. Phillips, C. Deser, R. C. J. Wills, F. Lehner, J. Fasullo, J. M. Caron, L. Brunner, and U. Beyerle, 2024: [The updated Multi-Model Large Ensemble Archive and the Climate Variability Diagnostics Package: New tools for the study of climate variability and change](https://egusphere.copernicus.org/preprints/2024/egusphere-2024-3684/). EGUsphere, 2024, 3684 (DOI: 10.5194/egusphere-2024-3684).


### Extra:

Please see [NCAR's Research
Data Archive](https://rda.ucar.edu/datasets/d651039/#) for more information about this dataset.