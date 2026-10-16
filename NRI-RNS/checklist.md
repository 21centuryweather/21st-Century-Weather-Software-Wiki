# Checklist before running your own regional experiment

So you're all set to begin your own regional modeling experiment? Great!  But, bear in mind this is no-easy feet. Taken from the UK Met Office manuals.
"""
Setting up the Unified Model for a new limited area domain is relatively straightforward, and will enable you to run higher resolution suites concentrating on your area of interest. However you should at the outset consider the time and effort involved.
"""

See [here](https://21centuryweather.github.io/UM_summary_docs/using.html#setting-up-a-new-limited-area-model-lam) for further advice.


Here is a quick checklist of all the issues you need to address.

## Resources

Ask yourself
1) Which project will I run the experiment under?
2) What are the current available resources? (See [here](https://21centuryweather.github.io/nci_resource_tools/dashboard.html#usage))
3) Register your request for disk storage and compute resources on Cumulus, e.g. ([here](https://21centuryweather.discourse.group/c/modelling-science/compute-requests-for-fy29/46) and [here](https://21centuryweather.discourse.group/c/centre-project-2-sst-movs-and-weather-systems/storage-allocation-requests-for-if69/50))
4) Check your own quota in your home directory with `$ quota -v`

## Ancillary generation

1) Use the tools [here](https://github.com/21centuryweather/UM_configuration_tools/blob/main/notebooks/UM_plot_domain.ipynb) to help size your domain.
2) Make sure your domains have even number of grid points!
3) Which land-surface data are you using to initialise your model?
    - ERA5-land? Make sure your second ancillary resolution is defined at regularly spaced 0.1 degree increments (i.e. 135.5, 135.6 etc)
    - BARRA2?  Make sure you second ancillary resolution is defined at 0.11 degree increments and is aligned with the existing BARRA2 grid.  You will need membership of the `ob53` project to access BARRA2 land-surface data.  Load one the BARRA2 files and check your second ancillary resolution points match the BARRA2 grid
4) Does your domain exceed 60 degrees latitude? If so, you won't be able to use the standard SRTM Orography. See [here](https://forum.access-hive.org.au/t/regional-ancillary-suite-error-srtm-data-not-available-for-regions-above-60-deg/5796) for further discussions.
      - Don’t exceed 58 degrees south when using BARRA land surface, because BARRA ends at 57.97 degrees south.

## Forecast run

1) Have you checked your ancillaries don't contain any NaNs using [this notebook?](https://github.com/21centuryweather/UM_configuration_tools/blob/main/notebooks/Check_UM_ancillaries.ipynb)
    - If your domain includes small islands in the Southern Ocean (e.g. Macquarie Island) there is a required work-around for small island vegetation canopy bug. See [here](https://forum.access-hive.org.au/t/aus2200-vegetation-fraction-ancil-creation-issues/1972/20) for more information.
2) Consider your PE layout. Due to how the UM decomposes a domain into a patchwork of sub-domains which each run on a separate processors, the number of overall points in your domain places a maximum of the number of processors you can use. See [here](https://forum.access-hive.org.au/t/issues-when-changing-domain-in-access-ram3/5943/5) and [here](https://forum.access-hive.org.au/t/issues-when-changing-domain-in-access-ram3/5943/7)

2) For larger layouts, make the number of PEs must be a multiple of 48. This ensures effective use of the NCI compute resources.

** TO DO **

- Guide to interpreting typical model errors – what to look for in the log files and what they likely mean.
- Guidelines for spatial resolution and model selection (GAL9/RALP3P3)
- Include guide to where the log files are and what is in each type