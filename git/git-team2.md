(content:git-team2)=
# Working with Git in a team

In this section we are going to see what it looks like to collaborate with others on gitHub projects. Up to now you were working with git alone using this workflow:

``` 
Modify a file --> Add --> Commit --> Push
```

So every time you run `git push`, git will compare the files in the remote repository with the version you are pushing and update everything. But when working with other people there are instances that can potentially create *merge conflicts*. While the name suggest otherwise a merge conflict is not necessarily an error, it happens when the version of a file on the local repo is not compatible with its version on the remote repo. Sometimes, Git is able to combine the two sets of changes without any trouble. But sometimes it gets stuck, and requires a human to intervene and decide how multiple changes should be combined, do you keep version A, version B or a combination of both?

## Identifying merge conflicts

Imagine a colleague added a new line in `read_era5.py` to import the numpy package. The file looks like this:

```python
import xarray as xr
import matplotlib.pyplot as plt
import numpy

# Load a NetCDF climate dataset
ds = xr.open_mfdataset("era5_data.nc")
print(ds)

# Plot a temperature time series
plt.plot(df["date"], df["temperature"])
plt.xlabel("Date")
plt.ylabel("Temperature")
plt.title("Temperature Over Time")
plt.show()

```

At the same time, you also added this line but decided that `import numpy as np` is the right way to go. When you commit and push the new version of the file to the remote repository you get this message:


```bash
Auto-merging read_era5.py
CONFLICT (content): Merge conflict in read_era5.py
Automatic merge failed; fix conflicts and then commit the result.
```

The message says that there are conflicts withe the content of `read_era5.py` and that the automatic merge failed. The good news is that we now which file is the issue. Here, doing small commits with small changes helps a lot. 

Now, if you come back to `read_era5.py`, the content has changed:

```python
import xarray as xr
import matplotlib.pyplot as plt
<<<<<<< HEAD
import numpy as np
=======
import numpy
>>>>>>> 318548cbe665e65b4dae4a5da8d4eba187a4c6b2

# Load a NetCDF climate dataset
ds = xr.open_mfdataset("era5_data.nc")
print(ds)

# Plot a temperature time series
plt.plot(df["date"], df["temperature"])
plt.xlabel("Date")
plt.ylabel("Temperature")
plt.title("Temperature Over Time")
plt.show()
```

The funny additions `<<<<<<<`, `=======` and `>>>>>>>` were added by git to highlight the merge conflict. Everything between `<<<<<<< HEAD` and `=======` correspond to the local version, and the content between `=======` and `>>>>>>>` is the version currently on GitHub. The long string is the commit id. 

If you are using git from the terminal, you can edit this file, remove the arrows and keep the version you prefer. Some GUIs and visual interfaces will let you choose the version you want to keep and take care of editing the file, for example [check this short video to see how to solve a merge conflict form an ARE session using the jupyter git extension](https://www.youtube.com/watch?v=glB5WTMoR-g).


Thi can also happen if you changed something on GitHub, forgot to run `git pull` locally and changed the same file. While merge conflicts comes with git, we'll try to avoid them as much as possible. 

### 1. Pull first

It is a good idea to run `git pull` before you start to work with the files in a repository, this way you will start working with the most current version of the repository. 

### 2. People work on different files

The easiest way to avoid merge conflicts is to tell people to work in different files and to not edit other's people files. This work.

In the long run you may need to work on someone else code and for those situations communication between team members is key.

### 3. People takes turns 

Maybe more than one person is working on the same file, it could be a script or maybe is documentation. That usually will be a job for many people. One solution is to take turns, you work on the file during the morning, your colleague works on it during the afternoon. 

### 4. People work on branches

Here is when the things get interesting and a little more complicated, but we are taking fully advantage of what git has to offer.

Branches may be a new concept but actually, you are always working on a branch without noticing. The "default" branch is usually called *main* because it is the principal, so you were "adding commit to the main" all this time. A **branch** in git is a tag that points to a specific *commit* in the repository from which other versions of the repository are created. 
Working in a branch allows you to modify files without modifying the same files in other branches, because essentially you are working on a “parallel” set of files.
When you want to pass the changes from your branch to the main branch, you would have to do a **merge** to combine the branches.

```{figure} images/branch.png
---
alt: "Diagram of a git tree that shows the main branch with 4 commits and a secondary branch called function that starts from commit 2 in the main branch anas has 2 new commits, independent from the main."
---
Git tree.
```

There are different ways to use branches. Maybe every person in the team has one branch and work on that branch until is time to merge everything to the main. Or maybe, and this is more common, each branch represent a new feature added to the project. So if you are adding a new function, you create a branch, develop the code and them merge it. 

Working with branches won't eliminate merge conflicts, but those will appear once when you merge your branch to the main and will prepare to deal with it.

Before we revise how the workflow looks like in this case, there is one new concept we need to mention: a **Pull Request** or PR is a GitHub tool that allows you to make changes to a branch and then request to the repo owner or maintainer to merge those changes into the main branch.
Pull requests can come from forks (will talk about that later) or from independent branches within the repository. 
They allow the maintainers and contributors to the project to review, discuss, request, and approve the changes and merge then to the main branch.

(content:git-team:scenario1)=
## Collaborative workflow 1

This diagram shows the workflow when you what to contribute to the repository using branches. Here we assume you have write permissions on the remote repository,

```{figure} images/no_fork.png
---
alt: 'Concept model of the workflow in scenario 1. The remote repository is copied into a local repository with a "clone". Files inside the local repository and the remote repository are synced with push and pull. The owned remote repository can be merged into the foreign remote repository with a pull request.'
---
Collaborative workflow without a fork.
```

1. Clone the repository on your computer.
2. Create a new branch.
3. Edit files, add them and make commits in that branch.
4. When the changes are done and ready, send a pull request to the remote repository to compare your changes in your branch with main branch.
5. The pull request is accepted and merged or you have to make new changes (go back to step 3).
6. Once the PR is accepted and merged, the main branch now has the updated changes and you can delete the branch you were working on.
7. The process can be repeated several times, in parallel or in sequence depending on the size of the team and project.

Let's try this with a exercise:

:::{admonition} Exercise
:class: important, dropdown

1. Choose a repo, it can be a repo you own  or, even better, someone else repo where you are a collaborator. You will need the URL (remember we use SSH option)
> We'll assume you are working with [this repo](content:git-alone:new-repo) from now on.
2. On the terminal, run `git clone <repo URL>`. You need to make sure you are in the right folder.
3. Create a new branch with `git checkout -b suggest-plot`, it will automatically switch to the new branch.
4. Edit o create a new file. For example, add the necessary code to plot the data in `read_era5.py`:

```diff
 import xarray as xr
+import matplotlib.pyplot as plt
 
 # Load a NetCDF climate dataset
 ds = xr.open_mfdataset("era5_data.nc")
 print(ds)
+
+# Plot a temperature time series
+plt.plot(df["date"], df["temperature"])
+plt.xlabel("Date")
+plt.ylabel("Temperature")
+plt.title("Temperature Over Time")
+plt.show()
```
> here the `+` at the beginning of a line means that we are adding it (or modifying it) to the file. The opposite will be true if we see a `-`. The above results is waht `git diff read_era5.py` returns after to modify the script. 

5. With `git status` you can check where you are:

```bash
On branch suggest-plot
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   read_era5.py

no changes added to commit (use "git add" and/or "git commit -a")
```
6. Add and commit the file as usual and send the new version to GitHub with a push. You will get something like this:

```bash
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 20 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 468 bytes | 468.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
remote: 
remote: Create a pull request for 'suggest-plot' on GitHub by visiting:
remote:      https://github.com/<username>/new-project/pull/new/suggest-plot
remote: 
To github.com:<username>/new-project.git
 * [new branch]      suggest-plot -> suggest-plot
branch 'suggest-plot' set up to track 'origin/suggest-plot'.
```
7. To open a pull request you need to go the repo on GitHub. You will see a big "Compare & pull request" green button. Click on that!

```{figure} images/pr-1.png
:width: 95%
:alt: 'Screenshot of repo in GitHub.com/username/new-project with a new commit in a branch that is not main. It shows a button that says "Compare & pull request" on the top of the page'

```

8. GitHub will compare main and suggest-plot branches to see if the proposed changes are compatible. In this step you will need to add a title and a message to open the PR. 

And that's it. The maintainer of the repo with get an email and with time, revise and hopefully accept your contribution. 

:::

Here is a list of the new commands you just used:

:::{admonition} Commands
:class: tip

- `git clone <repo URL>` to clone a repository, independently of if is yours or not. This command will download the repository in the folder you are in the terminal.
- `git checkout -b <name>` will create a new branch.
- `git checkout <name>` will move you to a different branch.

:::