# Introduction to Rose/Cylc

Running an atmospheric simulation is a complex process which requires the completion of many separate tasks. In order to co-ordinate these tasks, we requirer a job scheduler, or a **workflow engine**.

## Shell scripts and running jobs on Linux

Typically we run programs (or apps) on a linux computer using a shell scripts. We use the `bash` shell to interpret input from the command-line to execute programs.

For example, if you login to `gadi` and type `ls -la`, at the command-line you are executing the program `ls` with the additional arguments `la`. The output of the program is then directed via standard output to your terminal.

We can then create bash 'scripts' which allows to us process multiple inputs and to automate tasks from the command line instead of having to type everything manually.  A bash script is simply a text file with a list of commands, which we give special permissions to which allow the Linux operating system to execute it as a program.

If you want more familiarity with `bash`, there are many on-line tutorials and references available, e.g.:

https://www.freecodecamp.org/news/bash-scripting-tutorial-linux-shell-script-and-command-line-for-beginners/

or

https://www.geeksforgeeks.org/bash-scripting-introduction-to-bash-and-bash-scripting/

You can follow those tutorials on `gadi` but you will need to use a different text editor to `gedit` as it is not installed. You can use `vi` if you're familiar with it, otherwise you can use `nano` which will run inside a terminal window, or type `nedit &` to launch the `nedit` GUI text editor.  The `&` tells bash to run this job 'in background', which allows you to continue typing inputs via the command-line while the `nedit` window is active.

A deeper `bash` reference can be found here:

https://learn.microsoft.com/en-us/training/modules/bash-introduction/

Depending on your prior experience with Linux and `bash`, you may want to spend a few days working through these tutorials and examples to gain a better understanding of how `bash` commands and `bash` ENVIRONMENT work and familiarise yourself with the Linux directory structures on `gadi`.

Typically you will use `bash` scripts to execute programs required to simulate the atmosphere, whether these programs are pre-compiled executables (e.g. the UM itself), python scripts for pre-processing or post-processing, or bash commands themselves.

When we execute programs from the command-line, we are using an 'interactive' session. Typically this is used for small programs that require very few resources (memory, processors, disk space) and can be executed in a few seconds or minutes. For larger tasks, a super-computer uses a batch-scheduling system whereby `bash' scripts are submitted to a job scheduler queue with requests for memory, processors and storage. The job scheduler then processes each jobs when resources become available.

The NCI supercomputer `gadi` uses the `PBS` job scheduler. Documentation is available here:

https://opus.nci.org.au/pages/viewpage.action?pageId=236880320

But, what if we have a complex set of tasks that must run in a particular sequence? Can we create programs which processes `PBS` jobs in a user-specified workflow?  

## Task Scheduling for Atmospheric Simulation

A good example of a complex tasks that must run in a particular sequence is a  realtime atmospheric simulation (i.e. a weather forecasts).  Typically for a longer, multi-day forecast, the sequence of tasks involves:
1. Reading the previous weather forecast data initialised some three hours ago
2. Collect observations valid from three hours ago, to three hours in the future
3. Running a perturbation forecast model which uses an optimisation method to determine the initial condition which minimises the error between the short-term forecasts and observations over a six hour period. 
4. Use the optimal initial condition which minimises short-term error growth to run another short term forecast. These short term forecasts which have been computed against observations are known as 'analysis' or 'analyses'. They are our best estimate of the three-dimensional structure of the atmosphere at any point in time. 
5. Repeat the analysis computation every six hours.
5. Every 12 hours, run a longer forecast (e.g. 7 days into future)

Repeating this process in realtime 24/7 requires the interaction of many separate tasks. We need a system which submits programs, monitors their progress, keeps track of the current time and executes each task according to an ordered set of dependencies (i.e some tasks cannot run until some upstream tasks have finished). 

In order to do this, we require what is known as a workflow engine, which is a fancy name for a task scheduler.  

## The Cylc work engine 

Recently, the New Zealand National Institute of Water and Atmospheric Research (NIWA) developed its own task scheduler to handle its operational workflows - **cylc**. This scheduler was so elegant, powerful and (relatively) simple to use that it was adopted by the UK Met Office, who wrapped an external software layer - **rose** - around the `cylc` engine. 

Every time you run an atmospheric simulation using the UK Met Office `Unifed Model` (`UM`), 

When we refer to `rose/cylc`, we are referring to a `rose` framework (which constitutes various GUI tools and scripts) which launch the `cylc` workflow engine.

To understand how `cylc` works, visit the documentation here:

https://cylc.github.io/cylc-doc/7.9.3/html/index.html

Note the `gadi` is still using an earlier version of `cylc` (7.9.3) so make sure 

### First cylc tutorial ###

Let's run through the tutorial here: 

https://cylc.github.io/cylc-doc/7.9.3/html/tutorial.html

To launch `cylc` on `gadi` you need to load the `cylc` software into your interactive command-line session.
```
module use /g/data/hr22/modulefiles
module load cylc7/23.09
```
Follow the instructions in section 7.6 - 7.9 of the tutorial.

```
$ cylc import-examples /tmp
```

This will create a list of cylc examples files in 
```
~/cylc-run/examples
```
Remember `~` is a short-cut which refers to your home directory on `gadi`.

All your future `rose/cylc` tasks to run the `UM` will also be deployed in your `~/cylc-run` directory.

Let's run the 'Hello World' example. Go to the following path in `~/cylc-run` directory (the path is actually different to the official documentation):
```
cd ~/cylc-run/examples/7.9.7/tutorial/oneoff/basic
```
Using a text editor of your choice (`vi`, `nedit`, `nano`, `emacs`, `Microsoft VS Code`), copy the contents from the tutorial into a file named `suite.rc`.

Once complete, if you examine the contents of this file using the bash command-line program `more`, it should resemble this:
```
 $ more suite.rc 
[meta]
    title = "The cylc Hello World! suite"
[scheduling]
    [[dependencies]]
        graph = "hello"
[runtime]
    [[hello]]
        script = "sleep 10; echo Hello World!"
```
Ok, let's run this from the command line. Your output should contain the following.
```
$ cylc run

Loading cylc7/23.09
  Loading requirement: mosrs-setup/1.0.1
            ._.                                                       
            | |                 The Cylc Suite Engine [7.9.7]         
._____._. ._| |_____.           Copyright (C) 2008-2019 NIWA          
| .___| | | | | .___|   & British Crown (Met Office) & Contributors.  
| !___| !_! | | !___.  _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _
!_____!___. |_!_____!  This program comes with ABSOLUTELY NO WARRANTY;
      .___! |          see `cylc warranty`.  It is free software, you 
      !_____!           are welcome to redistribute it under certain  

*** listening on https://pgregory.pag548.gb02.ps.gadi.nci.org.au:43080/ ***

To view suite server program contact information:
 $ cylc get-suite-contact basic
```
Congratulations! You've just run your first `cylc` suite. 

### Second cylc tutorial ###

Let's move onto another `cylc` tutorial, which is located in the `rose` documentation. This tutorial deals with creating graphs, i.e. how we dictate the sequence in which tasks are run. The version of `rose` currently running supported on `gadi` for ACCESS-UM experiments is `2019.01.7` so make sure you using the following URL : 

https://metomi.github.io/rose/2019.01.8/html/tutorial/cylc/scheduling/graphing.html

Let's follow the Practical section at the bottom of that page.

**Remember!!** we have to use the `module use` and `module load` commands to add `rose` and `cylc` to our paths on gadi, in case you are startingn this tutorial from a fresh login session.
```
module use /g/data/hr22/modulefiles
module load cylc7/23.09
```

Back to the tutorial:
```
mkdir ~/cylc-run/graph-introduction
cd ~/cylc-run/graph-introduction
```
Create a `suite.rc` file in this directory using the template provided, and then create a task graph using the necessary syntax. When you execute
```
cylc graph .
```
the results show match the below image.

![Simple cylc graph](images/cylc-graph-tutorial.png)

### Third cylc tutorial ###

Read through the information available here : 

https://metomi.github.io/rose/2019.01.8/html/tutorial/cylc/scheduling/integer-cycling.html

This is important as it describes how the same sequence of tasks can be easily repeated. For a weather or climate simulation, you will be running a set of `UM` tasks that will need to be repeated every hour, six hours, every day, or every few days, depending on your problem. You may need to repeat these tasks for weeks, months, years or decades depending on your application.

Regardless of your application and temporal resolution, you will need to be acquainted with how `cylc` handles "cycling" or "repeating" workflows. This will help you understand how to diagnose errors in task flows should they arise, or how to add new tasks if required.

Run through the steps in the practical until your graph resembles the below imgaes.

![Simple cylc cylcing graph](images/cylc-cycle-tutorial.png)

### Fourth cylc tutorial ###

Let's move onto the next tutorial which looks at Date-time cycling, i.e. controlling task flow using real date-time objects. This is how your weather and climate `rose/cylc` suites will be controlled.

Access the fourth tutorial here:

https://metomi.github.io/rose/2019.01.8/html/tutorial/cylc/scheduling/datetime-cycling.html

Remember to load the rose and cylc executables into your current `gadi` environment using the `module load` commands.

> **_TIP:_**
> You can use the `bash` `alias` function to write a simple macro to load the `rose/cylc` environment with a simple command.
> e.g. in my `~/.bash_profile` file I declare the following:
> 
> ```alias start_rose="module use /g/data/hr22/modulefiles;module load cylc7/23.09"```
> Then if I'm at the terminal and I want to use `rose/cylc` I can just type the following:
> 
> ```
> $ start_rose
>   Loading cylc7/23.09
>      Loading requirement: mosrs-setup/1.0.1
>```

Work your way through the practical exercise, which in this case is a simple taskflow for weather forecasting.

Your final `cylc` graph should include the following sections:

![Simple cylc NWP graph 1](images/cylc-datetime1.png)
![Simple cylc NWP graph 2](images/cylc-datetime2.png)
![Simple cylc NWP graph 3](images/cylc-datetime3.png)

### Fifth cylc tutorial ###

Now that we understand how create a workflow in `cylc` where the workflow is defined as a sequence of tasks and dependencies, how do we determine what each task executes?

In this section you will learn how to assign executables (e.g. `bash` scripts, python scripts, or `UM` binaries) to a task.

Read through the following tutorial : 

https://metomi.github.io/rose/2019.01.8/html/tutorial/cylc/runtime/introduction.html

You will also learn many important tasks including:
- How to run a complex suite 
- How to use the cylc GUI to monitor task flows within a suite
- Understand the `~/cylc-run/<suite-name>/` directory structure to find log files, temporary input and output files and ancillary data.

When running this sample suite, your cylc GUI will resemble the following (assuming you have activate "graph view" in the second view using the buttons in the toolbar).

![Simple cylc runtime](images/cylc-runtime-tutorial.png)

You can inspect other job log files, e.g the `get_rainfall` task, e.g.
```
 more log/job/20000101T0600Z/get_rainfall/NN/job.out 
Suite    : runtime-introduction
Task Job : 20000101T0600Z/get_rainfall/01 (try 1)
User@Host: pag548@pgregory.pag548.gb02.ps.gadi.nci.org.au

2024-10-28T05:54:42Z INFO - started
2024-10-28T05:54:43Z INFO - succeeded
```
> **_NOTE:_** In the `cylc` directory structure, the subdirectory `NN` will always point to the latest job submission number. Logfiles from previous job submission attempts are retained to help keep track of earlier failures. Keep this in mind when you have to repeat a job submission because earlier jobs failed because of an error. 

### Sixth cylc tutorial ###

The next tutorial expands our `cylc` knowledge to include runtime configuration. How do we use `bash` environment variables to control the way the suite runs? E.g. can we use environment variables to specify the location of input data files?

Work your way through the following exercises:

https://metomi.github.io/rose/2019.01.8/html/tutorial/cylc/runtime/runtime-configuration.html

As mentioned earlier in the section on [running jobs on Linux](#shell-scripts-and-running-jobs-on-Linux) running on gadi, `cylc` is configured to use the `PBS` batch system to schedule jobs.

View the standard output from the 

What is the link wit