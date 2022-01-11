## Copy Output Data and Run script logs to S3 Bucket

Note, you will need permissions to copy to the S3 Bucket.

### The CTM_LOG files don't contain any information about the compute nodes that the jobs were run on.
Note, it is important to keep a record of the NPCOL, NPROW setting and the number of nodes and tasks used as specified in the run script: #SBATCH --nodes=16 #SBATCH --ntasks-per-node=8
It is also important to know what volume was used to read and write the input and output data, so it is recommended to save a copy of the standard out and error logs, and a copy of the run scripts to the OUTPUT directory for each benchmark.

```
cd /shared/build/openmpi_gcc/CMAQ_v533/CCTM/scripts
cp run*.log /fsx/data/output
cp run*.csh /fsx/data/output
```
### Examine the output files

```
cd /fsx/data/output/output_CCTM_v533_gcc_2016_CONUS_16x18pe_full
ls -lht
```

output:

```
total 173G
drwxrwxr-x 2 ubuntu ubuntu 145K Jan  5 23:53 LOGS
-rw-rw-r-- 1 ubuntu ubuntu 3.2G Jan  5 23:53 CCTM_CGRID_v533_gcc_2016_CONUS_16x18pe_full_20151223.nc
-rw-rw-r-- 1 ubuntu ubuntu 2.2G Jan  5 23:52 CCTM_ACONC_v533_gcc_2016_CONUS_16x18pe_full_20151223.nc
-rw-rw-r-- 1 ubuntu ubuntu  78G Jan  5 23:52 CCTM_CONC_v533_gcc_2016_CONUS_16x18pe_full_20151223.nc
-rw-rw-r-- 1 ubuntu ubuntu 348M Jan  5 23:52 CCTM_APMDIAG_v533_gcc_2016_CONUS_16x18pe_full_20151223.nc
-rw-rw-r-- 1 ubuntu ubuntu 1.5G Jan  5 23:52 CCTM_WETDEP1_v533_gcc_2016_CONUS_16x18pe_full_20151223.nc
-rw-rw-r-- 1 ubuntu ubuntu 1.7G Jan  5 23:52 CCTM_DRYDEP_v533_gcc_2016_CONUS_16x18pe_full_20151223.nc
-rw-rw-r-- 1 ubuntu ubuntu 3.6K Jan  5 23:22 CCTM_v533_gcc_2016_CONUS_16x18pe_full_20151223.cfg
-rw-rw-r-- 1 ubuntu ubuntu 3.2G Jan  5 23:22 CCTM_CGRID_v533_gcc_2016_CONUS_16x18pe_full_20151222.nc
-rw-rw-r-- 1 ubuntu ubuntu 2.2G Jan  5 23:21 CCTM_ACONC_v533_gcc_2016_CONUS_16x18pe_full_20151222.nc
-rw-rw-r-- 1 ubuntu ubuntu  78G Jan  5 23:21 CCTM_CONC_v533_gcc_2016_CONUS_16x18pe_full_20151222.nc
-rw-rw-r-- 1 ubuntu ubuntu 348M Jan  5 23:21 CCTM_APMDIAG_v533_gcc_2016_CONUS_16x18pe_full_20151222.nc
-rw-rw-r-- 1 ubuntu ubuntu 1.5G Jan  5 23:21 CCTM_WETDEP1_v533_gcc_2016_CONUS_16x18pe_full_20151222.nc
-rw-rw-r-- 1 ubuntu ubuntu 1.7G Jan  5 23:21 CCTM_DRYDEP_v533_gcc_2016_CONUS_16x18pe_full_20151222.nc
-rw-rw-r-- 1 ubuntu ubuntu 3.6K Jan  5 22:49 CCTM_v533_gcc_2016_CONUS_16x18pe_full_20151222.cfg
```

Check disk space

```
 du -sh
173G    .
```

### Copy the output to an S3 Bucket

Examine the example script

```
cd s3_scripts
cat s3_upload.c5n.18xlarge.csh

```

output:

```
#!/bin/csh -f
# Script to upload output data to S3 bucket
# NOTE: a new bucket needs to be created to store each set of cluster runs

aws s3 mb s3://c5n-head-c5n.18xlarge-compute-conus-output-01-05-2021
aws s3 cp --recursive /fsx/data/output/ s3://c5n-head-c5n.18xlarge-compute-conus-output-01-05-2021/fsx/data/output
```

Edit the script to include a new date stamp, then run the script.

```
./s3_upload.c5n.18xlarge.csh
```