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


It contains all the settings required to build version 13.5 of the UM.

For example, if you wanted to add information to the Fortran compiler flag, you could type something like
```
fcflags_overrides=-march=broadwell -axSKYLAKE-AVX512,CASCADELAKE,SAPPHIRERAPIDS
```
to give your UM better performance on other Intel architecture.

You can also access these features in `rose edit`. Select the `fcm_make_um` task on the left panel, wait for the option menu to appear, and then click `env -> Advanced compilation`

The task also contains default settings for optimisation levels. These are available in the `env -> Basic Compilation` menu.



Let's trigger this task.

After completion you will notice that your directory `~/cylc-run/<suite-id>/share/fcm_make_um` exists and contains various files and directories.

The source code for the UM (and its sub components, such as JULES) now lie in `~/cylc-run/<suite-id>/share/fcm_make_um/extract`

So if you wish to add or change any files in the UM source tree, make them in 
`~/cylc-run/<suite-id>/share/fcm_make_um/extract/um/src`

So this task does not build the UM, it just downloads the source from the external repositories hosted by the UK Met Office.

Check that your configuration options (e.g. compilation flags) are reflected in the contents of `~/cylc-run/<suite-id>/share/fcm_make_um/fcm-make2.cfg`. 

For example, if you selected to compile with `debug (debug)` optimisation level, the following entries should exist in your `fcm-make2.cfg` file
```
build.prop{class, fc.flags} = -i8 -r8 -mcmodel=medium      -std08  -g -traceback  -assume nosource_include -O2 -fp-model precise -qopenmp      
```

:::{note}

## fcm_make2_um

Once you've made the required changes to your source files and compilation flags, you can then compile your code by triggering `fcm_make2_um`.

