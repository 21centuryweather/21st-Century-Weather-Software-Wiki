# ERA5-based Weather Objects

There are 3 ERA5-based Weather Objects placed on Gadi in su28, located at:

```/g/data/su28/weatherfeatures.era5/```

For more indepth details about the following datasets, please see the files in ```/g/data/su28/weatherfeatures.era5/{model}/docu/```

### 1. Anticyclones (MAXCL):

ERA5-based anticyclones are calculated based on sea-level pressure (SLP). more specifically, closed isobars are identified in the SLP field and labeled as anticyclone areas.
To exclude artificial anticyclones over height topography (>1500 m), which can result from the pressure reduction to
sea level, anticyclone centers (SLP minima) are omitted where the topography exceeds 1500 m.
The most detailed description of the algorithm can be found in Sprenger et al. (2017). There, also differences
between the current ERA5 algorithm and the original ERA-Interim one in Wernli and Schwierz (2006) is discussed.

#### Data source:
**[1a]**```/g/data/su28/weatherfeatures.era5/maxcl/cdf/{year}```

where {year} is 1950 - 2022.

`AYYYY_MM`: netCDF files with anticyclone masks (areas inside closed isobars). Each file is a month's worth of data at hourly frequency. The following fields are available on the netCDF files:

- `FLAG` unfiltered anticyclone masks (0/1 label)

### 2. Cyclones (MINCL):

ERA5-based cyclones are calculated based on sea-level pressure (SLP). more specifically, closed isobars are identified
in the SLP field and labeled as cyclone areas. SLP minima within the cyclone areas are also identified and, by connecting the SLP positions (latitude/longitude) over consecutive times, cyclone tracks are determined. The first
instance (time, lat/lon) of the tracks is saved as the cyclogenesis position, the last instance as the cyclolysis. The
overall lifetime of a cyclone has to exceed 12 hours, otherwise the cyclone track and the corresponding cyclone area
is omitted. A cylcone track is allowed to have two ERA5 time instances (2 hours) without SLP minimum, i.e. 2-h gaps
in the cyclone tracks can be 'bridged'.

To exclude artificial cyclones over height topography (>1500 m), which can result from the pressure reduction to sea
level, cyclone centers (SLP minima) are omitted where the topography exceeds 1500 m. If there are several SLP
minima located inside a cyclone area, i.e. the SLP minima are enclosed by a common SLP contour, the cyclone mask
is split among all SLP minima, according to the SLP value (deeper SLP minima cover larger areas of the mask than
weaker SLP minima).

The most detailed description of the algorithm can bei found in Sprenger et al. (2017). There, also differences
between the current ERA5 algorithm and the original ERA-Interim one in Wernli and Schwierz (2006) is discussed.

#### Data source:
**[2a]**```/g/data/su28/weatherfeatures.era5/mincl/cdf/{year}```

where {year} is 1950 - 2022.

`CYYYY_MM`: netCDF files with cyclone masks (areas inside closed isobars). Each file is a month's worth of data at hourly frequency. The following fields are available on the netCDF files:

- `INPUT` unfiltered cyclone masks (0/1 label), i.e., no lifetime filter applied
- `LABEL` filtered cyclone masks; each cyclone mask gets a unique label that is consistent to
the label in the cyclone track files (see [2a])
- `AGE` age (in hours) of the cyclone attributed to LABEL
- `LIFETIME` overall lifetime (in hours) of the cyclone attributed to LABEL
- `PMIN` 0/1 field with 1 at the locations of SLP minima

### 3. Warm-Conveyor Belts (WCB)

ERA5-based warm conveyor belts calculated as trajectories ascending from levels below 790 hPa within 48 h by at
least 600 hPa. The starting positions are set equidistantly (80 km) over the whole globe, and the trajectories are
started every 6 h. Trajectory positions are saved every hour. The ascent has to occur in the neighborhood of a
cyclone. The trajectories are clustered into spatially coherent air streams, and if one of the cluster trajectories passes
across a cyclone mask the whole cluster is kept. The main data sources are [2d], the WCB masks at hourly temporal
resolution and separating regions affected by a WCB inflow, ascent and outflow. The ERA5 time period 1980-2020 is
available.

#### Data source:
**[3a]**```/g/data/su28/weatherfeatures.era5/wcb/cdf.1hourly/{year}```

where {year} is 1980 - 2022.

`hit_YYYY_MM`: netCDF files with WCB masks. Each file is a month's worth of data at hourly frequency. A 100-km gridding radius is applied (except for `PRESSURE` and `TIME`). The following fields are available on the netCDF files:

- `GT800` 0/1 label for grid points in WCB inflow (p > 800 hPa)
- `MIDTROP` 0/1 label for grid points in WCB ascent (400 hPa < p < 800 hPa) 
- `LT400` 0/1 labels for grid points in WCB outflow (p< 400 hPa, t <= 48 h)
- `TOTAL` any of GT800, MIDTROP or LT400
- `PRESSURE` pressure at WCB grid point (no 100-km radius applied)
- `TIME` time (in h) at the WCB grid points (no 100-km radius applied)

**[3b]**```/g/data/su28/weatherfeatures.era5/wcb/cdf.6hourly/{year}```

where {year} is 1980 - 2022.

`asc_YYYY_MM`: netCDF files with the time evolution of the WCB ascent from 00 to 96 h, in steps of 12 h. Time 00 corresponds to the time the trajectory starts ascending. The datetime in the file corresponds to the starting date of the WCB trajectories. A 100-km gridding radius is applied. The following fields are available on the netCDF files:

- `FLAG` position (0/1 label) of the WCB trajectories

`hit_YYYY_MM`: as in **[3a]**.

#### Additional information:
- WCB trajectories would be available on request
- The cyclone masks used for the filtering can be found at ```/g/data/su28/weatherfeatures.era5/mincl/cdf/```

### References:

Wernli, H., and Schwierz, C. (2006). Surface Cyclones in the ERA-40 Dataset (1958–2001). Part I: Novel Identification
Method and Global Climatology, Journal of the Atmospheric Sciences, 63(10), 2486-2507.

Sprenger, M., Fragkoulidis, G., Binder, H., Croci-Maspoli, M., Graf, P., Grams, C. M., Knippertz, P., Madonna, E.,
Schemm, S., Škerlak, B., & Wernli, H. (2017). Global Climatologies of Eulerian and Lagrangian Flow Features based on
ERA-Interim, Bulletin of the American Meteorological Society, 98(8), 1739-1748.

Madonna, E., Wernli, H., Joos, H., & Martius, O. (2014). Warm Conveyor Belts in the ERA-Interim Dataset (1979–
2010). Part I: Climatology and Potential Vorticity Evolution, Journal of Climate, 27(1), 3-26.

### Contact:
If there are any issues with these datasets please contact:

[Sam Green](https://www.21centuryweather.org.au/profile/sam-green)  
[Chenhui Jin](https://www.21centuryweather.org.au/profile/chenhui-jin)