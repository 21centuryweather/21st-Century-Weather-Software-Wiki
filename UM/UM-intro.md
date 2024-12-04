# The Unified Model

The ACCESS modeling suite is reliant on the UK Met Office’s Unified Model (UM) for atmospheric simulations. 

The UM is a collection of Fortran-based executables built to simulate the future state of the atmosphere. It is designed to interface with land/surface and sea-surface data.

The model is so named (i.e. "Unified") because it contains a single set of code that can be applied across a diverse range of length and timescales; from urban weather forecasts at grid resolutions of a few hundred metres, to global climate model projections extending into next century at grid resolutions of hundreds of kilometres.

You can access more detailed information about the UM here : 
https://21centuryweather.github.io/UM_summary_docs/introduction.html

For now, let's concentrate on running a very simple test suite, a 'minimal working example' to demonstrate how we can run the UM on `gadi` using a `rose/cylc` suite.

## Checking out a UM suite ##

If you need to, repeat the steps of the previous tutorial to authenticate your MOSRS credentials. Then, checkout suite `u-dl058` using
```
rosie co u-dl058
```
This suite was initially developed by scientists at the Bureau of Meteorology to research 'coupled' Numerical Weather Prediction using separate atmospheric and ocean models. For this tutorial, we have removed all `rose/cylc` logic related to coupling so the suite performs a simple atmospheric-only global simulation and low resolution.

## Examining the suite ##

The suite contains all the usual files you've become accustomed too inside a `rose/cylc` suite:
- A `rose-app.conf` file to control the master configuration of the suite.
- A `suite.rc` file to control the task flow
- An `app` directory to house `.conf` files for each application task.

Let's step through each task within this suite so we understand at a basic level, how the UM works on `gadi`.

### Master configuration ###

Let's quickly run through the master configuration file : `~/roses/u-dl085/rose-suite.conf`

This contains a list of `bash` environment variables which describe the following:
- `ANCILDATA` : Ancillary data directory
- `ARCHDIR` : Output archive directory
- `ARCHIVE_6HR` : Archiving temporal frequency
- `ATM_PPN` : Number of processors for atmospheric tasks
- `BUILD_UM` : Flag to compile our own UM executable
- `CALENDAR` : Calendar type (important for long climate simulations)
- `CASEDATA` : Directory of case study input files
- `COMPUTE_ACCOUNT` : Your gadi `PROJECT` used for resouce allocation
- `COMPUTE_HOST` : Name of supercomputer 
- `COMPUTE_QUEUE` : PBS job queue used for simulations
- `CONFIG_MODULE_NAME` : Extra module configuration data
- `CORE` : Type of CPU used in supercomputer environment
- `DATES` : List of dates to run simulations for
- `EXTRACT_HOST` : operating system type
- `HOUSEKEEP`: Flag to set housekeeping tasks
- `OCEANDIR` : Location of Ocean data
- `PPN` : Number of processors (generic)
- `QUEUE_PARALLEL` : PBS job queue type
- `RUNTYPE` : Flag for 'coupled' (modeling both the ocean and atmosphere) or 'uncoupled' (atmosphere only) simulation. 
- `RUN_METDB_ODB` : Option to use the UK Met Office Observation Database
- `SITE` : String to denote the supercomputer type
- `STOCHASTIC`: Flag for stochastic atmospheric physics
- `TASKLENGTH` : Data for UM task control
- `TASK_RUN` : Data for UM task control
- `TASK_TESTS` : Flag for UM task control
- `UMDIR` : Master directory for UM files
- `UM_RESOLUTION` : Global atmospheric resolution 
- `USE_COMORPH` : Flag for CoMorph convection scheme
- `USE_DOUBLE_PREC` : Flag for numerical precision
- `XIOS_NPROC` : Processors used for XIOS source-code compilation
- `xios_path` : Directory of `xios` libraries used by intel compilers. XIOS is an XML input/output server used to handle coupling between the UM atmosphere and the NEMO ocean model.

### Task flow ###

Let's walk through the `suite.rc` file to determine the task flow. 

Before we get to the `graph` section, there are several `jinja` macros (defined using `{% %}` constructs) the do the following:
- Set an alias for the rose `TASK_RUN_COMMAND`
- load the required `modules` on gadi
- Set some run-time options based on entries in the `rose-suite.conf` file, including:
    - Numerical precision 
    - Archiving frequency
    - 'CoMorph' convection scheme
    - Stochastic physics settings
- Set aliases for dates based on the input `DATES`

