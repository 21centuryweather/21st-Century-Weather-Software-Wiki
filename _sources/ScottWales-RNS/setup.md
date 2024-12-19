# Regional Nesting Suite

Another important component of Australian weather research is the 'Regional Nesting Suite' - RNS, designed to run high-resolution weather simulations for a relocatable domains over the Australian continent.

The suite was developed primarily by Scott Wales while working at CLEX - the Centre of Excellence for Climate Extremes. The suite is very similar to AUS2200 except that it usually describes two layers of nesting and is designed from smaller (and less computationally demanding) domains.

This documentation refers to earlier RNS suites `u-cy368` (for ancillary generation) and `u-cy369` (for atmospheric simulation). These suites are slightly older and have been replaced by `u-cy767` and `u-cy768`. These later suites are now maintained by Chermelle Engel at ACCESS-NRI, and updated documentation for these suites will be available soon.

In the interim, Scott's documentation will serve as a useful guide for those running high-resolution ACCESS atmospheric simulations using relocatable domains.

The original documentation can downloaded from https://github.com/scottwales/aus2200-training.

## Accounts

To run the tutorial suites you'll need a MOSRS Account so that you can download the suites and source code. Request an account from your [local sponsor](https://opus.nci.org.au/display/DAE/UK+Met+Office+environment+prerequisites)

You'll also need to be a member of the following NCI projects:

 * [`access`](https://my.nci.org.au/mancini/project/access): UM licensed software, modules and data
 * [`hr22`](https://my.nci.org.au/mancini/project/hr22): Rose and Cylc software
 * [`ki32`](https://my.nci.org.au/mancini/project/ki32): Source repository mirrors
 * [`ki32_mosrs`](https://my.nci.org.au/mancini/project/ki32_mosrs): MOSRS mirrors (requires membership of `ki32` and a MOSRS account)
 * [`hh5`](https://my.nci.org.au/mancini/project/hh5): Climate and Weather Conda environments
 * [`rt52`](https://my.nci.org.au/mancini/project/rt52): ERA5 mirror
 * [`zz93`](https://my.nci.org.au/mancini/project/zz93): ERA5-land mirror
 
Cylc at NCI is managed centrally, see [NCI's documentation](https://opus.nci.org.au/display/DAE/UK+Met+Office+Environment+on+NCI) for instructions on setting it up. Contact help@nci.org.au for assistance with using Cylc or the ARE platform. If you have trouble with the model itself you can ask questions on the [ACCESS Hive Forum](https://forum.access-hive.org.au/latest)

## VDI Session

Start up a VDI session at https://are.nci.org.au, using the settings:

```
Walltime:       4
Queue:          normalbw
Compute Size:   small
Project:        nf33
Storage:        gdata/access+gdata/hr22+gdata/ki32+gdata/hh5+gdata/rt52+gdata/zz93
```

Once your VDI session has started open a terminal and load the Rose/Cylc environment with:
```
module purge
module use /g/data/hr22/modulefiles
module load pbs cylc7
mosrs-auth
```
