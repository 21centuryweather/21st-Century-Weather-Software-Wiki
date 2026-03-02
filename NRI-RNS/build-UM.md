# Custom UM builds for rAM3

The Regional Nesting Suite (RNS) contains the ability to build a custom UM executable. Let's work through the steps required to compile a local exectuable for your own needs.

## Setting the build flag

In the `rose edit` GUI, change the `BUILD_MODE` option in the `General Run options` to `Build new executable`.

You can also change 
```
BUILD_MODE="new"
```
in `rose-suite.conf`.

Now run your suite hold all tasks.
```
$ rose suite-run -- --hold
```
You will notice a new family of tasks in the GUI: `BUILD`. This task family contains two tasks : `fcm_make_um` and `fcm_make2_um`.