# GSMaP

The Global Satellite Mapping of Precipitation is a dataset of hourly global rainfall maps and a re-analysis version of the Near-Real-Time product by JAXA Global Rainfall Watch for meteorological and climate studies. 


## Data location

```bash
/g/data/su28/GSMaP/
```

Only the daily data (average between 00 to 23 hours) was downloaded but 1 hourly is available too. More information is available from the PDF and readme file available in the directory and online and from the [GSMaP webpage](https://sharaku.eorc.jaxa.jp/GSMaP/index.htm). The global grid is limited to 60N-60S and resolution is 0.1 degree latitude/longitude. 

Description:

* `{version}/netcdf/rnl_gauge/day/00Z-23Z/` includes daily averaged precipitation (mm/hr) in netcdf format.
    * `v6` covers from March 2000 to February 2014.
    * `v7` covers from March 2014 to December 2019.
    

## License

Anyone using this product should register with the [JAXA GSMaP products portal](http://sharaku.eorc.jaxa.jp/GSMaP/index.htm). You should include acknowledgements and citations as directed in the pdf available with the data and from the GSMaP webpage. JAXA would also like to receive a copy of your manuscript, again see instructions on the webpage. Signing up is also recommended to receive the frequent updates on the GSMaP products, while they mostly refer to the Near-Real-Time and gauge products, the re-analysis is derived from them so they might contain important information.

## References

Kubota T. et al. (2020) Global Satellite Mapping of Precipitation (GSMaP) Products in the GPM Era. In: Levizzani V., Kidd C., Kirschbaum D., Kummerow C., Nakamura K., Turk F. (eds) Satellite Precipitation Measurement. Advances in Global Change Research, vol 67. Springer, Cham DOI: 10.1007/978-3-030-24568-9_20 


The documentation for this dataset derived from [CLEX - ARC Centre of Excellence for Climate Extremes wiki page](http://climate-cms.wikis.unsw.edu.au/GSMaP). 