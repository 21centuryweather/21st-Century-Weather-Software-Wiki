(content:getting-started)=
# Getting started on gadi

The National Computing Infrastructure ([NCI](https://www.nci.org.au/)) hosts the supercomputer `gadi` where you will be conducting most of your research. Through NCI, you will have access to high-performance computing (HPC) resources and large-scale data storage, including some of the most
used datasets like ERA5 and CMIP6 data.

In this section, we will guide you through the steps to get started on `gadi` using some handy tutorials developed by [ACCESS-NRI]([https://www.access-nri.org.au).

## 1. Setting up an account on `gadi`

To access `gadi`, you will need to set up an account through NCI. Follow the [instructions here](https://docs.access-hive.org.au/getting_started/set_up_nci_account/) to create your account. You will need a institutional email and the name of the projects you need to join. Projects gives you access to computational and disk storage resources on `gadi`. 21st Century Weather has 5 projects you can join:

| NCI Project  | Theme                           | Centre Projectcs                                      | NCI Project Lead  | SU/quarter + storage     |
|--------------|---------------------------------|-------------------------------------------------------|-------------------|------------------------|
| [gb02](https://my.nci.org.au/mancini/project/gb02)| Centre-wide Strategic Projects  | All              | gb02 Panel*       | 2.25 MSUs + 80 TB Storage |
| [fy29](https://my.nci.org.au/mancini/project/fy29)| High-Resolution Modelling       | Modelling        | Bethan White      | 1.25 MSUs + 20TB Storage  |
| [if69](https://my.nci.org.au/mancini/project/if69)| Circulation Change              | Weather System Dynamics, Variability & Warmer World,  | Chenhui Jin       | 1.25 MSUs + 20TB Storage  |
| [ng72](https://my.nci.org.au/mancini/project/ng72)| Weather Change                  | Weather Resources & High Impact Weather               | Andrew Brown      | 1.25 MSUs + 20TB Storage  |
| [su28](https://my.nci.org.au/mancini/project/su28) | Datasets for the Centre         | All                                                   | Sam Green         | 20 TB Storage          |

*gb02 panel: Navid Constantinou, Paul Gregory, Bethan White, Chenhui Jin, Andrew Brown, and Sam Green

If you are unsure which projects to join, please ask on Cumulus. 


## Accessing `gadi` via a terminal.

To connect to `gadi` via the terminal we use 'Secure Shell Protocol' or `ssh`. Even if you are not planning to use the terminal often, it is useful to know how to connect this way. The steps bellow will also help you if you want to use VSCode with `gadi`. 

Follow the instructions in the section [Logging into Gadi](https://docs.access-hive.org.au/getting_started/set_up_nci_account/#login-to-gadi). You will need your password. 

### Logging to Gadi without a password

Typing your password every time you want to log into `gadi` can be tedious. To avoid this, you can set up `ssh` keys to log in automatically without a password. ssh keys are a pair of cryptographic keys that allow secure access to remote systems. These keys are stored in your local machine and the remote server (gadi in this case) in a folder called `.ssh/` (it is a hidden folder!).

Follow the instructions in the section [Automatic login](https://docs.access-hive.org.au/getting_started/set_up_nci_account/#automatic-login) to create your ssh keys and copy the public key to `gadi`. Once you have done this, you should be able to log into `gadi` without a password.



## Using the Australian Research Environment (ARE)

NCI have created a set of web-based graphical tools to help perform your computational research. These tools (including a Virtual Desktop and the `JupyterLab` environment) are made available via the 'Australian Research Environment' ([ARE](https://are.nci.org.au/)) on `gadi`.

To access ARE, you will need to use your NCI credentials (the ones you used to create your `gadi` account). 

Follow the [instructions in the ACCESS-NRI documentation](https://docs.access-hive.org.au/getting_started/are/) to start an ARE session. The tutorial show how to create a:
- Virtual Desktop Interface (VDI)
- `JupyterLab` session

but there are other option available too.

:::{note}

Any ARE session will use service units or SUs, which are the currency used on `gadi` to allocate computational resources. Each project you join will have a certain number of SUs allocated to it per quarter. Small JupyterLab sessions will use very few SUs, but it can add up over time during the quarter. Please always check your SU usage via the [Computational Resource Dashboard](https://21centuryweather.github.io/nci_resource_tools/dashboard.html) and stop any ARE sessions you are not using.

:::