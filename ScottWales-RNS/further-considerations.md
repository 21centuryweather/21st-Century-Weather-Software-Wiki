# Further Considerations

## Parameters to change with resolution

When using RAL3 science the only parameters that need to change with resolution are the model timestep and the radiation timesteps, both in the 'configs' panel. The model timestep must evenly divide all other timesteps, including the radiation timestep, LBC cycle time and the cycle length.

## Ancillary Data Resolution

At very high resolutions the resolution of source ancillary datasets can start to become a concern. Check the output of the ancillary suite to make sure that there are no artifacts that could impact the model.

Normally you want to use the SRTM orography for regional runs, the default orography dataset is very coarse.

## Boundary Effects

There will always be artifacts around the edge of a regional run where the LAM model merges with the lateral boundary conditions. These boundary effects need to be removed from any analysis. Consider a trial run to check the size of the effect with your configuration to make sure the boundary effects do not cover your area of interest.

## Variable Resolution Grids

It's possible to define a grid where the outer rim is at a lower resolution than the inside, which can reduce the need for intermediate nests. This is used in the UKV and ACCESS-A grids. There is information on MOSRS for [creating variable resolution grids](https://code.metoffice.gov.uk/trac/rmed/wiki/suites/RAS/variablegrid).

## Decomposition and IO servers

When setting up a new domain it's important to consider the model performance. You should perform some trial experiments with different processor decompositions in order to find the most efficient setup. Generally more processors will make a job run faster, but the more you add the less effective each increase will be.

Large grids will also take up a lot of time just writing out data. In this case you can enable the model's IO servers, dedicated nodes for handling writing out data to disk.