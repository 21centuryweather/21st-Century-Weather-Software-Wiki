# Running VSCode on gadi ARE session

Connecting a VSCode window your an ARE Jupyter notebook session uses the same method to connect to a `gadi` compute node.

Again, we will rely on the use of the `ProxyJump` inside our `ssh` configuration file to connect to the ARE Jupyter notebook server via a `gadi` login node.

The difference is we need to start an ARE session, rather than an interactive `qsub`.

So, login to https://are.nci.org.au as normal and start your JupyterLab session as normal. Take note of the ID of the compute node hosting your JupyterLab session.

![SSH-gadi-compute](./images/JupyterLab.png)

Then use the Remote-SSH button on the bottom left corner to `Connect to Host` and then type in the ID of you    r ARE session.

![SSH-ARE-connect](./images/ARE_connect.png)

In this example I type `gadi-cpu-bdw-0004.gadi.nci.org.au`

![SSH-ARE-connect](./images/ARE_connect2.png)

Hit `Continue` when provided with a Host fingerprint.

:::{note}

The 'pop-up' menu states `Select configured SSH host or enter user@host`. Because we have entered `Host gadi-cpu-*` in our `~/.ssh/config` we **don't** need to enter `user@host`. Any host the begins with `gadi-cpu` will be automatically configured.
:::

Let's load a notebook into VScode. I'll load https://github.com/21centuryweather/animations/blob/main/workshop_demo/animation_notebook.ipynb.

(If you haven't clone this repository already, you will do so later in this workshop)

Now we need to load the notebook kernel. If we want to use an `xp65 analysis3` kernel, this is a **two-stage** process.

1. Load the `Python Interpreter` first via the `Command Palette`. In this example I'll load `analysis3-25.09` by selecting `Enter Interpreter Path`. I then enter `/g/data/xp65/public/apps/med_conda_scripts/analysis3-25.09.d/bin/python`

![SSH-ARE-connect](./images/ARE_interpreter.png)

![SSH-ARE-connect](./images/ARE_interpreter2.png)

![SSH-ARE-connect](./images/ARE_interpreter3.png)

2. Then click `Select Kernel` and select `Python Environment` and choose the environment associated with the `analysis3` version.

![SSH-ARE-connect](./images/ARE_kernel.png)

![SSH-ARE-connect](./images/ARE_kernel2.png)

If you **don't** do step one, you won't be able to see any of the `analysis3` kernels.

If you want to use any of your own kernels located in `~/.local/share/jupyter/kernels/` you can just click `Select Kernel` and then `Jupyter Kernel` when you connect for the first time. 

![SSH-ARE-connect](./images/local_kernels.png)

Note this list of 'local' kernels matches the same available here:

![SSH-ARE-connect](./images/ARE_kernels.png)

But it appears that if you have already loaded and selected an `analysis3` kernel, you won't be able to load the local kernels without disconnecting and re-connecting again. 

Now you can execute your Juypter notebook inside your VSCode session.

 