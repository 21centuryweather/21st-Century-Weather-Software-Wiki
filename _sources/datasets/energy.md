# Energy Demand dataset

This dataset contains predictions of daily energy demand for the Australian National Electricity Market (NEM) and participant States (QLD, NSW, VIC, SA, TAS) between 1940 and 2023.

The predictions were generated using an Extra Trees ensemble machine learning model with meteorological variables based on the ECMWF ERA5 reanalysis as predictors. Predictor data is also included in this repository.

The model was trained on demand observations from 2010-2016 and tested on observations from 2017-2019. This means the data here reflect how demand varies as a function of the weather according to the 2010-2019 power system.

The modelling procedure is described in Richardson et al. (2025), doi: 10.1088/1748-9326/ad9b3b. The data differ from those described in the paper because (1) the data length is longer here and (2) in the paper demand predictions are made using detrended predictors. The data here are not detrended.

### Data:
There are six files, one per region. Each file has one column for the date, columns for each of the predictors ordered left to right according to their importance in the model, and one column for predicted demand. Column names are summarised here with units:

date: yyyy-mm-dd
t2m: mean daily temperature [K]
t2m3: 3-day average of daily mean temperature [K]
t2m4: 4-day average of daily mean temperature [K]
t2max: maximum daily temperature [K]
msdwswrf : solar radiation at surface [W $m^{-2}$]
rh: relative humidity [%]
q: specific humidity [g $kg^{-1}$]
w10: 10m wind speed [m $s^{-1}$]
mtpr: mean total precipitation rate [kg $m^{-2} s^{-1}$]
cdd: cooling degree days [deg C]
prediction: demand prediction [MWh]

### Location:

The data is located at:

```bash
/g/data/su28/energy_demand
```

### How to cite this dataset:
If using these data, please cite the original publication, available from https://iopscience.iop.org/article/10.1088/1748-9326/ad9b3b.

A Zenodo repository will be created and also cirted here once available. 