# Custom UM builds for rAM3

The Regional Nesting Suite (RNS) contains the ability to build a custom UM executable. Let's work through the steps required to compile a local exectuable for your own needs.

## Setting the build flag

In the `rose edit` GUI, change the `BUILD_MODE` option in the `General Run options` to `Build new executable`.

You can also change 
```
BUILD_MODE="new"
```
in `rose-suite.conf`.

You may need to check the definition of `HPC_HOST` in `~/roses/<suite-id>/site/nci-gadi/suite-adds.rc`
from
```
{% HPC_HOST = ‘localhost’ % }

```
to
```
{% HPC_HOST = ‘gadi.nci.org.au’ % }
```
Now run your suite hold all tasks.
```
$ rose suite-run -- --hold
```
You will notice a new family of tasks in the GUI: `BUILD`. This task family contains two tasks : `fcm_make_um` and `fcm_make2_um`.

Let's see these how tasks work.

## fcm_make_um

If you look at the `rose` application configuration file for this task:
```

It contains all the settings required to build version 13.5 of the UM.

For example, if you wanted to add information to the Fortran compiler flag, you can add information to 

## fcm_make2_um

If you want to al