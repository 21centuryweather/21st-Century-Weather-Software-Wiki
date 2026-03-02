# Configuring an rAM3 `Flagship' run

The Centre of Excellence for Weather of the 21st Century has a set of `Flagship' experiments to provide deeper insights into weather phenomenon via hi-resolution modelling experiements.

These Flagship experiments have a different configuration than the standard NRI rAM3 suite to enable very large domains at high resolutions to be run efficiently.

## Checking out the rAM3 Flagship suite

When you checkout the Regional Nesting suite with the following command
```
rosie checkout u-by395/nci_access_ram3
```
You are retrieving the branch `nci_access_ram3` from the UK Met Office's `roses` `trac` repository.

:::{note}

`trac` is a version control software manager that predates `github`, but works in a similar way. It was built around the 'Subversion' software repository system (or `svn`), which also predates `git`.  

:::

You can log into the UK Met Office `trac` repository with your MOSRS credentials and see the suite here:

https://code.metoffice.gov.uk/trac/roses-u/browser/b/y/3/9/5

Note there are many branches of `u-by395`. These are all variants of the UK Met Office Regional Atmosphere Nesting suite, used by UK Met Office partners all over the world.

The standard NRI suite is here:
```
https://code.metoffice.gov.uk/trac/roses-u/browser/b/y/3/9/5/nci_access_ram3
```
The Flagship configuration of the `nci_access_ram3` is built on a seperate branch of `u-by395`. You can view it here:

https://code.metoffice.gov.uk/trac/roses-u/browser/b/y/3/9/5/Flagship

You can use `trac` to view the differences between the Flagship and the standard NRI suite by clicking [here](https://code.metoffice.gov.uk/trac/roses-u/changeset?old_path=%2Fb%2Fy%2F3%2F9%2F5%2FFlagship&old=348215&new_path=%2Fb%2Fy%2F3%2F9%2F5%2Fnci_access_ram3&new=348215).

These changes include:
- Using a UM executable compiled for NCI's 'sapphire rapids' architecture
- Open MP multi-threading is enabled.
- Using the UM's 'I/O server' to enable asynchronous parallel disk reading and writing - essential when deploying large numbers of CPUs across large domains
- The ability to use additional cloud outputs
- Submitting jobs to NCI's 'sapphire rapids' queue
- Setting 'striping' on certain output directories to enable parallel writing of individual files.

Note by default the Flagship is configured to use the following configuration:
- A ERA5 atmospheric driving model
- BARRA land surface initialisations
- Three seperate atmospheric nests (or resolutions)

You will want to change these configuration in the `rose-suite.conf` file (either manually, or graphically using `rose edit`) 

:::{note}

Thanks to Dale Roberts who developed these optimisations for the AUS2200 suite

:::

How do we access these features to run our own experiement?

## Checking out the rAM3 Flagship suite

Checking out the rAM3 Flagship suite is as easy as typing
```
rosie checkout u-by395/Flagship
```

BUT : if you already have the default NRI suite installed, this will fail with the message:
```
[FAIL] /home/548/pag548/roses/u-by395: already exists
```

So how can we work around this?  There are several strategies.

### Copying the default RNS suite to a different directory (easiest)

If we type
```
$ mv ~/roses/u-by395 ~/roses/u-by395-default
```
(or similar) this will copy your exiting RNS suite configuration to another directory. You can then checkout the `u-by395/Flagship` suite into your `~/roses` directory without any problems.

This method will also allow you to copy the nesting and resolution settings in your `rose-suite.conf` file to the Flagship.

If you want to revert back to your original RNS suite you could then type something like
```
$ mv ~/roses/u-by395 ~/roses/u-by395-Flagship
$ mv ~/roses/u-by395-default ~/roses/u-by395
```
Another option could be to set up symbolic links in your `~/roses` directory and have a link called `u-by395` that points to different RNS suites (i.e. the default and a Flagship)

:::{warning}

The `cylc` engine can only generate one set of ouputs from a given `rose` suite ID. So if you use the above method, note that your outputs from the Flagship branch of `u-by395` will overwrite whatever exists in the `~/cylc-run/u-by395` directory and vice-versa.

To prevent conflicts between the two different suites, you might want to clean this directory first by typing
```
$ rose suite-clean
```
before running from a new branch.

:::


### Switching branches (harder but more interesting)

We can keep our `u-by395/nci_access_ram3` branch on disk, and use `fcm` (which is the UK Met Offce wrapper for `svn`) to switch between branches. This is similar to using `git branch`.

Firstly, submit your MOSRS credentials via the `mosrs-auth` script so you can access the `svn` repositories.
```
$ mosrs-auth
```
Then navigate to your `u-by395` directory and type
```
$ cd ~/roses/u-by395
$ fcm branch-info
```
You should generate output along the lines of:
```
$ fcm branch-info
URL: https://code.metoffice.gov.uk/svn/roses-u/b/y/3/9/5/nci_access_ram3

<information on the branch author and revision>
--------------------------------------------------------------------------------
Branch Parent: https://code.metoffice.gov.uk/svn/roses-u/b/y/3/9/5/trunk@325101
```
We can use the above syntax to 'switch branches' between the default branch and the Flagship branch by typing
```
fcm sw https://code.metoffice.gov.uk/svn/roses-u/b/y/3/9/5/Flagship
```
This will pull changes from the central UK Met Office `roses` repository and update the local files on disk.

Let's check the output.
```
$ fcm branch-info
URL: https://code.metoffice.gov.uk/svn/roses-u/b/y/3/9/5/Flagship
Repository Root: https://code.metoffice.gov.uk/svn/roses-u
Revision: 348215
```
Ok you've successfully switched branches.

:::{tip}

You can find information on how to use `fcm` to manage your `rose` suites here:

https://metomi.github.io/fcm/doc/user_guide/code_management.html

https://metomi.github.io/fcm/doc/user_guide/command_ref.html

https://jules-lsm.github.io/tutorial/bg_info/tutorial_julesrose/jr_mqrg.html

The UK Met Office is transitioning from `svn`, `fcm` and `trac` to `git` and `github`. But if you're an advanced user, you might find it worthwhile learning some basic around the current system. This will give you the ability to merge different revision numbers, build and commit your own branches.

:::