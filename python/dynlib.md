---
title: Installing dynlib at NCI/gadi
layout: post
tags: python, fortran
---

Dynlib is meteorological analysis libary developed at University of Bergen. The central github repository is here : <https://git.app.uib.no/Clemens.Spensberger/dynlib>

Documentation for the library is available here : <https://folk.uib.no/csp001/dynlib_doc/intro.html>

# Dependencies

A list of dependencies to build the library is available here : <https://folk.uib.no/csp001/dynlib_doc/pkg.html>

Some of the analysis tools are compiled in Fortran, and the python tool `f2py` is used to link the Fortran modules to a python wrapper. Details about `f2py` are available at : <https://numpy.org/doc/stable/f2py/>

The main Fortran dependencies are:
- LAPACK (<https://netlib.org/lapack/>) -  libraries to solve linear equations.
- BLAS (<https://www.netlib.org/blas/>) - libraries for linear algebra.

These two libraries are available on NCI/gadi, see <https://opus.nci.org.au/pages/viewpage.action?pageId=248840668>

By loading the appropriate modules specified in that link, we can compile the `dynlib` fortran source. 

#  Installation

From a directory on gadi, type

{% highlight bash %}
$ git clone https://git.app.uib.no/Clemens.Spensberger/dynlib.git
Cloning into 'dynlib'...
remote: Enumerating objects: 4471, done.
remote: Counting objects: 100% (274/274), done.
remote: Compressing objects: 100% (105/105), done.
remote: Total 4471 (delta 224), reused 169 (delta 169), pack-reused 4197 (from 1)
Receiving objects: 100% (4471/4471), 1.86 MiB | 1010.00 KiB/s, done.
Resolving deltas: 100% (3055/3055), done.
Updating files: 100% (286/286), done.
{% endhighlight %}

You have now downloaded the latest version of the dynlib source files.  

These instructions will install dynlib using a standard NCI python environment, as I had problems linking the fortran modules with python using the CLEX ```analysis3``` conda environment.

Let's load the required modules. 
{% highlight bash %}
$ module load python3/3.10.0
Loading python3/3.10.0
  Loading requirement: intel-mkl/2021.4.0
{% endhighlight %}
Loading later python3 versions (e.g. python3.12.1) will fail because the library uses the deprecated `disutils` package for installation. 

Note this loads the required intel libraries required to compile the required fortran modules. You will need to upgrade the default version `gfortan` at NCI/gadi (8.5.0) so that it recognises the `-fallow-argument-mismatch` compiler argument provided in the file `/ext/spherepack3.2/make.inc` which passes parameters to the `spherepack` `Makefile`.
{% highlight bash %}
module load gcc/10.3.0
{% endhighlight %}
Now we need to edit the `compile` script to 
- Provide the python version (3)
- follow the NCI guidelines to load the LAPACK and BLAS libraries.

Edit line 13 of `compile` from
{% highlight bash %}
PY_VER=""
{% endhighlight %}

to 
{% highlight bash %}
PY_VER="3"
{% endhighlight %}

Edit line 60 from
{% highlight bash %}
LIBS="-L/usr/lib -L/usr/local/lib -L$(pwd)/ext/spherepack3.2/lib -lblas -llapack -lspherepack
{% endhighlight %}
to
{% highlight bash %}
LIBS="-L/usr/lib -L/usr/local/lib -L$(pwd)/ext/spherepack3.2/lib -lmkl_gf_lp64 -lmkl_core -lmkl_gnu_thread -lgomp -lm -lspherepack"
{% endhighlight %}

###  NOTE : You must now commit your local changes to your local copy of repository before attempting install!
{% highlight bash %} 
$ git commit .
{% endhighlight %} 

Once you have committed your local changes, you can then begin compiling and installing the libraries.

This python file `setup.py` will run the `compile` script that we just edited previously. This script will autocompile the `spherepack` modules, before it compiles and links the rest of the fortran code. Because we are using are standard NCI python environment, we will have to provide an install location as we don't have write permissions to `/apps/python3/3.10.0/lib/python3.10/site-packages/`. We can specify the install location using the 'user' or `prefix` command-line argument,e.g.
{% highlight bash %} 
$ python3 setup.py install --user
{% endhighlight %} 
or
{% highlight bash %} 
$ python3 setup.py install --prefix=FULL_PATH_TO_YOUR_LIBRARY_INSTALL
{% endhighlight %} 

For testing, you might want to use `--prefix` to specify a path on `/g/data/gb02`. The `--user` argument will install in the default location `~/.local/lib/python3.10/site-packages/`.

# Loading the module

To load the module within a python session (e.g. using the python3/3.10.0 environment we used to compile and install `dynlib` library) we can use the following (assuming we installed it with the `--user` option):
{% highlight python %} 
Python 3.10.0 (default, Nov 16 2021, 09:41:50) [GCC 8.4.1 20200928 (Red Hat 8.4.1-1)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import dynlib
>>> from dynlib import dynfor
>>> dir(dynfor)
['__doc__', '__f2py_numpy_version__', '__file__', '__loader__', '__name__', '__package__', '__spec__', '__version__', '_dynfor_error', 'config', 'consts', 'derivatives', 'detect', 'detect_lines', 'detect_rwb_contour', 'diag', 'diag_tend', 'ellipse', 'interpol', 'sphere', 'stat', 'tend', 'thermodyn', 'utils']
>>> print(dynfor.derivatives.ddx.__doc__)
res = ddx(dat,dx,dy,[nx,ny,nz])

Wrapper for ``ddx``.

Parameters
----------
dat : input rank-3 array('d') with bounds (nz,ny,nx)
dx : input rank-2 array('d') with bounds (ny,nx)
dy : input rank-2 array('d') with bounds (ny,nx)

Other Parameters
----------------
nx : input int, optional
    Default: shape(dat,2)
ny : input int, optional
    Default: shape(dat,1)
nz : input int, optional
    Default: shape(dat,0)

Returns
-------
res : rank-3 array('d') with bounds (nz,ny,nx)
{% endhighlight %} 

If we used the `--prefix` option we will have to tell python where to find our libraries. One way to do this is:
{% highlight python %} 
>>> import sys
>>> sys.path.append('FULL_PATH_TO_YOUR_LIBRARY_INSTALL')
>>> sys.path
['', '/apps/python3/3.10.0/lib/python310.zip', '/apps/python3/3.10.0/lib/python3.10', '/apps/python3/3.10.0/lib/python3.10/lib-dynload', '/home/548/pag548/.local/lib/python3.10/site-packages', '/apps/python3/3.10.0/lib/python3.10/site-packages', '/apps/python3/3.10.0/lib/python3.10/site-packages/numpy-1.21.4-py3.10-linux-x86_64.egg', 'FULL_PATH_TO_YOUR_LIBRARY_INSTALL']
{% endhighlight %}

Note python will search for `dynlib` according to the order of the paths in the `sys.path` list. If you wish to add a new path to the start of this list (index 0) so it is searched first, you can use
{% highlight python %} 
>>> sys.path.insert(0,'FULL_PATH_TO_YOUR_LIBRARY_INSTALL')
{% endhighlight %}

Or you can insert your new path at the beginning of command-line environment variable `PYTHONPATH`, e.g
{% highlight bash %} 
$ export PYTHONPATH="FULL_PATH_TO_YOUR_LIBRARY_INSTALL:$PYTHONPATH"
{% endhighlight %}

You can also load the module from within the CLEX `analysis3` `conda` environment.
{% highlight bash %} 
$ module use /g/data/hh5/public/modules
$ module load conda/analysis3
$ ipython
{% endhighlight %} 
{% highlight python %} 
Python 3.10.14 | packaged by conda-forge | (main, Mar 20 2024, 12:45:18) [GCC 12.3.0]
Type 'copyright', 'credits' or 'license' for more information
IPython 8.26.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: import dynlib

In [2]: from dynlib import metio

In [3]: ?dynlib
Type:        module
String form: <module 'dynlib' from '/home/548/pag548/.local/lib/python3.10/site-packages/dynlib/__init__.py'>
File:        ~/.local/lib/python3.10/site-packages/dynlib/__init__.py
Docstring:   <no docstring>

In [4]: dynlib.dynfor.derivatives.ddx
Out[4]: <fortran object>

In [5]: ?dynlib.dynfor.derivatives.ddx
Call signature: dynlib.dynfor.derivatives.ddx(*args, **kwargs)
Type:           fortran
String form:    <fortran object>
Docstring:     
res = ddx(dat,dx,dy,[nx,ny,nz])

Wrapper for ``ddx``.

Parameters
----------
dat : input rank-3 array('d') with bounds (nz,ny,nx)
dx : input rank-2 array('d') with bounds (ny,nx)
dy : input rank-2 array('d') with bounds (ny,nx)

Other Parameters
----------------
nx : input int, optional
    Default: shape(dat,2)
ny : input int, optional
    Default: shape(dat,1)
nz : input int, optional
    Default: shape(dat,0)

Returns
-------
res : rank-3 array('d') with bounds (nz,ny,nx)

In [7]: from dynlib.metio import datasource
{% endhighlight %} 

# Test scripts 

The repository comes with a variety of unit tests (located in the `tests` directory) and example-scripts (located in `examples'). For a variety of reasons, these tests and examples will not work out of the box on gadi:
-  Sample data paths are assumed to exist on a drive somewhere in Bergen, e.g. 
{% highlight python %} 
$ python test_dynlib.py 
Traceback (most recent call last):
  File "/g/data/gb02/pag548/dynlib/test_dynlib.py", line 17, in <module>
    dat, grid = get_instantaneous([(plev, 'u'), (plev,'v'), ], timeinterval)
  File "/g/data/gb02/pag548/dynlib/dynlib_env/lib/python3.10/site-packages/dynlib/metio/datasource.py", line 939, in get_instantaneous
    grid = get_static(**static_kwargs)
  File "/g/data/gb02/pag548/dynlib/dynlib_env/lib/python3.10/site-packages/dynlib/metio/erainterim.py", line 111, in get_static
    fo, oro = metopen(conf.staticfile, 'oro', verbose=verbose, no_dtype_conversion=no_dtype_conversion, 
  File "/g/data/gb02/pag548/dynlib/dynlib_env/lib/python3.10/site-packages/dynlib/metio/datasource.py", line 473, in metopen
    raise ValueError('%s.* not found in any data location. \n'
ValueError: ei.ans.static.* not found in any data location. 
Tried the following (in order):
      .
      /Data/gfi/share/Reanalyses/ERA_INTERIM/6HOURLY
      /Data/gfi/users/local/share
      /Data/gfi/users/csp001/share
{% endhighlight %} 
- It assumes that functions exist in some of the source files. (e.g. `lib/settings.py` doesn't appear to contain a function `get_active_context`)  Also, the requirement for `pygrib` isn't in the documentation), e.g.
{% highlight python %} 
(dynlib_env) [pag548@gadi-login-05 dynlib]$ python tests/test_plot.py 
Traceback (most recent call last):
  File "/g/data/gb02/pag548/dynlib/tests/test_plot.py", line 11, in <module>
    from create_ref_plots import path, gfile, obfile, odfile, req_levs, ofile as reffile, conf, make_plot_list
  File "/g/data/gb02/pag548/dynlib/tests/create_ref_plots.py", line 7, in <module>
    from lib.settings import get_active_context
ImportError: cannot import name 'get_active_context' from 'lib.settings' (/g/data/gb02/pag548/dynlib/tests/lib/settings.py)
(dynlib_env) [pag548@gadi-login-05 dynlib]$ python tests/test_func.py 
Traceback (most recent call last):
  File "/g/data/gb02/pag548/dynlib/tests/test_func.py", line 9, in <module>
    from create_ref_data import path, gfile, obfile, odfile, diagnostics, _prepare_args
  File "/g/data/gb02/pag548/dynlib/tests/create_ref_data.py", line 4, in <module>
    import pygrib
ModuleNotFoundError: No module named 'pygrib'
{% endhighlight %} 
- Missing files and functions altogether, e.g.
{% highlight bash %} 
 python tests/test_func.py 
Traceback (most recent call last):
  File "/g/data/gb02/pag548/dynlib/tests/test_func.py", line 9, in <module>
    from create_ref_data import path, gfile, obfile, odfile, diagnostics, _prepare_args
  File "/g/data/gb02/pag548/dynlib/tests/create_ref_data.py", line 12, in <module>
    import lib.humidity
ModuleNotFoundError: No module named 'lib.humidity'
{% endhighlight %} 

## Workarounds

The location of sample data used for the tests and examples scripts can be found in the following scripts:
- `lib/settings.py` : See the default configuration for the class `settings_obj`
- `lib/metio/erainterim.py` : See the values of `'datapath'` in the `settings_obj` configuration class.
- `examples/example_attribute.py` : Several data paths are hard-coded in the example scripts.

## Depedency on basemap

The python module `basemap` is required by many of the `dynlib` plotting routines. The user will have to install this module themselves as it is not included in the standard NCI/gadi environments, or in the CLEX `analysis3` environments.

A recommended way to install `basemap` is to install it on top of your existing `analysis3` environment using a *virtual environment*, following the process available here : <https://climate-cms.org/posts/2024-06-05-mixing-python-envs.html>. 

To replicate this example for `basemap` using a virtual environment named *dynlib_env* created from inside the `analysis3` environment, execute the following:
{% highlight bash %} 
$ python3 -m venv dynlib_env --system-site-packages
$ source dynlib_env/bin/activate
{% endhighlight %} 

The new environment `dynlib_env` will contain all the existing `analysis3` modules as well as `basemap`.
{% highlight bash %} 
(dynlib_env) $ ipython
{% endhighlight %} 
{% highlight python %} 
/g/data/hh5/public/apps/miniconda3/envs/analysis3-24.04/lib/python3.10/site-packages/IPython/core/interactiveshell.py:937: UserWarning: Attempting to work in a virtualenv. If you encounter problems, please install IPython inside the virtualenv.
  warn(
Python 3.10.14 | packaged by conda-forge | (main, Mar 20 2024, 12:45:18) [GCC 12.3.0]
Type 'copyright', 'credits' or 'license' for more information
IPython 8.26.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: import xarray as xr

In [3]: import pygrib

In [4]: import metpy.calc as mpcalc

In [5]: from mpl_toolkits.basemap import Basemap

In [6]: import netCDF4
{% endhighlight %} 
