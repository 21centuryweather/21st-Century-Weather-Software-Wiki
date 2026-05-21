# direnv

direnv is a small bash program that allows you to modify the environment automatically as you enter a directory. 
This means that if you’ve got a project that uses a particular set of modules you can just add a `.envrc` file in the root of your project with all the module load commands that you need and never miss them.

For example, for those running ACCESS models, having a `.envrc` in the `roses/` folder can be very useful:

```bash
module use /g/data/hr22/modulefiles
module load cylc7
export PROJECT=gb02
```
So every time you `cd` into your `roses/` folder, `cylc7` will get loaded automatically, giving you access to the `rose` commands, and your default project will be switched to gb02.
When you leave the directory, everything will be unloaded, making this *environment* independent of the rest.

You can use direnv to automatically activate the python environment for the project. 
If you followed [our guide to build virtual environments on top of analysis3]((content:conda-env)), you can add a `.envrc` file in the same folder with

```bash
module use /g/data/xp65/public/modules
module load conda/analysis3
source .venv/bin/activate
```

And if you are just using analysis3, you can omit the last line. 

## Installation 

direnv is already available if you are a member of gb02. 
To use it you'll need to change your `~/.bashrc` to add:

```bash
prepend_path PATH /g/data/gb02/public/tools/direnv_bin/
eval "$(direnv hook bash)"
```

After that, run `source .bashrc` or open a new terminal window and you're done!

If you are not a member of `gb02` or you prefer to install direnv in your own home directory, follow [the instructions in the official website.](https://direnv.net/)

## Using direnv

To use direnv in a project folder you need to create a `.envrc` file. You can use one of the examples above as a starting point. 

In a `.envrc` file it is possible to load any module (including `conda/analysis3`), define system variables and add things to your `PATH`. 

1. Create a `.envrc` file inside the project folder

2. Add the modules and variables you need. Save and close the file.

3. Run `direnv allow` to *approve* the new environment. You will need to repeat this every time you update the `.envrc` file. 

4. And done, enjoy your by-project environments!