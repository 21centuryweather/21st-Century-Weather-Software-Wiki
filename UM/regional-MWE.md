# Running the Regional Nesting Suite

Now we have generated some ancillaries, we can run the UM forecast model!

Let's checkout a 'minimum working example' for the Regional Nesting suite:
```
$ cd ~/roses/
$ git clone git@github.com:21centuryweather/RNS_minimum_working_example.git
```

This is a specially-developed suite designed to run with the Regional Ancillary suite we ran before. All extra logic and features related to operational forecasting, UM source code compilation, atmospheric chemistry, UK-specific features etc. have been removed. 

Because of this
- The suite will not render correctly in the `rose edit` GUI, but
- The `rose-suite.conf` is simple enough that we can examine it directly.

## Mapping the ancillary directories to forecast regions

:::{important}
The following is a very important concept within the UM Regional Modelling suites.
:::

Recall for the RAS we defined
- One ancillary region (Lismore-Tiny)
- Two resolutions nested in this region
  - The outer resolution 'era5'
  - An inner resolution 'd1100'

These are labelled in the RAS `rose-suite.conf` as
```
rg01_name="Lismore-tiny"
...
rg01_rs01_name="era5"
...
rg01_rs02_name="d1100"
```
Now if let's examine the RNS `rose-suite.conf`. It contains:
```
dm_nregns=1
dm_name="ec"
..
rg01_name="Lismore-Tiny"
..
rg01_nreslns=1
```
In the RNS (i.e. the forecasting suite) the 'driving model' ancillaries are a separate category from the forecast regions. Remember our 'driving model' is used to provide lateral boundary conditions and initial conditions **ONLY!**. 

So the RNS only starts counting resolutions and regions for domains where the UM forecast model is run.

The table below summarises this mapping between the RAS ancillaries and the RNS domain definitions. This mapping applies for a single region.

| Ancillary Suite | Nesting suite | Domain type    |   |   |
|-----------------|---------------|----------------|---|---|
| rg01_rs01       | dm            | Driving model  |   |   |
| rg01_rs02       | rg01_rs01     | Forecast model nest 1 |
| rg01_rs03       | rg01_rs02    | Forecast model nest 2 |
| rg01_rs04       | rg01_rs03    | Forecast model nest 3|
| ... | ... | ... |
| rg01_rs0[N] | rg01_rs0[N-1] | Foremost model nest N-1 |

For our minimum working example we are only running with N=2 ancillary domains, so we are only running a single forecast domain(N=1)

:::{important}
Keeping track of which ancillary files define your first Forecast model nest will be very important when considering your choice of land-surface dataset set. This will be touched on later.
:::

In the RNS `rose-suite.conf` file, the following definitions are used to define the location of the ancillary files for the driving model and first forecast model nest:
```
dm_ec_lam_ancil_dir="/scratch/$PROJECT/$USER/cylc-run/RAS_minimum_working_example/share/data/ancils/Lismore-tiny/era5"
rg01_rs01_ancil_dir="/scratch/$PROJECT/$USER/cylc-run/RAS_minimum_working_example/share/data/ancils/Lismore-tiny/d1100"
```
These directories should have been created and populated with ancillary data when you ran the RAS suite. 

**Make sure they still have data in them!**

:::{note}
- The RNS and RAS assume you are running on your default `gadi` project which is specified in the file `~/.config/gadi-login.conf`. If the value of `PROJECT` in this file has changed, you might have to manually alter the definition of the above ancillary directory files.
- NCI routinely clean the contents of your `/scratch/$USER` directory every 100 days. So your ancillaries may get deleted if you haven't touched that directory recently.
:::

