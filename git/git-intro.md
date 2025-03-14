# Why git? #

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

You can include code in a repository, for example as python scripts of jupyter notebooks, but also text documents, data and any other file you need for your project. There are a few consideration in terms of the size of the files that we cover in the --TODO complete with name and link--  section. 

If you are starting to work with git, we suggests the [Working with Git alone](git/git-alone.md) sections. If you are planning to work as part of a team or you want to contribute to others' people project, the [Working with Git in a team](git/git-team.md) is the section for you. 

### Before you start

Make sure you have:

* git installed in your laptop if you are working locally. If you are planning to work on Gadi, everything is already installed there.
* a GitHub account and follow the instructions to set up permissions. 


1. To introduce yourself to git, that is to let it know your name and email you will need to run this on the terminal:

```
git config --global user.name 'Mary Stratus'
git config --global user.email 'mary@example.com'
```

2. Configure the permissions

TODO add instructions to configure token/ssh keys



TODO add references to reproducibility.rocks and any other source to give credit.