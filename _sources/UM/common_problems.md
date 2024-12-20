# Common rose/cylc problems

This will be a maintained list of problems encountered (and fixed) with running `rose/cylc` ACCESS tasks.

## Jobs submitted but not running

A common issue is that you start your `rose` suite, which submits various jobs, but they never run.

i.e. the job state in the `cylc` GUI is 'submitted' and the job colour is bright green, but the job never begins running and never turns deep green.

This is often caused by disk quota issues. If you have insufficient disk space, either in your home directory, your `/g/data/` or `/scratch` directories, your PBS job will not be allowed to run and will be held indefinitely.

PBS jobs will also be held if there are insufficient project allocations (e.g. you've run out of 'kSU').

See https://opus.nci.org.au/spaces/Help/pages/230490410/Why+are+my+jobs+not+running+...

for more details.

To help manage your disk quota, use some of the commands in this link:

https://opus.nci.org.au/spaces/Help/pages/236880003/Basic+Navigation...

## Cannot login to a persistent session

If you can't login to your persistent session, you may have conflicting modules which don't use the standard `ssh` environment at NCI. Often the cause of this conflict is the `conda/analysis` module developed by CLEX and maintained by ACCESS-NRI.  You will have to unload any conflicting modules.

See [this link](../mosrs/mosrs-intro.md#persistent-sessions) for more information.

## Persistent session cannot see files in /g/data/access

If you are using a persistent session created before you were a member of the `access` project, your ACCESS suite will be unable to locate any files on `/g/data/access` and hence will fail.

The solution to his problem is to delete your persistent session and create a new one, after you have joined the `access` project.