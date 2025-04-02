(content:git-notebooks)=
# Tracking changes on jupyter notebooks

We already mentioned that one of the great advantages of working using git is that you get the history of changes for the files inside your repository. 

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
-   `git diff --staged` shows you the changes in a file when we already added it to the staging area. 

:::

It is also possible to compare different versions of a file if you know the label associated to a commit. One way to use this easily is by using the label `HEAD` that refers to the most resent commit. So by doing 

```
git diff HEAD~2 read_era5.py
```

We get the different between the current version and 2 commits before that. 

Looking at the difference between 2 versions of a file is also useful and important to review pull request. For example, the PR created during the first exercise in the [Scenario 1](content:git-team:scenario1) will look like this when we check the differences between the current version of the file and the one proposed:

```{figure} images/diff-pr.png
---
alt: "Screenshot of github.com, it shows a pull request called add plot in the tab 'Files changed', on the left github show the current version of the file and on the right shows the new version where the new lines are highlighted. Note: the github interface is not accessible to screen readers."
---
Files changed tab on a pull request in the GitHub interface.
```

In this same section you can add comments, suggestions and commits to the same branch. 

But this is only possible because the file in question is a plain text file. For images for example, git wil replace the old file with the new one, even if the changes in the image are small. And different formats will have different challenges. On the next section will see how to work with git and jupyter notebooks. 

## Tracking changes in a jupyter notebook

Notebooks are JSON files under the hood and for that reason, the diff between versions of a file will be very hard to read. Here is an example:

```bash
diff --git a/analysis.ipynb b/analysis.ipynb
index d5a4aae..efe708f 100644
--- a/analysis.ipynb
+++ b/analysis.ipynb
@@ -30,10 +30,24 @@
    "metadata": {},
    "outputs": [],
    "source": [
-    "# Load a NetCDF climate dataset\n",
+    "# Load a NetCDF file\n",
     "ds = xr.open_mfdataset(\"era5_data.nc\")\n",
     "print(ds)"
    ]
+  },
+  {
+   "cell_type": "code",
+   "execution_count": null,
+   "metadata": {},
+   "outputs": [],
+   "source": [
+    "# Plot a temperature time series\n",
+    "plt.plot(df[\"date\"], df[\"temperature\"])\n",
+    "plt.xlabel(\"Date\")\n",
+    "plt.ylabel(\"Temperature\")\n",
+    "plt.title(\"Temperature Over Time\")\n",
+    "plt.show()"
+   ]
   }
  ],
  "metadata": {
```

With the last commit we added the code to plot a temperature time series and change a comment from "Load a NetCDF climate dataset" to "Load NetCDF file". But it is hard to read in between all the metadata associated to each cell. Also, the metadata of a cell can change even if the content stay the same making the task of tracking changes challenging. 

One possibility to review changes locally is [nbdime](https://nbdime.readthedocs.io/). By using `nbdiff <commit1 sha> <commit2 sha> <path>` we get this:

```bash
nbdiff analysis.ipynb (commit1 sha) analysis.ipynb (commit2 sha)
--- analysis.ipynb (commit1 sha)  (no timestamp)
+++ analysis.ipynb (commit2 sha)  (no timestamp)
## modified /cells/3/source:
@@ -1,3 +1,3 @@
-# Load a NetCDF climate dataset
+# Load a NetCDF file
 ds = xr.open_mfdataset("era5_data.nc")
 print(ds)


## inserted before /cells/4:
+  code cell:
+    source:
+      # Plot a temperature time series
+      plt.plot(df["date"], df["temperature"])
+      plt.xlabel("Date")
+      plt.ylabel("Temperature")
+      plt.title("Temperature Over Time")
+      plt.show()

```

ndbime is good to work locally but if you or a college want to review changes or contributions on PR on GitHub, we'll need a tool like [reviewNB](https://www.reviewnb.com). This is a separate app that allows you to review Jupyter Notebooks and pull request on GitGub. 

```{figure} images/notebook-gh.png
---
alt: "Screenshot of github.com, it shows a pull request called change plot in the tab 'Files changed', on the left github show the current version of the file and on the right shows the new version where the new lines are highlighted. Note: the github interface is not accessible to screen readers."
---
Files changed tab on a pull request for a jupyter notebook.
```

This particular example is simple, we are only changing a few lines in the code and not metadata because we didn't include the output of each cell. But the diff can be more complex and difficult to read. For those cases there are different tools and solutions, for example reviewNB, This app present the diff in a tidy way:

```{figure} images/notebook-nb.png
---
alt: "Screenshot of app.reviewnb.com, it shows a pull request called change plot, on the left it show the current version of the file and highlights the parts changed and on the right shows the new version where the new lines are also highlighted. Note: accessibility has not being tested for this website."
---
Files reviewNB interface showing a pull request for a jupyter notebook.
```

To be able to use reviewNB you will need to logging on [reviewnb.com](https://www.reviewnb.com/) with your GitHub account and give reviewNB permissions to access your repositories (o or more). Then, you can navigate to the commits or pull request you want to review. You will also get an email from reviewNB every time someone opens a new pull request. Notice that here we are using the reviewNB interface instead of the GitHub website. 

reviewNB has also great resources to learn how to use it, we recommend you [starting with this section](https://www.reviewnb.com/git-jupyter-notebook-ultimate-guide).