# Debugging Python with VSCode

Let's load and run the file `mean_air_temp.py` from the https://github.com/21centuryweather/software_engineering_demos repo.  (It will be in the `CoE_workshop_2025` subdirectory). This is a trivial test script which
- Loads in some sample air temperature data.
- Uses a user-defined function `calculate_mean_loop` to compute the temporal average for the first 10 days of the dataset.
- Then compares it against the `xarray` built-in method `.mean(dim='time')` and outputs the difference.

Ideally these two averages should be the same. Yet the user-defined function differs. Why? Let's use a debugger to find out.

## What is debugging?

Debugging is an investigation to try and find the source of 'bugs' or errors in code. The bugs may caused by:
- syntax errors : we've typed the wrong symbol, or used the wrong order of symbols, so the code functions differently to its intended purpose.
- logic errors : the code is mechanically correct but it carries out the wrong order of tasks, creating the wrong answer.
- semantic errors : the code is working correctly but it has incorrect names and labels assigned to it variables or functions.

We have seen previously that a Linter will detect and fix basic syntax errors that will prevent a script from running. But if the script runs, but produces incorrect results, we will need to conduct a forensic investigation to ensure how we **think** the code is executing is the same as how it **actually** executes.

Typically, undergraduate coding involves using lots of 'print' statements to standard output to view the values of parameters and variables as they change throughout the code.

But once our codes grow in complexity, our debugging methods must also grow in sophistication.

## What is a debugger?

A debugger is a module or executable that allows the code to be executed, line-by-line, and will display the value of a variables at each line.

Key concepts involved in debugging:
- **Breakpoints** : A long script may involve thousands of lines spread across multiple files. We don't want to examine every one. So we place a breakpoint at a specific line number, which tells the debugger to 'stop the program at this point'
- **Step Over** : Once we have stopped program execution at a breakpoint, we will 'step over' individual lines of code, one-at-a-time. If we encounter any function calls at a single line, we will 'step over' them.
- **Step Into** : If we encounter a function call on a single line, we will 'step into' that function and proceed with a line-by-line analysis.
- **Continue** : Cease our line-by-line analysis, and continue to the next breakpoint, or until the program ends (successfully or otherwise). 

## First steps, using Pdb

Python comes with its own debugging module - [`pdb`](https://docs.python.org/3/library/pdb.html). This runs inside a terminal so it doesn't have all the visualisation tools that an IDE debugger would. But it's a good place to start.