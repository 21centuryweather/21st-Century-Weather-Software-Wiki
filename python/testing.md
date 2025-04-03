# Testing your code

There are many instances in the course of your research where you will have to rely on someone else's code, or share your own code, e.g. hackathons. 

But how can you trust that someone else's code works? If someone has modified your original code, how do you ensure it still works?

This is one of many use-cases for the creation of code tests. If we build automated tests of code we write, this allows us to:
1. Install code from other researchers and test it works in our environment.
2. Collaborate on developed code, with a set of benchmarks that prevent researchers from accidentally introducing errors.

Testing is also great for your own code development. We've all had situations were we accidentally introduced a bug into our own scripts and it took us a day to chase it down. Building and maintaining tests of your code will instantly help identify the source of the error.

During code development, we often build small test cases to check our algorithm. Once we are confident that it works, we usually extend our code to work across larger datasets. Creating a 'unit test' (a test on a small function) is just a formalised way of integrating testing and checking into our code. 

Fortunately, `python` has a fantastic integrated testing framework called `pytest`. The official documents are available [here](https://docs.pytest.org/en/stable/index.html) and the internet is full of great resources, e.g. [here](https://pytest-with-eric.com).

## Examples with pytest

Let's return to the sample solar code we created earlier for our parallelization example [here](../python/concurrent_futures.md#try-it-yourself).

### Testing your own functions

The solar data is located across various directories which correspond to different product version numbers. Depending on the day you are analysing, your input directory will change.

The original code contained some simple logic to return the correct version string.
```python
    valid_date = dt.datetime(2019,4,1)
    if date < valid_date:
        return 'v1.0'
    elif date >= valid_date:
        return 'v1.1'
```
This simple statement applies compares two python `datetime` objects. It assumes the variable `date` is a valid `datetime` object. However, what will happen if `date` is not a `datetime` object? Would you end up in the wrong directory?  Would the script throw an error and stop? 

As this script isn't mission critical (e.g. nothing will blow up if it stops) we can enforce a simple `type` check of `date`, and then send the user a useful error message upon exit. We can add this logic before we conduct the `datetime` comparison.
```python
    # Check input date is a datetime object
    if isinstance(date,dt.datetime):
        pass
    else:
        LOG.fatal(f'{date} is not a datetime object')
        LOG.fatal('Check the processing of datetime objects for irradiance retrieval')
        raise SystemExit
```
The full code is available [here](https://github.com/21centuryweather/software_engineering_demos/blob/main/solar_example/solar_example/load_data.py)

So this function `get_version()` is rather critical. We don't want to load the incorrect data for a given day. So a good test of this function would involve:
- Applying a range of dates and checking the returned version string is correct
- Anticipating possible errors by the user, e.g. entering the data as a string or integer instead of a `datetime` object.

A simple test function could look something like this
```python
    dates = [dt.datetime(2019,3,30),
             dt.datetime(2019,3,31),
             dt.datetime(2019,4,1),
             dt.datetime(2019,4,2)]

    versions = [ get_version(date) for date in dates ]

    assert versions == ['v1.0', 'v1.0', 'v1.1', 'v1.1']

```
So we give the function a range of days across the threshold date of April 1 2019. We expect the version for the last two days of March 2019 will be `v1.0`, and the days afterward will be `v1.1`.  The python `assert` statement will return `True` if the condition is met, or raise an `AssertionError` if not.

Once you wrote this simple test function, you can wrap some `pytest` infrastructure around it. Create a new function beginning with `test_` in a file named `test_<function>.py`. [Here](https://www.skill2lead.com/pytest/pytest-naming-conventions.php) is an example of the `pytest` naming conventions. These conventions allow `pytest` to automatically scan files, typically located in a `tests` directory, and run all your test functions. 

If we wanted to test the other fuctionality of our `get_version()` function, we can use `pytest` functions to check the function exits smoothly when bad inputs are given.
```python
    # Test the SystemExist for bad values
    with pytest.raises(SystemExit) as pytest_exit:
        get_version(20190401)
    assert pytest_exit.type == SystemExit

    with pytest.raises(SystemExit) as pytest_exit:
        get_version('20190401')
    assert pytest_exit.type == SystemExit
```

### Testing external modules

We can also use `pytest` to make our code resilient to changes and upgrades of common libraries and modules we use in our research. For example you might develop a script which is heavily reliant on `pandas` for data processing, or `eofs` for computing Empirical Orthogonal Functions. You may move onto another task for a six months or a year, and then come back to this script and run it using another, up-to-date `python` environment.

But what happens if the `pandas` or `eofs` functions you've used have changed? Maybe `pandas` or `eofs` have undergone significant upgrades. The functions may no longer exist. Or their input arguments may have changed. Are you able to replicate the same `python` environment that existed last year? But is that compatible with the latest module your using for another, related task? If you have to make changes to your code so that it runs in your new environment, can you be sure you got the same answers that you did last year?

By building tests around functions which rely on imported, external modules, we provide a benchmark that can be referred to in the future. We can also quickly identify the source of any future problems, if our `tests` have sufficient fidelity to test every external function call.

Looking back at the solar example, it uses a function `tilting_panel_pr` which relies on several functions of the external `pvlib` library. These includes calls to:
- `pvlib.pvsystem.retrieve_sam`
- `pvlib.solarposition.get_solarposition`
- `pvlib.tracking.singleaxis`
- `pvlib.irradiance.get_extra_radiation`

etc.

We can quickly build a test for `tilting_panel_pr`.  In the research script, this is applied to arrays extracted from a large dataset of 10 minute daily data (with dimensions 103 x  1726 x 2214). But we can test the function with a much smaller subset. We can extract a small subset of this data to provide as inputs to `tilting_panel_pr`.

INSERT FULL VERSION

```python

### Putting it all together

    actual_ideal_ratio = tilting_panel_pr(
        pv_model = 'Canadian_Solar_CS5P_220M___2009_',
        inverter_model = 'ABB__MICRO_0_25_I_OUTD_US_208__208V_',
        ghi=ghi_clean,
        dni=dni_clean,
        dhi=dhi_clean,
        time=time_1d_clean,
        lat=lat_1d_expanded_clean,
        lon=lon_1d_expanded_clean
    )  
```
The full test file is `test_data.py` inside `solar_example/tests`.  You can run the tests from the `solar_example` directory simply by executing
1.  `$ source env.sh`
2. `pytest`

`pytest` is smart enough to search for all the necessary files and run all available tests.

However, if we want nicer outputs via logging, we can create a `pytest.ini` file and specify logging outputs (by default, `pytest` will suppress logging). Then we can call

3. `pytest tests`

And this will produce output with more information. 

In this case, our tests which involve the `pvlib` library are rather coarse. If this library was an early, prototype library (i.e. version number < 1.0) we might extend our test coverage and build unit tests for every `pvlib` function we call. 

:::{note}

A unit test refers to testing a single, irreducible function. I.e. testing the smallest component of code possible.
:::

If we are confident that `pvlib` is stable and that any future changes will be incremental, we can use our above test as a 'smoke test' which tests the integration of multiple function calls.

## Further concepts and reading

This is a very quick review of testing concepts. Testing itself is a whole programming discipline, and `pytest` contains functionality for government agencies and large corporations to maintain mission-critical production code containing millions of lines of code.

You don't need to understand or use all of these capabilities, but a simple application of basic testing will serve your well in the long run. It will help you maintain your research code over the life of your project.

There are other `pytest` features which you may want to use in future, such as `fixtures` and `mocks`. This [blog post](https://medium.com/worldsensing-techblog/tips-and-tricks-for-unit-tests-b35af5ba79b1) contains some examples.

`Fixtures` can be used to generate a set of input arguments that can be passed to your suite of test functions so they can in turn, test your functions imported from your main code base directory.

### Unit tests vs Smoke tests vs Regression tests

As mentioned before, unit tests apply to the smallest components of code. If we want to test how we combine multiple functions, that is usually referred to as a 'smoke' test. A complete end-to-end test of a production workflow is usually referred to as a 'regression' test. There are other intermediate steps too, such as 'acceptance' tests.

See [here](https://dev.to/crazyvaskya/a-beginners-guide-to-testing-unit-smoke-acceptance-4ngh) and [here](https://keploy.io/blog/community/developers-guide-to-smoke-testing-ensuring-basic-functionality) for further discussions.

### Code coverage

'Code coverage' is an important concept in code testing. Ideally, we want to cover as many lines as code with tests. There are various `python` plugins which can tell us how much of our code has been tested. See this [blog post](https://medium.com/@keployio/mastering-python-test-coverage-tools-tips-and-best-practices-11daf699d79b) for details.

A popular tool for measuring code coverage is https://coverage.readthedocs.io/en/7.8.0/

### Test driven development

At the most extreme example, when building and maintaing production code, you may write the tests before you actually write the code. This is known as 'test-driven development'. The overheads in using this kind of development make it not suited to a research environment. But its an interesting example of how you can invert a programming paradigm. 

Here is quick [blog post](https://medium.com/@muirujackson/python-test-driven-development-6235c479baa2) with `python` examples for 'test-driven-development'. 

A detailed set of tutorials, along with a discussion on the the pros and cons of 'test-driven-development' is available [here](https://pytest-with-eric.com/tdd/pytest-tdd/#What-You’ll-Learn). 