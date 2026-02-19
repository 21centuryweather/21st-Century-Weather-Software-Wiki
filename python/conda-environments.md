(content:conda-env)=
# Building custom python environments on top of `conda/analysis3`

<script src="https://cdn.jsdelivr.net/gh/atteggiani/animated-terminal/animated-terminal.min.js" defer></script>


This section will guide you through the steps to create a conda environment on top of `conda/analysis3` and is based on the materials developed byt the CLEX team. If you are interested in learning how to use the `conda/analysis3` environment, please refer to the [Intro to Python](content:intro-python) section.

## Introduction

The conda analysis environments in the `xp65` project are a community-led approach to create a central 
python environment specifically for the analysis of climate and weather data. As such, these environments
contain a wide range of packages curated by ACCESS-NRI that can fulfil the requirements of most 
analysis workflows. 

You can request ACCESS-NRI to install additional packages ([check this post for more details](https://forum.access-hive.org.au/t/requesting-new-software-packages-on-nci-systems/2834)). Adding or not new packages will depend on the compatibility with the existing packages.

If they are unable to include the package you need in the conda environment, that doesn't mean that you need to create a new conda environment
from scratch to use that package. 

## `pip install --user`

Lets use the [xWMB](https://github.com/hdrake/xwmb) package as an example.
There is no conda distribution for this package, it can only be installed by `pip` from a git repository. This
package also has an extensive list of dependencies, most of which are available in the `conda/analysis3` environment, so using the `analysis3` 
environment as a base will save quite a lot of disk space.
 
The simplest approach is to load the `conda/analysis3` environment and `pip install --user` the xWMB package.
<terminal-window typingDelay=30 lineDelay=200>
    <terminal-line data="input">module use /g/data/xp65/public/modules</terminal-line>
    <terminal-line data="input">module load conda/analysis3</terminal-line>
    <terminal-line data="input">pip install -\\\-user git+https://github.com/hdrake/xwmb.git@main</terminal-line>
    <terminal-line>Collecting git+https://github.com/hdrake/xwmb.git@main</terminal-line>
    <terminal-line>  Cloning https://github.com/hdrake/xwmb.git (to revision main) to /scratch/v45/dr4292/tmp/pip-req-build-qox565pe</terminal-line>
    <terminal-line>  Running command git clone -\\\-filter=blob:none -\\\-quiet https://github.com/hdrake/xwmb.git /scratch/v45/dr4292/tmp/pip-req-build-qox565pe</terminal-line>
    <terminal-line>  ...</terminal-line>
    <terminal-line>Successfully installed regionate-0.0.1 sectionate-0.2.1 xbudget-0.1.0 xgcm-0.8.2.dev15+g7492277 xwmb-0.1.1 xwmt-0.1.1</terminal-line>
</terminal-window>

This is sufficient in certain cases, for instance, quickly testing user code, or when long-term
reproducibility is not necessary. The downside with this approach is that it can lead to problems in the future. First of all, everything will be installed in your `/home` directory, which has a small quota and can easily be filled by local `pip` installations. Another consideration is that the `analysis3` module
is updated every month as new environments are released. For example, if you load `conda/analysis3` today,
you'll get the `conda/analysis3-26.01` environment. If you were to load the same module a couple of months from
now, you'll get the `conda/analysis3-26.04` environment. As time goes on, your
locally installed package will get further and further out of date, and could potentially conflict with newer
packages installed in updated analysis environments.

In our advice to users, we recommend the following Advanced Settings for ARE environments:

![ARE recommended settings](./images/ARE-recommended.png "ARE recommended JupyterLab settings")

Note the `PYTHONNOUSERSITE=1` environment variable. This will have the effect of preventing anything installed with `pip install --user` from loading at all. This setting also works in the terminal and can be used as a first step to diagnosing any issues with python package imports.

## Virtual environments
What we recommend is creating a [virtual environment](https://docs.python.org/3/library/venv.html). A virtual environment is a self-contained Python
environment built on top of an existing Python installation. The self-contained
nature of a virtual environment means that any issues due to conflicting 
dependencies can be avoided entirely. Being able to base the environment on top of
an existing `conda/analysis3` environment means that the installation can be kept
small, as most of the dependencies will have already been satisfied. 

There are only two additional commands required to install `xWMB` into a virtual environment, as shown below:
<terminal-window typingDelay=30 lineDelay=200>
    <terminal-line data="input">module use /g/data/xp65/public/modules</terminal-line>
    <terminal-line data="input">module load conda/analysis3</terminal-line>
    <terminal-line data="input">python3 -m venv xwmb_venv -\\\-system-site-packages</terminal-line>
    <terminal-line data="input">source xwmb_venv/bin/activate</terminal-line>
    <terminal-line data="input" inputChar="(xwmb_venv) $">pip install git+https://github.com/hdrake/xwmb.git@main</terminal-line>
    <terminal-line>Collecting git+https://github.com/hdrake/xwmb.git@main</terminal-line>
    <terminal-line>  Cloning https://github.com/hdrake/xwmb.git (to revision main) to /scratch/v45/dr4292/tmp/pip-req-build-qox565pe</terminal-line>
    <terminal-line>  Running command git clone -\\\-filter=blob:none -\\\-quiet https://github.com/hdrake/xwmb.git /scratch/v45/dr4292/tmp/pip-req-build-qox565pe</terminal-line>
    <terminal-line>  ...</terminal-line>
    <terminal-line>Successfully installed regionate-0.0.1 sectionate-0.2.1 xbudget-0.1.0 xgcm-0.8.2.dev15+g7492277 xwmb-0.1.1 xwmt-0.1.1</terminal-line>
</terminal-window>

```{warning}
Do not use `pip install --user` here, as this will result in the packages being installed in your `~/.local` directory, leaving the virtual environment empty
```

And that's it. We now have a fully self-contained environment with a custom 
package built on top of the `conda/analysis3` environment. It is important to note
that this virtual environment is tied to the version of `conda/analysis3` that was loaded
at the time of its creation, so it avoids the out-of-date dependencies issue mentioned earlier.
<terminal-window typingDelay=30 lineDelay=200>
    <terminal-line data="input" inputChar="(xwmb_venv) $">which python3</terminal-line>
    <terminal-line>~/xwmb_venv/bin/python3</terminal-line>
    <terminal-line data="input" inputChar="(xwmb_venv) $">ls -l ~/xwmb_venv/bin/python3</terminal-line>
    <terminal-line>lrwxrwxrwx 1 dr4292 v45 67 Jun  5 14:43 /home/563/dr4292/xwmb_venv/bin/python3 -> /g/data/xp65/public/apps/med_conda_scripts/analysis3-25.08.d/bin/python3</terminal-line>
</terminal-window>

Immediately after loading the `conda/analysis3` module, you can run
`source xwmb_venv/bin/activate` and any `python` command or script will now run under the new virtual environment.
```
#!/usr/bin/env bash
#PBS -l walltime=1:00:00,mem=4GB,ncpus=1,storage=gdata/xp65

module use /g/data/xp65/public/modules
module load conda/analysis3
source xwmb_venv/bin/activate

python3 my_script.py
```

 If you're running a script directly, you can
change the path in the [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) line to the path to the `python` symlink in the virtual environment:
```
#!/home/563/dr4292/xwmb_venv/bin/python3

import xwmb
...
```

## Troubleshooting
Virtual environments are designed to be disposable, so if something goes wrong, instead of trying
to fix the environment in place, it is safe to simply delete the environment and start again.
There are many reasons this might need to happen, for example, you're installing an old
package that is not compatible with e.g. `numpy` installed in the `conda/analysis3` environment.
In that case, you could simply delete the environment, load an older `conda/analysis3` environment
and re-create it.

<terminal-window typingDelay=30 lineDelay=200>
    <terminal-line data="input" inputChar="(xwmb_venv) $">deactivate</terminal-line>
    <terminal-line data="input">rm -rf xwmb_venv</terminal-line>
    <terminal-line data="input">module unload conda</terminal-line>
    <terminal-line data="input">module load conda/analysis3-25.08</terminal-line>
    <terminal-line data="input">python3 -m venv xwmb_venv --system-site-packages</terminal-line>
    <terminal-line data="input">source xwmb_venv/bin/activate</terminal-line>
    <terminal-line data="input" inputChar="(xwmb_venv) $">pip install git+https://github.com/hdrake/xwmb.git@main</terminal-line>
</terminal-window>

In a similar vein, if you're installing more packages into a virtual environment and they
begin to conflict with each other, the best course of action is to create a new virtual environment.
Virtual environments are isolated from each other, so rather than creating large, monolithic 
environments, it is common practice to create virtual environments for a single package, and have
multiple virtual environments depending on your immediate requirements.

In general, if anything in the environment doesn't work for whatever reason, the best course of action
is to delete it and try again. You may need to try different `conda/analysis3` environments, or, if none of those
work, use one of the NCI installed Python modules as your base. 
```{note}
NCI's Python modules do not have `dask`, `xarray`, etc. installed, so the resultant virtual environments could be quite large.
```


## Jupyter Kernel
There is one more step necessary to have your new virtual environment usable as a kernel in an ARE JupyterLab session. 
You'll need to call `ipykernel install` while the virtual environment is activated.
<terminal-window typingDelay=30 lineDelay=200>
    <terminal-line data="input">module use /g/data/xp65/public/modules</terminal-line>
    <terminal-line data="input">module load conda/analysis3</terminal-line>
    <terminal-line data="input">source xwmb_venv/bin/activate</terminal-line>
    <terminal-line data="input" inputChar="(xwmb_venv) $">python3 -m ipykernel install -\\\-user -\\\-name xwmb-venv -\\\-display-name "xWMB + conda/analysis3"</terminal-line>
</terminal-window>

```{note}
Since this virtual environment is built on top of `conda/analysis3`, you do not need to install `ipykernel` in it, as this is already present in the conda environment. 
```

Now, when
you launch a new ARE JupyterLab instance, you will be able to see your new environment,
launch a notebook and import the `xWMB` package

![Jupyterlab launch](./images/notebook_virtualenv_launch.gif)

You only have to do this once. If you delete the environment and recreate it at the same path,
Jupyterlab will still be able to use it.

## Summary

By creating a virtual environment using the `conda/analysis3` 
environment as your base,
its possible to add in packages ACCESS-NRI can't install, without having the overhead of maintaining
your own large conda environment. Furthermore, using a virtual environment  is more stable and easier to maintain and reuse than installing software using `pip install --user`. Virtual environments are simple to
install, and simple to delete and re-create as necessary. Virtual environments can also be seamlessly integrated into
ARE JupyterLab sessions, allowing you to use it anywhere you would use an `analysis3` conda environment.


## Acknowledgements
Thanks to Davide Marchegiani for [animated-terminal.js](https://github.com/atteggiani/animated-terminal.js).
Thanks to the CLEX CMS team for the original materials this section is based on.