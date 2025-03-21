# Transferring data

When working on `gadi`, often you'll have to transfer large amounts of data (gigabytes or terabytes) between different disks to deal with changing project quotas.

Sometimes you may also need upload or download data from `gadi`.

Managing terabytes of data between disks is not a trivial process, for several reasons:
- Networking issues during the copying process could cause the task to fail before completion
- The task may exceed the `gadi` login node job limits (30 minutes)
- Excessive disk use can overload the `gadi` login nodes.
- Disk errors may occur, i.e. the copied file may be incomplete or corrupted.

To guard against these problems, NCI recommend the following steps.
1. When uploading or downloading data, use a dedicated `gadi` 'data-mover' node (as opposed to standard login node) for smaller, interactive tasks. Using these nodes reduces CPU and I/O loads on the login nodes.
 
:::note
You cannot remotely login to a 'data-mover' node. It can only be used for securely copying files using `scp`
:::
2. Use the `rsync` command instead of the `cp` copy command. This gives your the ability to resume an interrupted transfer.
3. For larger tasks that may take more than 30 minutes, use the `copyq` `PBS` queue and submit a batch job.

NCI have provided [this link](https://opus.nci.org.au/spaces/Help/pages/236880317/File+Transfer...#FileTransfer...-rsync) for extra information on the above points.

There is also this dedicated [link](https://opus.nci.org.au/spaces/Help/pages/236880320/Job+Submission...#JobSubmission...-copyq) with a guide on using the `copyq`job queue.

## Example

In this example, we will use the `copyq` queue to transfer about 7 Gigabytes of ERA5 anticyclone data from a `su28` data disk to a `gb02` scratch disk. 

For transferring large amounts of data, we can use the 'Message Digest Algorithm 5' or `MD5` checksum to check for data integrity. 

So firstly, I can use some `bash` logic to create `MD5` checksums for every file. First I will specify the location of the data files, and where I want the `MD5` data to be written to.
```
source_dir=/g/data/su28/weatherfeatures.era5/maxcl/cdf

md5_dir=/g/data/gb02/pag548/md5_keys

project=su28
```
Then I will loop over all the subdirectories in `source_dir` (there is one for every year from 1950 to 2022) and generate a checksum for each file in each yearly subdirectory.
```
cd ${source_dir}
for dir in *
do
    cd ${source_dir}/${dir}
    echo "$PWD find  -type f -print0 | sort -z | xargs -r0 md5sum > ${md5_dir}/${dir}_${project}_md5sum.txt"
    find  -type f -print0 | sort -z | xargs -r0 md5sum > ${md5_dir}/${dir}_${project}_md5sum.txt
done
```
The bash logic does the following:
- uses `find` to grab every filename inside the subdirectory
- pipes this list of filenames using the `|` command, to the `sort` command to sort the list of filenames into alphabetical order,
- the pipes the sorted lists into `md5sum`
- the directs the output of `md5sum` into a text file  `${md5_dir}/${dir}_${project}_md5sum.txt`

Here is an example output file:
```
more 2021_su28_md5sum.txt
87d8d3e7f429b308d4d2a44553abfd77  ./A2021_01.nc
cca154c44d474aabda928ae26789ea9e  ./A2021_02.nc
a95f322bbe0198265b78582f296166a7  ./A2021_03.nc
4d114b872b56188413c0229ec302d668  ./A2021_04.nc
621e9ddca68930440976279f585f54d5  ./A2021_05.nc
b2f8a29057b36b33e3f2ed4ef6baa884  ./A2021_06.nc
etc.
```
So the contents of every monthly netCDF file of ERA5 anticyclone data, valid for a given year, is now associated with a unique hash string.

Now, let's create a `bash` script called `copy_directory.sh` to use `rsync` to transfer the dataset between our two disks.
```
#!/bin/bash
#PBS -l mem=2GB
#PBS -l ncpus=1
#PBS -q copyq
#PBS -l walltime=2:00:00
#PBS -l storage=gdata/su28+scratch/gb02
#PBS -P gb02
#PBS -o /home/548/pag548/code/software_engineering_demos/file_transfers/md5sum_copyq_rsync/copyq.out
#PBS -e /home/548/pag548/code/software_engineering_demos/file_transfers/md5sum_copyq_rsync/copyq.err

source_dir=/g/data/su28/weatherfeatures.era5/maxcl/cdf
target_dir=/scratch/gb02/pag548/

rsync -vP -r ${source_dir} ${target_dir}
```
:::note
I have to specify the disks I want to transfer the data from using the `#PBS storage` flag.
:::
I can then submit this task to the `copyq` job queue using
```
$ qsub copy_directory.sh
```
:::note
Make sure `copy_directory.sh` has user executable permissions, i.e. it is coloured <span style="color:green">green</span> in your terminal, or has the user permissions `-rwx` when you use a 'long list' (i.e. `$ ls -l copy_directory.sh`). To make a `bash` script executable by a user, type `chmod u+x <filename>`
:::
You can check the progress of the job by typing
```
$ qstat
Job id                 Name             User              Time Use S Queue
---------------------  ---------------- ----------------  -------- - -----
137293930.gadi-pbs     copy_directory.* <your userid>                   0 Q copyq-exec  
Qs
```
In this example the job is 'queued' in the `copyq` job queue and is waiting its turn to be executed.

The job will take just over a minute to finish. If you look at your 'standard output' file (defined using the `#PBS -o` flag) it should look something like
```
======================================================================================
                  Resource Usage on 2025-03-20 15:45:45:
   Job Id:             137294139.gadi-pbs
   Project:            gb02
   Exit Status:        0
   Service Units:      0.12
   NCPUs Requested:    1                      NCPUs Used: 1               
                                           CPU Time Used: 00:00:48        
   Memory Requested:   10.0GB                Memory Used: 6.49GB          
   Walltime requested: 02:00:00            Walltime Used: 00:01:25        
   JobFS requested:    100.0MB                JobFS used: 0B              
```
If we look at these files in their new location, their permissions will have changed to reflect our own user-id and project memberships, e.g.
```
$ ls -lh 2020
total 101M
-rw-r--r-- 1 pag548 gb02 8.6M Mar 20 15:45 A2020_01.nc
-rw-r--r-- 1 pag548 gb02 7.6M Mar 20 15:45 A2020_02.nc
-rw-r--r-- 1 pag548 gb02 8.5M Mar 20 15:45 A2020_03.nc
-rw-r--r-- 1 pag548 gb02 9.1M Mar 20 15:45 A2020_04.nc
-rw-r--r-- 1 pag548 gb02 8.9M Mar 20 15:45 A2020_05.nc
etc
```
That is another advantage of using `rsync` over `cp`.

Now, to check the copied files, we can use the same logic above to generate `MD5` checksums that we used before. Except this time, we will provide the directory where the copied files are (e.g. `/scratch/gb02`) and specify the new project (e.g. `gb02`).

Now we will have two sets of `MD5` checksum files for both sets of files, i.e. the originals and their copies. In this example, you will have 
```
2022_gb02_md5sum.txt
2022_su28_md5sum.txt
```
In the directory where you created your `MD5` files for each yearly subdirectory.

Now we can use some `bash` to compare the contents of each `md5sum.txt` file and check they are identical.
```
md5_dir=/g/data/gb02/pag548/md5_keys

cd ${md5_dir}

source_project=su28
target_project=gb02

for year in {1950..2022}
do
  
	target_md5=${year}_${target_project}_md5sum.txt
	source_md5=${year}_${source_project}_md5sum.txt
	diff ${target_md5} ${source_md5}

done
```
This is one way to loop over all `.txt` files in our `md5_dir` directory and use the bash command `diff` to check the file contents are identical.

If this command exits with no output, the files have been copied correctly and you can be confident of their data integrity.

The example scripts above are available [here](https://github.com/21centuryweather/software_engineering_demos/tree/main/file_transfers/md5sum_copyq_rsync)

You can alter them to reflect the directory structures of the data you are trying to copy.

You may have to alter the `PBS` parameters in your `copyq` job if you run out of memory or exceed walltime. 
