(content:new-configurations)=
# Creating, documenting and running new configurations

This section is intended to document the process of creating new configurations for ACCESS-AM3 in the context of the 21st Century Weather Centre. It's not intended to replace the ACCESS-NRI documentation, if you are interested in learning how to run ACCESS-AM3, please refer to the [ACCESS-NRI documentation](https://docs.access-hive.org.au/models/run_a_model/run_access-am3/).

The section assumes you are using `cylc8`, if not, it's a good moment to do the switch. 

## New configurations?

If you are planning to run ACCESS-AM3 with a different set of ancillaries, or different initial conditions, or you are changing anything in the suite, **that is a new configuration**.

When you run the model with a specific configuration, you get an **experiment**.

As an example for this tutorial, lest say we want to run ACCESS-AM3 with a new initial condition, based on the current `release-n512e-aeroclim` configuration. We'll call it `n512e-1983`.

## Workflow 

### 1. Clone the repository

The 21st Century Weather configurations are store in a private repository (a fork of the ACCESS-NRI repository) at `https://github.com/21centuryweather/access-am3-configs`. You need to be a member of the organisation to have access to the repository. If you don't have access, you can request it on Cumulus.

To clone the specific configuration:

```bash
git -C ~/roses clone git@github.com:21centuryweather/access-am3-configs.git -b release-n512e-aeroclim
```

:::{admonition} Note
:class: tip

If you plan to use more than one configuration or want to explore them, you can clone the entire repository and switch between branches. 

```bash
git -C ~/roses clone git@github.com:21centuryweather/access-am3-configs.git
``` 
:::

If you already have the repository cloned, you can update any branch (bring any changes from the remote repository) with `git pull` or fetch a new branch from the remote repository with `git fetch origin <branch-name>`.



### 2. Create and switch to a new branch

We recommend to create a new branch for each new configuration, this will make it easier to track changes, share the configuration and reproduce experiments. 

To create a new branch, navigate to the folder ~/roses/access-am3-configs` use the following commands:

```bash
git checkout release-n512e-aeroclim # to switch from main to the n512e beta relase configuration
git checkout -b n512e-1983
``` 

This will switch the repository to the `release-n512e-aeroclim` branch and create a new branch called `n512e-1983`. At this point both branches are identical.


:::{admonition} Note 
:class: tip

Check the current branch you are using and any new changes with `git status`.

:::

Now you can make any changes to the configuration, for example, changing the initial conditions to use a restart from 1983.

### 3. Tracking changes

It's very important to track any changes you make to the configuration. This will help you to reproduce experiments and share your configuration with others. We'll track changes by doing commits to the configuration branch and pushing them to the remote repository on GitHub.

Let's change the initial conditions to use a restart from 1983. Edit the file `site/nci_gadi.rc` and change the line:

```bash
{% set AINITIAL = '/g/data/gb02/public/AM3/restartdumps/n96e/deva.da19830101_00' %}
```
And these 2 lines to run the experiment from January 1, 1983 for 12 months in `rose-suite.conf_nci_gadi`:

```bash
EXPT_BASIS='19830101T0000Z'
EXPT_RUNLEN='P12M'
```
When running `git status` you will see that the files `site/nci_gadi.rc` and `rose-suite.conf_nci_gadi` have been modified:

```bash
$ git status
On branch n512e-1983
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   rose-suite.conf_nci_gadi
	modified:   site/nci_gadi.rc

no changes added to commit (use "git add" and/or "git commit -a")
```

As git mention, we need to add the files to the staging are and commit them:

```bash
git add rose-suite.conf_nci_gadi site/nci_gadi.rc
git commit -m "Changed initial conditions to use restart from 1983 and run for 12 months"
```

The changes, now are being tracked by git in your local repository. You can test the configuration and run experiments. If you need to do further changes, maybe something went wrong, it's crucial to repeat the steps to track **all** the changes in the configuration. 

:::{admonition} Note 
:class: tip

If you want to see the changes you have made to a file, you can use `git diff <file>`. This will show you the differences between the current version of the file and the last committed version. It is a good practice to do this before committing the changes, to make sure you are not committing any unwanted changes.

:::

### 4. Push changes to GitHub

When you are ready, you will need to push the configuration (the new branch) to the remote repository on GitHub:

```bash
git push --set-upstream origin n512e-1983
```

Now you can check your configuration on GitHub and share it with others. 

:::{admonition} Note 
:class: tip

You can do as many commits and update (push) the branch to the remote repository as many times as you need until the configuration is ready to go. Always be sure to:

* Track all the changes you make to the configuration.
* Create a new branch for each new configuration.

:::