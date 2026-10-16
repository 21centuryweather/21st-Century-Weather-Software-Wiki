# Doing-it-yourself

So you're ready to build your own regional model simulation? Great!  But, bear in mind this is no-easy feet. Taken from the UK Met Office manuals.
"""
Setting up the Unified Model for a new limited area domain is relatively straightforward, and will enable you to run higher resolution suites concentrating on your area of interest. However you should at the outset consider the time and effort involved.
"""

See [here](https://21centuryweather.github.io/UM_summary_docs/using.html#setting-up-a-new-limited-area-model-lam) for further advice.

Here is a quick checklist of all things we have covered. 
- Ensure your domains have an even number of grid points.
- Don’t exceed 60 degrees south along your southern boundary. This is due to the definition SRTM ancilliary files used to define orography. There are work-arounds, but they are complex. See [here](https://forum.access-hive.org.au/t/regional-ancillary-suite-error-srtm-data-not-available-for-regions-above-60-deg/5796/5)
- If using BARRA land surface:
    - You need NCI project membership `ob53` to read BARRA files!
    - Don’t exceed 58 degrees south when using BARRA land surface, because BARRA ends at 57.97 degrees south.
    - Ensure the outer resolution which defines the driving aligns with the BARRA lat/lon co-ords at 0.11 intervals
- If your domain includes small islands in the Southern Ocean (e.g. Macquarie Island) this is a required work-around for small island vegetation canopy bug. See [here](https://forum.access-hive.org.au/t/aus2200-vegetation-fraction-ancil-creation-issues/1972/20) for more information.

- Guidelines for min/max numbers of processors.
    - Due to how the UM decomposes a domain into a patchwork of sub-domains which each run on a seperate processor, the number of overall points in your domain places a maximum of the number of processors you can use. See [here](https://forum.access-hive.org.au/t/issues-when-changing-domain-in-access-ram3/5943/5_ and [here](https://forum.access-hive.org.au/t/issues-when-changing-domain-in-access-ram3/5943/7)

- Check your ancillaries have no 'Nan' points (i.e. missing values) over land!  You can use this [notebook](https://github.com/21centuryweather/UM_configuration_tools/blob/main/notebooks/Check_UM_ancillaries.ipynb) to check.

** TO DO **

- Guide to interpreting typical model errors – what to look for in the log files and what they likely mean.
- Guidelines for spatial resolution and model selection (GAL9/RALP3P3)
- Include guide to where the log files are and what is in each type