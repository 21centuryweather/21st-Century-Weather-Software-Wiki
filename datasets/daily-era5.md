---
title: "ERA5 daily"
location: "/g/data/su28/ERA5/daily"
maintainer: "TBD"
short_description: "Reanalysis. Variables in daily frequency remaped to 1 degree"
---
# ERA5 daily 

ERA5 is a climate reanalysis dataset, covering the period 1950 to present. This dataset includes 2d and 3d common variables in daily frequency. The data was also remaped from 0.25 degree to 1 degree.

Variables

2D:

* 2t (2 metre temperature, K)  
* mer (Mean evaporation rate, kg m**-2 s**-1)  
* msl (Mean sea level pressure, Pa) 
* mslhf (Mean surface latent heat flux, W m**-2)
* msshf (Mean surface sensible heat flux, W m**-2)
* mtpr (Mean total precipitation rate, kg m**-2 s**-1)
* mvimd (Mean vertically integrated moisture divergence, kg m**-2 s**-1)
* viwve (Vertical integral of eastward water vapour flux, kg m**-1 s**-1)
* viwvn (Vertical integral of northward water vapour flux, kg m**-1 s**-1)
* vo (Vorticity (relative), s**-1) 
* tp (Total precipitation, m)

3D:

* q (Specific humidity, kg kg**-1) 
* t (Temperature, K)
* u (U component of wind, m s**-1)
* v (V component of wind, m s**-1)
* w (Vertical velocity, Pa s**-1)
* z (Geopotential, m**2 s**-2)

In 22 pressure levels (10,20,30,50,70,100,150,175,200,225,250,300,400,500,600,700,800,850,900,925,950,1000hPa) 

## Data location

```bash
/g/data/su28/ERA5/daily
```

For hourly and monthly data check NCI project `rt52`. 

## Acknowledgment  

When using this dataset, please acknowledge [Linyuan Sun](https://www.21centuryweather.org.au/profile/linyuan-sun) for processing ERA5 hourly data to daily. 