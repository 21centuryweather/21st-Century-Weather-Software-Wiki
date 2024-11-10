# Using MOSRS and Rose

Let's run a very small `rose/cylc` suite via the UK Met Office `MOSRS` service.

Before doing so, we will start what is called a 'persistent session' on `gadi`.

## Persistent sessions ##

What is a persistent session and why do we need it?  Because `gadi` `ARE` sessions have strict time limits, as does the `PBS` job scheduler. Individual `PBS` jobs on `gadi` have a time limit of 48 hours, see 

https://opus.nci.org.au/pages/viewpage.action?pageId=236881198

If we are running a weather or climate session that will take days (or weeks, or months!) to run, we need an ability to co-ordinate all these sub-tasks for an extended period of item.

Hence, NCI has created the persistent sessions capability. It creates a temporary virtual machine which runs indefinitely. This allows us to launch a `rose/cylc` suite without any time limits.

NCI has the following documentation describing how to setup and configure persistent sessions:

https://opus.nci.org.au/pages/viewpage.action?pageId=241927941

https://opus.nci.org.au/display/Help/Persistent+Sessions

https://opus.nci.org.au/display/DAE/Persistent+Sessions+For+Cylc+Jobs

ACCESS-Hive also has the following:

https://access-hive.org.au/models/run-a-model/run-access-cm/#start-a-new-persistent-session

As you can see, there are a few ways to configure a persistent session. My preferred way is via an `ssh` terminal session.

After starting a session - e.g.
```
$ persistent-sessions start <session-name> -p <project>
session db78f6e1-aef2-b7ba-03fb-b8768f0a84e1 running - connect using
  ssh <session-name>.<user-id>.<project>.ps.gadi.nci.org.au
```
I make sure the file `~/.persistent-sessions/cylc-session` contains the session name:
```
$ more ~/.persistent-sessions/cylc-session 
<session-name>.<user-id>.<project>.ps.gadi.nci.org.au
```
Then I log into the session.
```
$ ssh -Y <session-name>.<user-id>.<project>.ps.gadi.nci.org.au
```

## MOSRS authentication ##

Once you have connected to your persistent-session, you will have to load the required `rose/cylc` modules and then authenticate with the central MOSRS repository.

A successful authentication will look like.
```
 mosrs-auth
INFO: You need to enter your MOSRS credentials here so that GPG can cache your password.
Please enter the MOSRS password for <username>>: 
INFO: Checking your credentials using Subversion. Please wait.
INFO: Successfully accessed Subversion with your credentials.
INFO: Checking your credentials using rosie. Please wait.
INFO: Successfully accessed rosie with your credentials.
```

## Checking out a suite ##

Let's run a tiny test suite developed by Paul Leopardi at ACCESS-NRI. You can find the full documentation here :

https://github.com/ACCESS-NRI/training-day-2024-regional_model/blob/main/access_rose_cylc/rose_cylc_example.md

You can check out Martin's suite using the rosie `check out` command
```
rosie co u-cq161
```
Which creates a local copy of the `rose/cylc` suite to `~/roses/u-cq161`.

Let's walk through the `suite.rc` file to see what this suite will do.

The `[scheduling]` section tells us which tasks will run, and in which order. This suite only contains three tasks:
```
get_stream => build_stream => run_stream
```

The `[runtime]` section tells us **how** these tasks will run.

Firstly there are three sections (or namespaces) which define the local environment: `[[root]]`, `[[local]]` and `[[HPC]]`, where the suite will run and in this instance, the `PBS` job specifications.

There each task has its own namespace: `[[get_stream]]`, `[[build_stream]]` and `[[run_stream]]`.

The first task ``[[get_stream]]`` inherits the `[[local]]` namespace and runs the `get_stream` script after some preliminary `bash` commands.

The second task `[[build_stream]]` inherits the `[[HPC]]` namespace and executes the `build_stream` script. 

The last task `[[run_stream]]` inherits the same namespace and executes the `run_stream` script.



NOTE. You will have to restart your sessions after quarterly maintenance.