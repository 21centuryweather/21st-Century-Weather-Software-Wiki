# Suite configuration

The private repository `https://github.com/21centuryweather/access-am3-configs` includes two branches to run a low (n96) and a high (n512) resolution: 

* `dev-n512-aeroclim`
* `dev-n96e-aeroclim`

The main differences with the alpha release version, it that this configuration runs with climatological aerosols. 

## Initial conditions

Both suites are prepared to run with `n96` restarts. In the case of `n512` the reconfiguration step will interpolate all the necessary variables. 

Also, both suite include a `fix_cable_restart` step to populate cable variables after the reconfiguration. While this is not necessary for `n96`, is included to match the configuration of the high resolution branch. 

Restart files: available for every January from 1983 to 2009 at `/g/data/gb02/public/AM3/restartdumps/n96e`. 

To change the restart used edit the file `site/nci_gadi.rc`:

```
{% set AINITIAL = '/path/to/restart/deva.da20070101_00' %}
```

## Ancilliaries

Ancillaries for n96 and n512 where generated using the suite `u-dj813` (copy in `/g/data/gb02/public/AM3/ancils/u-dj813_om3-025deg/`). 

Manual fixes:
* Veg frac. For n512 it requires a manual fix to normalize values and remove NAs. 

## Optimization

The IO Server was activated for both suites. 

| Resolution  | NPROCX | NPROCY | IO | Threads | Processors | nodes | IO_tasks |
|-------------|--------|--------|----|---------|-----------|-------|----------|
| n512        |     32 |     32 | 80 |       2 |      2208 |    46 |        8 |
| n96         |     24 |     24 | 48 |       2 |      1248 |    26 |        4 |

## Output variables

The variables saved are listed [here](https://docs.google.com/spreadsheets/d/11S5hY54BPHzoi2v6KYZeUeVo-76Q8QLqwKfjJCPuCrI/edit?gid=1519343531#gid=1519343531). The different groups of variables or "Packages" can be activated/deactivated from the gui.

> Currently the COPS, 2D Extra and 3D Extra packages are not active because they haven't been fully tested. They can be activated from the GUI in `UM/namelist/Model Input and Output/STASH Request and Profiles/STASH request` (upper right corner drop-down menu).

Things to consider when activating these packages:
* Output streams (pp9 and pp10) may require more headers to allocate all the variables.
* If field files are to big (more than 100gb) the netcdf conversion will probably fail. My recommendation is to switch from monthly output to daily (to create 30 small files instead of a big one). This is done also in the pp configuration.
* To save some space we may want to deactivate the 3D ERA5 Diagnostics package to save some space. 

## Resources

Resources per simulated month with the current configuration and output 

| Resolution | KSU | Wall time | Output  (netcdf)|
|---------------|:---------------:|:---------------:|:---------------:|
| `n96` |0.462 |00:11:05  | 8.8 GB|
| `n512` | 11.7   | 02:40:00  |  226 GB  |

