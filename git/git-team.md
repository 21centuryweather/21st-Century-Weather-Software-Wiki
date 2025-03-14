# Working with Git in a team

In this section we are going to see what it looks like to use collaborative workflows
to work with other people to do data analysis and software development.
Previously you were working with git individually:

``` 
Modify a file --> Add --> Commit 
```

You did all this in a branch of the repository that we usually call *main* because it is the main one. A **branch** in git is a tag that points to a specific *commit* in the repository from which other versions of the repository are created. 
Working in a branch allows you to modify files without modifying the same files in other branches, because essentially you are working on a “parallel” set of files.
When you want to pass the changes from your branch to the main branch, you would have to do a **merge** to combine the branches.

Collaborative workflow includes two new concepts: *forks* and *pull requests*.

- A **fork** is a copy of another person's or team's repository that will be stored in your GitHub account.
Both the original and the fork are on GitHub, the difference is that you can make
modifications to the version that is in your account.
- **Pull Request** or PR is a GitHub tool that allows you to make changes to a fork or branch and then request to the maintainer to merge those changes into the main repository.
Pull requests can come from forks or from independent branches within the repository. They allow
the maintainers and contributors to the project to review, discuss, request, and approve the changes and merge then to the main branch.

Learning how to effectively use these tools and concepts can make collaborating with other people
much easier. You may even be able to use this workflow even on an individual basis.

When you are collaborating on a project, you may find yourself in one of the following situations
of these situations:

- Scenario 1: You have write permissions on the remote repository,
in this case you don't need to use forks.
- Scenario 2: You do not have write permissions on the remote repository, so you will need to use forks.

In both scenarios we will assume that you will work with branches to separate
your work from other people's before passing them to the main branch.

### Scenario 1

This diagram shows the workflow when we **don't** need to do a fork.

![Concept model of the workflow in scenario 1. The remote repository is copied into a local repository with a "clone". Files inside the local repository and the remote repository are synced with push and pull. The owned remote repository can be merged into the foreign remote repository with a pull request.](images/no_fork.png)

1. Clone the repository on your computer following the [these same instructions](/materials/day2/01-git/#creating-a-new-repository).
2. Create a new branch.
3. Edit files, add them and make commits in that branch.
4. When the changes are done and ready, send a pull request to the remote repository to
to compare your changes in your branch with main branch.
5. The pull request is accepted and merged or you have to make new changes (go back to step 3).
6. Once the PR is accepted and merged, the main branch now has the updated changes and you can delete the branch you were working on.
7. The process can be repeated several times, in parallel or in sequence depending on the size of the team and project.

### Scenario 2

For fork-based workflows, the process is as follows:

![Concept model of the workflow in scenario 2. A foreign remote repository can be forked to an owned remote repository with a "fork". The remote repository is copied into a local repository with a "clone". Files inside the local repository and the remote repository are synced with push and pull. The owned remote repository can be merged into the foreign remote repository with a pull request.](images/si_fork.png)

1. Create a fork of the main repository (if you don't already have one).
2. Clone the forked repository on your computer.
3. Create a new branch in the forked repository.
4. Make changes to the files and commit them to the branch.
5. When everything is ready, open the pull request. If you need to make new
changes you will have to go back to step 4.
6. If the PR is accepted and merged, the main branch in the main repository will be updated and the new branch can be deleted.
7. Finally you can synchronize your forked repository with the main repository.


