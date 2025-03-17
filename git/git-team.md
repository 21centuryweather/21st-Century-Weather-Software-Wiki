# Working with Git in a team

In this section we are going to see what it looks like to use collaborative workflows to work with other people to do research and software development.
Previously you were working with git individually using this workflow:

``` 
Modify a file --> Add --> Commit --> Push
```

You did all this in a branch of the repository that we usually call *main* because it is the main one. A **branch** in git is a tag that points to a specific *commit* in the repository from which other versions of the repository are created. 
Working in a branch allows you to modify files without modifying the same files in other branches, because essentially you are working on a “parallel” set of files.
When you want to pass the changes from your branch to the main branch, you would have to do a **merge** to combine the branches.

```{figure} images/branch.png
---
alt: "Diagram of a git tree that shows the main branch with 4 commits and a secondary branch called function that starts from commit 2 in the main branch anas has 2 new commits, independent from the main."
---
Git tree.
```

Collaborative workflow includes two new concepts: *forks* and *pull requests*.

- A **fork** is a copy of another person's or team's repository that will be stored in your GitHub account.
Both the original and the fork are on GitHub, the difference is that you can make modifications to the version that lives in your account.
- **Pull Request** or PR is a GitHub tool that allows you to make changes to a fork or branch and then request to the repo owner or maintainer to merge those changes into the main branch.
Pull requests can come from forks or from independent branches within the repository. 
They allow the maintainers and contributors to the project to review, discuss, request, and approve the changes and merge then to the main branch.

Learning how to effectively use these tools and concepts can make collaborating with other people much easier. 
You may even be able to use this workflow even on an individual basis.

When you are collaborating on a project, you may find yourself in one of the following situations:

- Scenario 1: You have write permissions on the remote repository,
in this case you don't need to use forks.
- Scenario 2: You do not have write permissions on the remote repository, so you will need to use forks.

In both scenarios we will assume that you will work with branches to separate
your work from other people's before passing them to the main branch.

### Scenario 1

This diagram shows the workflow when we **don't** need to do a fork.

```{figure} images/no_fork.png
---
alt: "Concept model of the workflow in scenario 1. The remote repository is copied into a local repository with a "clone". Files inside the local repository and the remote repository are synced with push and pull. The owned remote repository can be merged into the foreign remote repository with a pull request."
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
> here the `+` at the beginning of a line means that we are adding it (or modifying it) to the file. The opposite will be true if we see a `-`. 
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

```{figure} images/fork-1.png
:width: 70%
:alt:"Screenshot of a forked repo in GitHub.com/username/new-project with a new commit. It shows a button that says "Compare & pull request" on the top of the page"

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

### Scenario 2

For fork-based workflows, the process is as follows:

```{figure} images/si_fork.png
---
alt: "Concept model of the workflow in scenario 2. A foreign remote repository can be forked to an owned remote repository with a "fork". The remote repository is copied into a local repository with a "clone". Files inside the local repository and the remote repository are synced with push and pull. The owned remote repository can be merged into the foreign remote repository with a pull request."
---
Collaborative workflow with a fork.
```

1. Create a fork of the main repository.
2. Clone the forked repository on your computer.
3. Create a new branch in the forked repository.
4. Make changes to the files and commit them to the branch.
5. When everything is ready, open the pull request. If you need to make new
changes you will have to go back to step 4.
6. If the PR is accepted and merged, the main branch in the main repository will be updated and the new branch can be deleted.
7. Finally you can synchronize your forked repository with the main repository.

Let's go through the workflow with a exercise.

:::{admonition} Exercise
:class: important, dropdown

In this second scenario we assume you don't have write permission in the repo you want to collaborate. For example, you may want to collaborate with this [*cadavre exquis*](https://en.wikipedia.org/wiki/Exquisite_corpse)! The idea is to add a new sentence to the story [here](https://github.com/paocorrales/cadavre-exquis).    

1. Go to [github.com/paocorrales/cadavre-exquis](https://github.com/paocorrales/cadavre-exquis)
2. Fork the repository using the “Fork” button on the top right corner. This will create a copy of the repo into your GitHub account.
3. Copy the url of the repo and clone it to your computer. On the terminal, run `git clone <repo URL>`. Remember to use the SSH option. 

The repo is very small, it only have a README and a text file called `story.txt`. So, in this cadavre exquis, each collaborator adds a phrase to the story in a new line. The new phrase needs to be some how connected with the previous. 

4. Open the `story.txt` file and add a phrase.
5. Save the file, create a commit and push it to your repo. 
6. Push it to your remote repository.

Now, if you go to the remote repo on GitHub, you will see something like this::

TODO: add screenshot

7. Click on “Open pull request”.
8. And click on “Create pull request”.
10. We are almost there. Complete the pull request with a title and a message and then finish the PR

And that’s it! The owner/s of the repo will receive an email. They can review the pull request, comment on it and eventually accept the contribution. When that happen the changes you made will appear in the rep
:::


:::{tip}

If you prefer or need to do everything from the terminal, including forks and PR, GitHub has a command line called [gh](https://github.com/cli/cli). 

So, for example if you what to create a PR, from the terminal run:
```bash
 gh pr create --title "Pull request title" --body "Pull request body"
```
Adding a title and a message in the body of the pull request. The Terminal will return the link to the pr in the web. It’s useful to remember the number.

You can also add comments after opening the pull request with:
```bash 
gh pr comment <number of pr> --body "Some comment"
```
:::
