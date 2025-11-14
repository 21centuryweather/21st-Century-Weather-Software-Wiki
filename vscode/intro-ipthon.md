# Developing python code

Python is an open-source scripting language that has found huge appeal across a wide range of applications including web and internet development, database management, network programming and education. It has also became the default choice for scientific and numerical programmers all over the world. See [here](https://www.python.org/about/) for more information.

The easiest way to start coding in python is to simply start at a terminal command line. E.g. from a Mac OS terminal.
```
% python3
Python 3.14.0 (main, Oct  7 2025, 09:34:52) [Clang 17.0.0 (clang-1700.0.13.3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> print ('hello world')
```
As straightforward as this is, trying to build complicated scripts and projects purely from a terminal is difficult. Typically, we will start writing our python script in a text editor, and then execute this script (or sections of it) inside a python session to produce our desired outputs.

To help do this, there is the `IPython' tool

## IPython - or Interactive Python.

IPython is a powerful interactive shell that also provides the underpinnings of python Jupyter notebooks. While python notebooks are a fantastically powerful tool, they come with require large overheads and can sometimes promote bad programming practises.

Jupyter notebooks are fantastic for visualisation and for creating workflows. But if you are building code from scratch, we recommend using IPython.

The advantages of using IPython over a standard python session are numerable, and include:
- Smart autocompletion and tab completion to find function names, object methods/attributes and file paths within your code
- 'magic' commands (prefixed with `%`) that provide extra functionality such as
    -  %run script.py: Execute a Python script in your interactive session.
    -  %timeit expression: Benchmark the execution time of a line of code.
    - %paste: Paste multi-line code blocks without indentation issues.
    - %env: View/set environment variables.
- Shell integration using `bash` commands (e.g. `ls`, `pwd`) so you don't have to swap from your python session to a shell terminal.
- Syntax highlighting and output formatting.

IPython is installed with any of the standard `analysis3` environments on gadi. Just type `ipython` after loading your `xp65` `analysis3` environment.
```
$ ipython
Python 3.11.13 | packaged by conda-forge | (main, Jun  4 2025, 14:48:23) [GCC 13.3.0]
Type 'copyright', 'credits' or 'license' for more information
IPython 8.37.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: ls
logger_test.py  sample.py
```
If you are running on your own laptop or PC, you will need to install IPython yourself.

Documentation for IPython is available [here](https://ipython.readthedocs.io/en/stable/index.html), which includes some [tutorials](https://ipython.readthedocs.io/en/stable/interactive/tutorial.html) and [installation instructions](https://ipython.readthedocs.io/en/stable/install/install.html).

So we can quickly develop python code by writing a script in a text editor and executing the whole script inside IPython using `%run`, or we can paste multiple blocks of code into IPython and check the values of the variables. 

As an example, you can clone the repository https://github.com/21centuryweather/software_engineering_demos and open the file `CoE_workshop_2025/scripts/data_processor.py` in a text editor (or a web browser) and copy the script into an IPython session.
```python
In [7]: import time
   ...: 
   ...: def process_data(data_list):
   ...:     """
   ...:     Simulates a data processing task.
   ...:     """
   ...:     total = sum(data_list)
   ...:     # Simulate a time-consuming operation
   ...:     time.sleep(0.01)
   ...:     average = total / len(data_list)
   ...:     return total, average
   ...: 
   ...: if __name__ == "__main__":
   ...:     test_data = list(range(10000))
   ...:     start_time = time.time()
   ...:     result_sum, result_avg = process_data(test_data)
   ...:     end_time = time.time()
   ...: 
   ...:     print(f"Data processed: {len(test_data)} items")
   ...:     print(f"Total Sum: {result_sum}")
   ...:     print(f"Average: {result_avg:.2f}")
   ...:     print(f"Execution Time (Standard): {end_time - start_time:.4f} seconds")
   ...: 
Data processed: 10000 items
Total Sum: 49995000
Average: 4999.50
Execution Time (Standard): 0.0126 seconds
```
Or you can execute the entire script by providing the path to the `%run` magic command, e.g.
```python
In [1]: %run ~/code/software_engineering_demos/CoE_workshop_2025/scripts/data_processor.py
Data processed: 10000 items
Total Sum: 49995000
Average: 4999.50
Execution Time (Standard): 0.0102 seconds
```
We can run other 'magic' functions, e.g. 
```python
In [3]: test_data = list(range(10000))

In [4]: %timeit process_data(test_data)
10.3 ms ± 10.3 μs per loop (mean ± std. dev. of 7 runs, 100 loops each)
```
And inspect the variables in the script, e.g.
```
In [6]: type(test_data)
Out[6]: list

In [7]: test_data[:5]
Out[7]: [0, 1, 2, 3, 4]
```

But is there a way to improve this workflow further? Can we integrate our script editing with a current python session?

## Using VSCode as an Integrated Development Environment