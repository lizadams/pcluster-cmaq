Performance Optimization

## ParallelCluster Configuration

Selection of the compute nodes depends on the domain size and resolution for the CMAQ case, and what your model run time requirements are.
Larger hardware and memory configurations may also be required for instrumented versions of CMAQ incuding CMAQ-ISAM and CMAQ-DDM3D.
The ParallelCluster allows you to run the compute nodes only as long as the job requires, and you can also update the compute nodes as needed for your domain.

## CMAQv5.4 Benchmarks

| Benchmark Name | Grid Domain | Recommended EC2 Instance| vCPU   |  Cores | Memory | EFA Network Bandwidth | Storage (EBS Only) | On Demand Hourly Cost | Spot Hourly Cost | Region | Time (hr) per Simulation Day | Cost per Simulation Day |
| -------------- | ----------- | ----------  | ------ | ---    |----    | ---------------       | ----  | -------------------   | -------------    | ----- | --- | --- |
| Training 12km Listos | (25x25x35)   | c6a.2xlarge    | 8 | 4 | 16 GiB | Up to 12500 Megabit | gp3 | 0.306 | 0.2879 | anywhere | .0459 | $.014 |
| 12NE3                | (100x100x35) | c6a.8xlarge   | 32  | 16 | 64 GiB | 12500 Megabit  | gp3 | 1.224  | 1.0008 | anywhere | .274    | $.335 |
| 12US1                | (459x299x35) | c6a.48xlarge | 192 | 96|  384 GiB | 50000 Megabit  | gp3 | 7.344  | 5.5809 | anywhere | .827    | $6.07 |
| 12US1                | (459x299x35) | hpc6a.48xlarge | n/a  | 96 | 384 GiB | 100 Gbps     | gp3 | 2.88   | n/a    | us-east-2b | .877 | $2.53 |
| 12US1                | (459x299x35) | hpc7g.8xlarge | n/a | 64 (2x32)  | 256 GiB      |  200 Gbps | gp3 | 1.6832*2nodes | n/a    | us-east-1  | .855 | $2.87 |


```{note}
*Hpc6a instances have simultaneous multi-threading disabled to optimize for HPC codes. This means that unlike other EC2 instances, Hpc6a vCPUs are physical cores, not threads.  *Hpc6a instances available in US East (Ohio) and GovCloud (US-West) *HPC6a is available ondemand only (no spot pricing)
```

```{note}
*Two hpc7g.8xlarge nodes with 32 cores/node can run the 12US1 case as it has 256 GiB memory. hpc7g.16xlarge with 64cores/node only has 128 GiB memory, and can't run the 12US1 case on 1 node
```

```{note}
*hpc7g instances have simultaneous multi-threading disabled to optimize for HPC codes. The instances with fewer cores, 16, 32 pes are custom to only those instances, you are not sharing a slice of an instance (this also removes the need for pinning).
<a href="https://aws.amazon.com/blogs/hpc/application-deep-dive-into-the-graviton3e-based-amazon-ec2-hpc7g-instance/">hpc7g offers 16, 32 or 64 physical cpu instance size at launch</a>
```

```{note}
If you are using SPOT pricing, ie. for the c6a.48xlarge compute nodes.
Sometimes, the nodes are not available for SPOT pricing in the region you are using.
If this is the case, the job will not start runnning in the queue, see AWS Troubleshooting.
<a href="https://docs.aws.amazon.com/parallelcluster/latest/ug/troubleshooting.html">ParallelCluster Troubleshooting</a>
To avoid this, use the hpc EC2 instances, ie. hpc6a.48xlarge or hpc7g.16xlarge.
```


Data in table above is from the following:
<a href="https://calculator.aws/#/addService/ec2-enhancement?nc2=h_ql_pr_calc">Sizing and Price Calculator from AWS</a>

![Extent of Domain for the Benchmarks](CMAQv5.4_Benchmark_Domains.png)

### An explanation of why a scaling analysis is required for Multinode or Parallel MPI Codes

Quote from the following link.

"IMPORTANT: The optimal value of --nodes and --ntasks for a parallel code must be determined empirically by conducting a scaling analysis. As these quantities increase, the parallel efficiency tends to decrease. The parallel efficiency is the serial execution time divided by the product of the parallel execution time and the number of tasks. If multiple nodes are used then in most cases one should try to use all of the CPU-cores on each node."

```{note}
For the scaling analysis that was performed with CMAQ, the parallel efficiency was determined as the runtime for the smallest number of CPUs divided by the product of the parallel execution time and the number of additional cpus used. If smallest NPCOLxNPROW configuration was 18 cpus, the run time for that case was used, and then the parallel efficiency for the case where 36 cpus were used would be parallel efficiency = runtime_18cpu/(runtime_36cpu*2)*100
```

```{seealso}
<a href="https://researchcomputing.princeton.edu/support/knowledge-base/scaling-analysis">Scaling Analysis - see section on Multinode or Parallel MPI Codes</a>

<a href="https://researchcomputing.princeton.edu/support/knowledge-base/slurm#multinode">Example Slurm script for Multinode Runs</a>
```

## Slurm Compute Node Provisioning

AWS ParallelCluster relies on SLURM to make the job allocation and scaling decisions. The jobs are launched, terminated, and resources maintained according to the Slurm instructions in the CMAQ run script. The YAML file for Parallel Cluster is used to set the identity of the head node and the compute node, and the maximum number of compute nodes that can be submitted to the queue. The head node can't be updated after a cluster is created. The compute nodes, and the maximum number of compute nodes can be updated after a cluster is created. 

Number of compute nodes dispatched by the slurm scheduler is specified in the run script using #SBATCH --nodes=XX #SBATCH --ntasks-per-node=YY where the maximum value of tasks per node or YY limited by many CPUs are on the compute node.

Resources specified in the YAML file: 

* Ubuntu2004 
* Disable Simultaneous Multi-threading
* Spot Pricing 
* Shared EBS filesystem to install software

* 1.2 TiB Shared Lustre file system with imported S3 Bucket (1.2 TiB is the minimum file size that you can specify for Lustre File System) mounted as /fsx <b>or</b> EBS volume 500 GB size mounted as /shared/data

* Slurm Placement Group enabled
* Elastic Fabric Adapter Enabled

```{seealso}
<a href="https://aws.amazon.com/ec2/instance-types/c5/">EC2 Instance Types</a>
```

```{note}
Pricing information in the tables below are subject to change. The links from which this pricing data was collected are listed below.
```

```{seealso}
<a href="https://aws.amazon.com/ec2/spot/pricing/">EC2 SPOT Pricing</a>
```

```{seealso}
<a href="https://aws.amazon.com/ec2/pricing/on-demand">EC2 On-Demand Pricing</a>
```

```{seealso}
<a href="https://docs.aws.amazon.com/parallelcluster/latest/ug/spot.html">Working with Spot Instances - ParallelCluster</a>
```

<a href="https://aws.amazon.com/blogs/hpc/application-deep-dive-into-the-graviton3e-based-amazon-ec2-hpc7g-instance/">hpc7g offers 16, 32 or 64 physical cpu instance size at launch</a>

```{note}
Sometimes, the nodes are not available for SPOT pricing in the region you are using. 
If this is the case, the job will not start runnning in the queue, see AWS Troubleshooting. 
<a href="https://docs.aws.amazon.com/parallelcluster/latest/ug/troubleshooting.html">ParallelCluster Troubleshooting</a>
```


## Benchmark Timings for CMAQv5.4 12US1 Benchmark 

### Benchmark Timing for c6a.48xlarge

Table 2. Timing Results for CMAQv5.4 2 Day 12US1 Run on Parallel Cluster with c6a.xlarge head node and c6a.48xlarge Compute Nodes with Disable Simultaneous Multithreading turned on (using physical cores, not vcpus) 

| CPUs | NodesxCPU | COLROW | Day1 Timing (sec) | Day2 Timing (sec) | TotalTime | CPU Hours/day InputData   |    InputData | Equation using Spot Pricing | SpotCost | Equation using On Demand Pricing | OnDemandCost |
| ---- | ------    | ---   |  -------------     | ------------      | --------- | ------------------------  | ----------   | ------------------------------ | ----     | ------------------------------  |  ------  |
| 96   | 1x96 | 12x8    |           3153.2      |  3485.9          | 6639.10    | 1.844                     |    /fsx         |  $5.5809/hr * 1 node * 1.844 = | 10.29 | 7.34/hr * 1 node * 1.844 = | 13.53 |
| 192     | 2x96 | 16x12   |  1853.4     | 2035.1           |  3888.50  | 1.08      | /fsx            | $5.5809/hr * 2 node * 1.08 = | 12.05 | 7.34/hr * 2 node * 1.08 = | 15.85  |
| 288     | 3x96 | 16x18 | 1475.9      |  1580.7          | 3056.60   | .849        |  /fsx           |  5.5809/hr * 3 node * .849 = | 14.21  | 7.34/hr * 3 node * .849 = | 18.6 |

### Benchmark Timing for hpc6a.48xlarge

Table 2. Timing Results for CMAQv5.4 2 Day 12US1 Run on Parallel Cluster with c6a.xlarge head node and c6a.48xlarge Compute Nodes with Disable Simultaneous Multithreading turned on (using physical cores, not vcpus)

| CPUs | NodesxCPU | COLROW | Day1 Timing (sec) | Day2 Timing (sec) | TotalTime | CPU Hours/day InputData   |    InputData | Equation using Spot Pricing | SpotCost | Equation using On Demand Pricing | OnDemandCost |
| ---- | ------    | ---   |  -------------     | ------------      | --------- | ------------------------  | ----------   | ------------------------------ | ----     | ------------------------------  |  ------  |
| 96   | 1x96 | 12x8    |   3157.3         |  3493.4           | 6650.70    |1.84     |    /fsx     |  n/a | n/a | 2.88/hr * 1 node * 1.845 = | 5.32 |
| 192     | 2x96 | 16x12   |  1850.0    | 2058.0           |  3908.00  | 1.085      | /fsx            | n/a | n/a | 2.88/hr * 2 node * 1.085 = | 6.25  |
| 288     | 3x96 | 16x18 | 1491.5      |  1599.8          | 3091.30   | .859        |  /fsx           | n/a | n/a  | 2.88/hr * 3 node * .859 = | 7.41 |


### Benchmark Timing for hpc7g.8xlarge with 32 processors per node

Table 3. Timing Results for CMAQv5.4 2 Day 12US1 Run on Parallel Cluster with c7g.large head node and hpc7g.8xlarge Compute Nodes with 32 processors per node.

| CPUs | NodesxCPU | COLROW | Day1 Timing (sec) | Day2 Timing (sec) | TotalTime | CPU Hours/day |  InputData   |    Equation using On Demand Pricing | OnDemandCost |
| ---- | ------    | ----   | ------------     | -------------      | --------- | ------------  | ------------ | -------------------------------- |    -- |
| 32   | 1x32      | 4x8    |  6933.3    |  6830.2     | 13763.50   | 3.82      |  /fsx          |   1.6832/hr * 1 node * 3.82 = | 6.42 |
| 64   | 2x32      | 8x8   |  3080.9     |  3383.5     | 6464.40  | 1.795        | /fsx          | 1.6832/hr * 2 node * 1.795 = | 6.04  |
| 96   | 3x32      | 12x8   |  2144.2     |  2361.9     | 4506.10  | 1.252       | /fsx          | 1.6832/hr * 3 node * 1.252 = | 6.32  |
| 128  | 4x32      | 16x8   |  1696.6     |  1875.7     | 3572.30  | .992        | /fsx          | 1.6832/hr * 4 node * .992 = | 6.68  |


### Benchmark Timing for hpc7g.16xlarge with 64 processors per node

Table 4. Timing Results for CMAQv5.4 2 Day 12US1 Run on Parallel Cluster with c7g.large head node and hpc7g.16xlarge Compute Nodes with 64 processors per node.

| CPUs | NodesxCPU | COLROW | Day1 Timing (sec) | Day2 Timing (sec) | TotalTime | CPU Hours/day |  InputData   |    Equation using On Demand Pricing | OnDemandCost |
| ---- | ---       | ----   | -------------     | ------------    | --------- |  -----------    | ------------ | -------------------------------- | --- |
| 64   | 1x64      | 8x8    |  crash            |  crash          | crash     |  n/a            |    /fsx      | 1.6832/hr * 1 node * n/a = | n/a |
| 128       | 2x64 | 8x16   |  2074.2           | 2298.9          | 4373.10   | 1.215           |    /fsx      | 1.6832/hr * 2 node * 1.214 = | 4.089  |
| 192       | 3x64 | 12x16  | 1617.1            | 1755.3          | 3372.40   | .937            | /fsx/        | 1.6832/hr * 3 node * .937  = | 4.730  |
| 256       | 4x64 | 16x16  | 1347.3            | 1501.4          | 2848.70   | .7913           | /fsx/        | 1.6832/hr * 4 node * .7913  = | 5.327  |
| 320       | 5x64 | 16x20  | 1177.0            | 1266.6          | 2443.60   | .6788           | /fsx/        | 1.6832/hr * 5 node * .6788  = | 5.713  |



# Benchmark Scaling Plots for CMAQv5.4 12US1 Benchmark

### Benchmark Scaling Plot for hpc6a.48xlarge

Figure 1. Scaling per Node on hpc6a.48xlarge Compute Nodes (96 cores/node)

![Scaling per Node for hpc6a.48xlarge Compute Nodes (96cpu/node](../../qa_plots/scaling_plots/hpc6a.48xlarge_Scaling_Node.png)


```
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=96
```

### Benchmark Scaling Plot for hpc7g.8xlarge
 
Figure 2. Scaling per CPU on hpc7g.8xlarge compute node (32 cores/node)

![Scaling per CPU for hpc7g.8xlarge Compute Nodes (32cores/node](../../qa_plots/timing_plots/HPC7gpes_32_64.png)


### Benchmark Scaling Plot for hpc7g.16xlarge

Figure 3.  Scaling per Node on hpc7g.16xlarge Compute Nodes (64 cores/node)

![Scaling per Node for hpc7g.16xlarge Compute Nodes (32cores/node](../../qa_plots/timing_plots/HPC7g.18xlargepes_64_128.png)


## Total Time and Cost versus CPU Plot for hpc7g.8xlarge 

Figure 4. Plot of Total Time and On Demand Cost varies as additional CPUs are used. Note that the run script and yaml settings settings that were optimized for running CMAQ on the cluster.

![Total Time and Cost for hpc7g.16xlarge Compute Nodes (64cores/node](../../qa_plots/timing_plots/hpc7g.16xlarge_Time_Cores.png)


Figure 5. Plot of Total Time and On Demand Cost versus CPUs for c6a.48xlarge

![Total Time and Cost for c6a.48xlarge Compute Nodes (96cores/node](../../qa_plots/timing_plots/c6a.48xlarge_Time_Cores.png)

# Cost Information 

Cost information is available within the AWS Web Console for your account as you use resources, and there are also ways to forecast your costs using the pricing information available from AWS.

### Cost Explorer

Example screenshots of the AWS Cost Explorer Graphs were obtained after running several of the CMAQ Benchmarks, varying # nodes and # cpus and NPCOL/NPROW.  These costs are of a two day session of running CMAQ on the ParallelCluster, and should only be used to understand the relative cost of the EC2 instances (head node and compute nodes), compared to the storage, and network costs.

In Figure 6 The Cost Explorer Display shows the cost of different EC2 Instance Types: note that c6a.48xlarge (purple) is highest cost - as these the most expensive compute nodes that were used. The hpc7g.16xlarge compute nodes incurred less cost (green).

Figure 6. Cost by Instance Type - AWS Console 

![AWS Cost Management Console - Cost by Instance Type](../../qa_plots/cost_plots/AWS_12US1_Bench_instance_type.png)

In Figure 7 The Cost Explorer displays a graph of the cost categorized by usage by spot or OnDemand, NatGateway, or Timed Storage. Note: c6a.48xlarge is highest generating cost resource, but other resources such as storage on the EBS volume and the network NatGatway or SubnetIDs also incur costs

Figure 7. Cost by Usage Type - AWS Console 

![AWS Cost Management Console - Cost by Usage Type](../../qa_plots/cost_plots/AWS_12US1_Bench_usage.png)

In Figure 8. The Cost Explorer Display shows the cost by Services including EC2 Instances, S3 Buckets, and FSx Lustre File Systems

Figure 8. Cost by Service Type - AWS Console

![AWS Cost Management Console - Cost by Service Type](../../qa_plots/cost_plots/AWS_12US1_Bench_service_type.png)


### Compute Node Cost Estimate

Head node c7g.large compute cost = entire time that the parallel cluster is running ( creation to deletion) = 6 hours * $0.0725/hr = $ .435 using ondemand pricing.


Table 5. Extrapolated Cost of compute nodes used for CMAQv5.4+ Annual Simulation based on 2 day 12US1 benchmark

| Benchmark Case | Compute Node | Number of PES |  Number of Nodes | Pricing    |   Cost per node | Time to completion (hour)   | Equation Extrapolate Cost for Annual Simulation | Annual Cost                | Days to Complete Annual Simulation | 
| -------------  | --------     |------------  |  --------------- | -------    |  -------------- | ------------------          |  ------------------------------------------- | ----    |  -------------------------------    |
| 2 day 12US1    | c6a.48xlarge   | 96          |         1       |  ONDEMAND    |   $7.344/hour   |   6639.10/3600 = 1.84      |  1.84/2 * 365 = 336.6 hours/node * 1 node =   336.6  hr * 7.344/hr =   | $2471 | 14 |
| 2 day 12US1    | hpc6a.48xlarge   | 96          |         1       |  ONDEMAND    | $2.88/hour   |   6639.10/3600 = 1.84      |  1.84/2 * 365 = 336.6 hours/node * 1 node =   336.6  hr * 2.88/hr =   | $969 | 14 |
| 2 day 12US1    | hpc7g.16xlarge | 128          |         2       |  ONDEMAND    |   $1.6832/hour  |   4574.00/3600 = 1.27      |  1.27/2 * 365 = 231.87 hours/node * 2 nodes = 463.75 hr * $1.6832/hr = | $780 | 9.6 |
| 2 day 12US1    | hpc7g.16xlarge | 192          |         3       |  ONDEMAND    |   $1.6832/hour  |   3509.80/3600 = .9749      |  .9749/2 * 365 = 177.9 hours/node * 3 nodes = 533.75 hr * $1.6832/hr = | $898 | 7.4 |

```{note}
These cost estimates depend on the availability of number of nodes for the instance type. If fewer nodes are available, then it will take longer to complete the annual run, but the costs should be accurate, as the 12US1 Domain Benchmark scales well up to this number of nodes. 
The cost of running an annual simulation on 2 hpc7g.16xlarge nodes using OnDemand Pricing is $780, the cost of running an annual simulation on 3 hpc7g.16xlarge nodes using OnDemand pricing is $898. If you run on only 2 nodes, then you would pay less, but wait longer for the run to be completed, 9.6 days using 2 nodes versus 7.4 days using 3 nodes.
```

### Storage Cost Estimate

```{seealso}
<a href="https://aws.amazon.com/fsx/lustre/pricing/">AWS Lustre Pricing</a>
```


Table 6. Lustre SSD File System Pricing for us-east-1 region

| Storage Type | Storage options   | 	Pricing with data compression enabled*	| Pricing (monthly)  |
| --------     | ----------------  |   ------------------------------------    | -----------------  |
| Persistent   | 125 MB/s/TB       | 	$0.073                                  |	$0.145/month |
| Persistent   | 250 MB/s/TB       | 	$0.105                                  |	$0.210/month |
| Persistent   | 500 MB/s/TB       | 	$0.170                                  | 	$0.340/month |
| Persistent   | 1,000 MB/s/TB     |   $0.300                                  | 	$0.600/month | 
| Scratch      | 200/MB/s/TiB      |    $0.070 	                               |        $0.140/month |	

Note, there is a difference in the storage sizing units that were obtained from AWS. 

```{seealso}
<a href="https://www.techtarget.com/searchstorage/definition/tebibyte-TiB#:~:text=Tebibyte%20vs.&text=One%20tebibyte%20is%20equal%20to,when%20talking%20about%20storage%20capacity">TB vs TiB</a>
```

Quote from the above website;
"One tebibyte is equal to 2^40 or 1,099,511,627,776 bytes."
One terabyte is equal to 1012 or 1,000,000,000,000 bytes. 
A tebibyte equals nearly 1.1 TB. 
That's about a 10% difference between the size of a tebibyte and a terabyte, which is significant when talking about storage capacity."

Lustre Scratch SSD 200 MB/s/TiB is tier of the storage pricing that we have configured in the yaml for the cmaq parallel cluster.

```{seealso}
<a href="https://docs.aws.amazon.com/parallelcluster/latest/ug/SharedStorage-v3.html#SharedStorage-v3-FsxLustreSettings">YAML FSxLustreSettings</a>
```

Cost example:
    0.14 USD per month / 730 hours in a month = 0.00019178 USD per hour

Note: 1.2 TiB is the minimum file size that you can specify for the lustre file system

    1,200 GiB x 0.00019178 USD per hour x 24 hours x 5 days = 27.6 USD

Question is 1.2 TiB enough for the output of a yearly CMAQ run?

For the output data, assuming 2 day CONUS Run, all 35 layers, all 244 variables in CONC output

```
cd /fsx/data/output/output_CCTM_v532_gcc_2016_CONUS_16x8pe_full
du -sh
```

Size of output directory when CMAQ is run to output all 35 layers, all 244 variables in the CONC file, includes all other output files

```
173G .
```

So we need 86.5 GB per day

Storage requirement for an annual simulation if you assumed you would keep all data on lustre filesystem

     86.5 GB * 365 days = 31,572.5 GB  = 31.5 TB


### Annual simulation local storage cost estimate

Assuming it takes 5 days to complete the annual simulation, and after the annual simulation is completed, the data is moved to archive storage.

     31,572.5 GB x 0.00019178 USD per hour x 24 hours x 5 days = $726.5 USD


To reduce storage requirements; after the CMAQ run is completed for each month, the post-processing scripts are run and completed, and then the CMAQ Output data for that month is moved from the Lustre Filesystem to the Archived Storage. Monthly data volume storage requirements to store 1 month of data on the lustre file system is approximately 86.5 x 30 days = 2,595 GB or 2.6 TB.  

      2,595 GB x 0.00019178 USD per hour x 24 hours x 5 days = $60 USD


Estimate for S3 Bucket cost for storing an annual simulation

```{seealso}
<a href="https://aws.amazon.com/s3/pricing/?p=pm&c=s3&z=4">S3 Storage Pricing Tiers</a>
```

| S3 Standard - General purpose storage |    Storage Pricing  |
| ------------------------------------  |    --------------   |
| First 50 TB / Month                   |     $0.023 per GB   |
| Next 450 TB / Month                   |     $0.022 per GB   |
| Over 500 TB / Month                   |     $0.021 per GB   |


### Archive Storage cost estimate for annual simulation - assuming you want to save it for 1 year

31.5 TB * 1024 GB/TB * .023 per GB * 12 months  = $8,903

| S3 Glacier Flexible Retrieval (Formerly S3 Glacier) |    Storage Pricing |
| --------------------------------------------------  |    --------------  |
| long-term archives with retrieval option from 1 minute to 12 hours|      |	
| All Storage / Month| 	$0.0036 per GB   |

S3 Glacier Flexible Retrieval Costs 6.4 times less than the S3 Standard

31.5 TB * 1024 GB/TB * $.0036 per GB * 12 months  = $1393.0 USD

Lower cost option is S3 Glacier Deep Archive (accessed once or twice a year, and restored in 12 hours)

31.5 TB * 1024 GB/TB * $.00099 per GB * 12 months  = $383 USD

### CycleCloud and ParallelCluster Price Comparison of Cost Estimate for Annual Simulation (Filesystem + Compute) 

| Vendor | Cluster Name | Resource Type | Virtual Machine | Nodes | Cores | Minimum Storage Size (GB) | Storage Hourly Price | SPOT $/hr | OnDemand $/hr | CMAQv5.4 two-day runtime (sec) | CMAQv5.4 two-day runtime (hr) | Annual Cost Equation | Total Time (hr/node) | Annual Cost Spot | Annual Cost OnDemand | Storage Cost NFS | Storage Cost Lustre | Storage Cost Beeond | Days to Complete Annual Simulation | Total Cost for Annual Run (Lustre, Compute Node, Scheduler, NFS Storage) | Total Cost for Annual Run (Beeond, Compute Node, Scheduler, NFS Storage) | Cost Savings of using Beeond Filesystem |
| ----   | -----------  | ------------  | --------------  | ----- | ----  | -----------------------   | -------------------  | --------  | ------------  | ----------------------------   | ----------------------------  | -------------------  | -------------------  | ---------------  | -------------------  | ---------------  | ------------------  | ------------------  |  ------------------------------    |  ------------------------------- | ------------ | ----- |
| Azure  |  CycleCloud  | Compute       | HB120_v3        | 3     | 288   |                          |                       | $0.36     | $3.60         | 3255.3                         | 0.90425                       | .904/2 * 365 = | 165.025 | $178.23    | $1782               |            |                           |                     |   **6.9**                      | **$2,462**                                                   | **$1847** |  $615 |
| Azure  |  CycleCloud  | Login         | Standard_D8as_v4 | 1    | 8     |                          |                     | N/A         | $0.0048       | 6510.6                                  | 1.8               | 1.8/2*365 =    | 330.05125           | N/A           | $2                |   |  |  |  |  | |
| Azure  |  CycleCloud  | Scheduler     |  Standard_D4s_v3 | 1    |  4    |                          |                     | N/A         | $0.19         | 6510.6                                  | 1.8               | 1.8/2*365 =   | 330.05125           | N/A           | $63               |   |  |  |  |  | |
| Azure  |  CycleCloud  | NFS Storage   | Premium SSD LRS  |      |       |  1                       | $0.0001100          |             |               |                                         |                      |                             |                     |               |                  |  $0.0363056     |                 | | | | |
| Azure  |  CycleCloud  | Lustre Storage| Ultra tier (500 MB/s/TiB) | | | 4000                       | $0.000466           |             |               |                                         |                      |                             |                     |               |                  |                 | $307.607765 | | | | | |
| Azure | CycleCloud    | Beeond        | 2 * 960 GB NVMe (block) |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  $0 | | | |  |
|       |               |               |                         |  |  |  |  |  |  |  |  |  |  |  |  |  |  |     | | | |  |
| AWS   | ParallelCluster | Compute     | hpc7g.16xlarge  | 3     | 192   |                          |                     | N/A         | $1.68         | 3509.8                          | 0.98                 | .98/2 * 365 | 177.9 | N/A | $898 |     |   |   | **7.4** | **$1006** | |  $81.9 |
| AWS   |  ParallelCluster | Scheduler  | c7g.large | 1  | 2  |   |   |  N/A  | $0.07 | 7019.6 | 1.95 | 1.95/2*365= | 355.8 | N/A  | $25.73  |  |  |  |  |  |  | |
| AWS   |   ParallelCluster | Shared Storage | EBS: GP3 |  |  |  1  | $0.00010959 |  |  |  |  |  |  |  |  |  $0.03899812 | |  |  |  |  | |
| AWS   |  ParallelCluster  | Lustre  | Scratch SSD 200 MB/s/TiB |  |  |  1200  | $0.00019178 |  |  |  |  |  |  |  |  |  |  $40.94749118 | |  |  |  |  |


### Assumptions for Price Estimate for Annual Simulation (Filesystem + Compute)

* Assuming that you have an anual simulation turn-around time requirement of < 8 days (less than 3787 seconds for 2 day benchmark)
* Assuming you have the scheduler and filesystems available for 2 * the length of the compute node time for building CMAQ, installing input data, and copying output data to S3 bucket.
* Note, SPOT pricing is not available for AWS hpc7g.16xlarge
* Note, SPOT pricing is not recomended for the scheduler node
* Note, instructions for how to use SPOT pricing on Azure are not yet available
* Note, have not replicated using the Beeond Filesystem on AWS
* Note, assuming Lustre Filesystem is used at least as long as the scheduler node
* Note, Lustre Filesystem is created before Azure CycleCloud, and would need manual deletion after the run, recommend using Beeond Filesystem due to level of difficulty of provisioning Lustre Filesystem on CycleCloud
* Assuming that you have the scheduler node running 2x longer than the compute nodes

Timings for the CycleCloud Cluster are available on the tutorial: <a href="https://cyclecloud-cmaq.readthedocs.io/en/latest/user_guide_cyclecloud/Performance-Opt/performance-optimization.html#slurm-compute-node-provisioning">CycleCloud 12US1 Benchmark Results</a>



# Recommended Workflow for extending to annual run

Post-process monthly save output and/or post-processed outputs to S3 Bucket at the end of each month.

Still need to determine size of post-processed output (combine output, etc).

      86.5 GB * 31 days = 2,681.5 GB * 1 TB/1024 GB =  2.62 TB

Cost for lustre storage of a monthly simulation

      2,681.5 GB x 0.00019178 USD per hour x 24 hours x 5 days = $61.7 USD

Goal is to develop a reproducable workflow that does the post processing after every month, and then copies what is required to the S3 Bucket, so that only 1 month of output is imported at a time to the lustre scratch file system from the S3 bucket.
This workflow will help with preserving the data in case the cluster or scratch file system gets pre-empted.

