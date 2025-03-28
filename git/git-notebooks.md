# Tracking changes on jupyter notebooks

We already mentioned that one of the great advanges of working using git is that you get the history of changes for the files inside your repository. 

For example if we were to check the difference between the first version of the file `read_era5.py` and second version when we add how to plot the data, we'll see something like this 

```bash
diff --git a/read_era5.py b/read_era5.
index bd0c3f6..d92a209 100644
--- a/read_era5.py
+++ b/read_era5.py
@@ -1,5 +1,13 @@
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

Here, `a/read_era5.py` is the *old* version and `b/read_era5.py` the new version that we'll commit to the repository. The "+" denotes the lines we added or changed and sometimes you will see s "-" for the deleted or changed lines. The second line in the output `index bd0c3f6..d92a209 100644` tells exactly which versions of the file Git is comparing; bd0c3f6 and d92a209 are unique computer-generated labels for those versions.

:::{admonition} Commands
:class: tip

-   `git diff <file>` will return the difference between the comited version of a file and the modified -or current version.
-   `git diff --staged` shows you the changes in a file when we alread added it to the staging area. 

:::

It is also possible to compare different versions of a file if you know the label asociated to a commit. One way to use this easily is by using the label `HEAD` that referes to the most resent commit. So by doing 

```
git diff HEAD~2 read_era5.py
```

We get the different between the current version and 2 commits before that. 

Looking at the difference between 2 versions of a file is also useful and importat to review pull request. For example, the PR created during the first exsercise in the [Scenario 1](content:git-team:scenario1) will look like this when we check the differences between the current version of the file and the one proposed:

```{figure} images/diff-pr.png
---
alt: "Screenshot of github.com, it shows a pull request called add plot in the tab 'Files changed', on the left github show the current version of the file and on the right shows the new version where the new lines are highlighted. Note: the github interface is not accessible to screen readers."
---
Files chanched tab on a pull request in the GitHub interface.
```

In this same secction you can add comments, suggestions and commits to the same branch. 

But this is only possible because the file in question is a plain text file. For images for example, git wil replace the old file with the new one, even if the changes in the image are small. And different formats will have different challenges. On the next section will see how to work with git and jupyter notebooks. 




The issue with notebooks
A solution