# Introduction

Do you have something like this in your computer?
```
/home/pao/Documents/thesis
├── script.py
├── thesis.Rmd
├── thesis_reviwed.Rmd
├── thesis_reviwed2.Rmd
├── thesis_final.Rmd
├── thesis_finalfinal.Rmd
├── this_is_it.Rmd
├── now_this_is_it_for_real_this_time_i_swear.Rmd
└── FINAL.Rmd
```
Probably we all have, or had something like this at one point because we didn’t use a version control system. Version control systems manage the evolution and changes of a set of files that we’ll call repository. If you ever looked at the history of a Google Docs file, it is like that but in a very controlled way. Git is one popular version control system.

If you work alone, git is great to track changes and recover previous version of your files. You can also use a remote repository to have a back up and share your work.

If you work as a team you can take advantage of all the above and also use version control as a tool to collaborate and organize the various versions of the same file present in the multiple computers you and they use.

You can include code in a repository, for example as python scripts of jupyter notebooks, but also text documents, data and any other file you need for your project. There are a few consideration in terms of the size of the files that we cover in [a future section](content:git-alone:file-size). 

If you are starting to work with git, we suggests the [Working with Git alone](content:git-alone) section. If you are planning to work as part of a team or you want to contribute to others' people project, the [Working with Git in a team](content:git-team) is the section for you. 

And if you are going to include jupyter notebooks on your repositories and plan to track and review changes, we recommend you also check the [Tracking changes on jupyter notebooks](content:git-notebooks) section. 

### Before you start


:::{admonition} Global Hackathon 2025
:class: warning
If you are here because of the training sessions for the Global Hackathon, please make sure you install everything locally. NCI systems will be down on May 6th so we'll work locally (from everyone's computers) instead of from Gadi.

:::

Make sure you have:

* git installed in your laptop if you are working locally. If you are planning to work on Gadi, everything is already installed there.
* a GitHub account and follow the instructions bellow to set up permissions. 


#### 1. To introduce yourself to git

That is to let it know your name and email you will need to run this on the terminal:

```
git config --global user.name 'Mary Stratus'
git config --global user.email 'mary@example.com'
```

#### 2. Configure the permissions between git and GitHub

* **Create a SSH key**

In the terminal on Gadi or your laptop:

```
ssh-keygen -t ed25519 -C "your_email@example.com"
```
you will need to press enter 3 times (to confirm file name and passphrase -people usually leave this empty, but [here](https://access-hive.org.au/getting_started/set_up_nci_account/#change-default-project-on-gadi) are better instructions on how to set up a passphrase). This creates 2 files in the `.ssh` folder:

* `id_ed25519` do not share with anyone
* `id_ed25519.pub` share this with GitHub

Then add the new key to the ssh agent with:

```
eval "$(ssh-agent -s)"
```
and

```
ssh-add ~/.ssh/id_ed25519
```

* **Add the new key on GitHub**

Now you can copy the public key (the content of `.ssh/id_ed25519.pub`) to add it to your GitHub Settings:

1. On github.com, go to your Settings
2. Click on **SSH and GPG keys** on the Access section (left panel)
3. New SSH Key
4. Add a title (Usually the name of the computer) and paste the public key on the **key** fill.
5. And add. You may need to provide your password or 2FA. 

#### 3. Extra configuration that will make your life easier

We mention this to things [Working with Git alone](content:git-alone), in the mean time we recommend to run this on the terminal:

```
git config --global push.autoSetupRemote true
git config --global init.defaultBranch main
```

More advance config: [How Core Git Developers Configure Git](https://blog.gitbutler.com/how-git-core-devs-configure-git/)

### Choose your own adventure

The next sections will assume you are working from a terminal on Gadi or on your laptop. There are many ways to interact with git, here are a few other options.

#### Working locally

* **Windows:** when you install git on Windows you are also installing git bash and all the commands we use here should work. If you prefer a visual interface there are many options like GitHub desktop and GitKraken. If you use VSCode or Jupyter Lab, there are git extensions you can install to create commits and push/pull from remote repositories. 

* **Linux/Mac:** you already have a working terminal. If you prefer a visual interface there are many options like GitHub desktop and GitKraken. If you use VSCode or Jupyter Lab, there are git extensions you can install to create commits and push/pull from remote repositories. 

#### Working on NCI systems

* **Logging to Gadi from a terminal:** this is the default option.

* **ARE:** If you use a virtual machine, you will have access to a terminal. If your work will Jupyter Lab, there is git extension you can install to create commits and push/pull from remote repositories. If you use RStudio Server, you will find a "git" tab that works as a git interface. 


### References

The materials in this section are inspired on:

* [An R reproducibility toolkit for the practical researcher](https://reproducibility.rocks/)

* [Version Control with Git from The Carpentries](https://swcarpentry.github.io/git-novice)