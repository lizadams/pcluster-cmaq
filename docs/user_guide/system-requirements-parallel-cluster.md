## System Requirements for Parallel Cluster

### Please set up a alarm on AWS 

Configure alarm to receive an email alert if you exceed $100 per month (or what ever monthly spending limit you need).
It may be possible to set up daily or weekly spending alarms as well.

### Software Requirements

* Git
* Compilers (C, C++, and Fortran) - GNU compilers version ≥ 8.3
* MPI (Message Passing Interface) -  OpenMPI ≥ 4.0
* NetCDF (with C, C++, and Fortran support)
* I/O API
* Slurm Scheduler
* AWS CLI v3.0 installed in a virtual environment
* pcluster is the primary AWS ParallelCluster CLI command. You use pcluster to launch and manage HPC clusters in the AWS Cloud and to create and manage custom AMI images
* Edit YAML Configuration Files using vi, nedit or other editor (yaml doesn't accept tabs as spacing)

### AWS CLI v3.0 is available in the following AWS Regions
Note, the scripts in this tutorial use the us-east-1 region, but the scripts can be modified to use any of the supported regions listed in the url below.

<a href="https://docs.aws.amazon.com/parallelcluster/latest/ug/supported-regions-v3.html">CLI v3 Supported Regions</a>



### Hardware Requirements

#### Recommended Minimum Requirements

The size of hardware depends on the domain size and resolution for  your CMAQ case, and how quickly your turn-around requirements are.
Larger hardware and memory configurations are also required for instrumented versions of CMAQ incuding CMAQ-ISAM and CMAQ-DDM3D.
The Parallel Cluster allows you to run the compute nodes only as long as the job requires, and you can also update the compute nodes as needed for your domain

#### Recommended Parallel Cluster Configuration for CONUS Domain

Head node:

* c5n.large

Compute Node:

* c5n.18xlarge 
with 192 GiB memory, 14 Gbps EBS Bandwidth, and 100 Gbps Network Bandwidth

Figure 1. AWS Recommended Parallel Cluster Configuration (Number of compute nodes depends on setting for NPCOLxNPROW and #SBATCH --nodes=XX #SBATCH --ntasks-per-node=YY )

![AWS Minimum Viable Product Configuration](../diagrams/aws_minimum_viable_product.png)

#### AWS ParallelCluster doesn't make job allocation or scaling decisions. It simple tries to launch, terminate, and maintain resources according to Slurm’s instructions.

Number of compute nodes dispatched by the slurm scheduler is specified in the run script using #SBATCH --nodes=XX #SBATCH --ntasks-per-node=YY where the maximum value of tasks per node or YY limited by many CPUs are on the compute node.  

For c5n.18xlarge, there are 36 CPUs, so maximum value of YY is 36 or --ntask-per-node=36.  

If running a job with 180 processors, this would require the --nodes=XX or XX to be set to 5 compute nodes, as 36x5=180.  

The setting for NPCOLxNPROW must also be a maximum of 180, ie. 18 x 10 or 10 x 18 to use all of the CPUs in the parallel cluster.


<a href="https://aws.amazon.com/blogs/aws/new-c5n-instances-with-100-gbps-networking/">C5n Instance </a>
Each vCPU is a hardware hyperthread on the Intel Xeon Platinum 8000 series processor. You get full control over the C-states on the two largest sizes, allowing you to run a single core at up to 3.5 Ghz using Intel Turbo Boost Technology.

The new instances also feature a higher amount of memory per core, putting them in the current “sweet spot” for HPC applications that work most efficiently when there’s at least 4 GiB of memory for each core. The instances also benefit from some internal improvements that boost memory access speed by up to 19% in comparison to the C5 and C5d instances.

The C5n instances incorporate the fourth generation of our custom Nitro hardware, allowing the high-end instances to provide up to 100 Gbps of network throughput, along with a higher ceiling on packets per second. The Elastic Network Interface (ENI) on the C5n uses up to 32 queues (in comparison to 8 on the C5 and C5d), allowing the packet processing workload to be better distributed across all available vCPUs. 


Software: 

* Ubuntu2004 
* Disable Simultaneous Multi-threading
* Spot Pricing 
* Shared EBS filesystem to insall software
* 1.2 TB Shared Lustre file system with imported S3 Bucket (1.2 TB is the minimum file size that you can specify for Lustre File System)
* Slurm Placement Group enabled
* Elastic Fabric Adapter Enabled on c5n.18xlarge


<a href="https://aws.amazon.com/blogs/aws/new-c5n-instances-with-100-gbps-networking/">AWS c5n Pricing</a>

<a href="https://aws.amazon.com/ec2/spot/pricing/">EC2 SPOT Pricing</a>

<a href="https://aws.amazon.com/ec2/pricing/on-demand">EC2 On-Demand Pricing</a>

<a href="https://docs.aws.amazon.com/parallelcluster/latest/ug/spot.html">Working with Spot Instances - Parallel CLuster</a>


Table 1. EC2 Instance On-Demand versus Spot Pricing (price is subject to change)

| Instance Name	| vCPUs |  RAM      |  EBS Bandwidth	| Network Bandwidth | Linux On-Demand Price | Linux Spot Price | 
| ------------  | ----- | --------  | ---------------   | ---------------   | --------------------  | ---------------  |
| c4.large	| 2	| 3.75 GiB  |   Moderate	|  500 Mbps         | 	$0.116/hour         | $0.0312/hour     |
| c4.8xlarge	| 36	| 60 GiB    |	10 Gbps	        |  4,000 Mbps       | 	$1.856/hour         | $0.5903/hour     |
| c5n.large	| 2	| 5.25 GiB  |	Up to 3.5 Gbps	| Up to 25 Gbps     |   $0.108/hour         | $0.0324/hour     |
| c5n.xlarge	| 4	| 10.5 GiB  |	Up to 3.5 Gbps	| Up to 25 Gbps     |   $0.216/hour         | $0.0648/hour     |
| c5n.2xlarge	| 8	| 21 GiB    |	Up to 3.5 Gbps	| Up to 25 Gbps     |   $0.432/hour         | $0.1740/hour     |
| c5n.4xlarge	| 16	| 42 GiB    | 	3.5 Gbps	| Up to 25 Gbps     |   $0.864/hour         | $0.2860/hour     |
| c5n.9xlarge	| 36	| 96 GiB    |	7 Gbps	        | 50 Gbps           |   $1.944/hour         | $0.5971/hour     |
| c5n.18xlarge	| 72	| 192 GiB   |	14 Gbps	        | 100 Gbps          |   $3.888/hour         | $1.1732/hour     |

Using c5n.18xlarge as the compute node, it costs 3.888/hr/1.1732/hr = 3.314 times as much to run on demand versus spot pricing

Table 2. Timing Results for CMAQv5.3.3 2 Day CONUS2 Run on Parallel Cluster with c5n.large head node and C5n.18xlarge Compute Nodes

| Number of PEs | #Nodesx#CPU | NPCOLxNPROW | Day1 Timing (sec) | Day2 Timing (sec) | Total Time(2days)(sec) | SBATCH --exclusive | Data Imported or Copied | DisableSimultaneousMultithreading(yaml)| Answers Matched | Cost using Spot Pricing | Cost using On Demand Pricing | 
| ------------- | -----------    | -----------   | ----------------     | ---------------      | -------------------        | ------------------ | --------------          | ---------                              |   -------- | --------- | ------ |
| 180           |  5x36          | 10x18         | 2481.55              | 2225.34              |    4706.89                 |  no                | copied                  |  false                                 |            | 1.1732/hr * 5 nodes * 1.307 hr = $7.66 | 3.888/hr * 5 nodes * 1.307 hr = $25.4 |
| 180           |  5x36          | 10x18         | 2378.73              | 2378.73              |    4588.92                 |  no                | copied                  |  true                     | 10x18 did not match 16x18 | 1.1732/hr * 5 nodes * 1.2747 = $7.477 | $ 24.77 |
| 180           |  5x36          | 10x18         | 1585.67        | 1394.52         |    2980.19           |  yes                | imported    |  true        |            | 1.1732/hr * 5nodes * 2980.9 / 3600 = $4.85 | $16.05 | 
| 256           |  8x36          | 16x16         |  1289.59       | 1164.53         |    2454.12           |  no                 |  copied           |  true    |            | 1.1732/hr * 8nodes * 2454.12 / 3600 = $6.398  | $21.66 |
| 256           |  8x36          | 16x16         |  1305.99       | 1165.30         |    2471.29           |  no                |   copied    |   true    |            | 1.1732/hr * 8nodes * 2471.29 / 3600 = $6.44 | $21.11 |
| 256           |  8x36          | 16x16         |  1564.90       | 1381.80         |    2946.70           |  no                |   imported  | true   |            | 1.1732/hr * 8nodes * 2946.7 / 3600 = $7.68 | $25.55 |
| 288           |  8x36          | 16x18         | 1873.00        | 1699.24         |     3572.2           |  no                |  copied     |    false       |            | $9.3 | $30.83 |
| 288           |  8x36          |  16x18        |  1976.35       | 1871.61         |     3847.96          |  no                |  Copied     |  true         |            | $10.02 | $33.22 |
| 288           |  8x36          | 16x18         |  1197.19       | 1090.45         |     2287.64          |  yes               |  Copied     |  true         |             16x18 matched 16x16 | $5.95 | $19.72
| 288           |  8x36          | 18x16         | 1206.01        | 1095.76         |     2301.77          |  yes               |  imported   |  true        |             | $5.98 | $19.83 |

Total c5n.18xlarge compute cost of Running Benchmarking Suite using SPOT pricing = $71.7

Figure 2. Cost by Instance Type - AWS Console

![AWS Cost Management Console - Cost by Instance Type](../qa_plots/cost_plots/AWS_Bench_Cost.png)


Figure 3. Cost by Usage Type - AWS Console

![AWS Cost Management Console - Cost by Usage Type](../qa_plots/cost_plots/AWS_Bench_Usage_Type_Cost.png)

Figure 4. Cost by Service Type - AWS Console

![AWS Cost Management Console - Cost by Service Type](../qa_plots/cost_plots/AWS_Bench_Service_Type_Cost.png)

Head node c5n.large compute cost = entire time that the parallel cluster is running ( creation to deletion) = 6 hours * $0.0324/hr = $ .1944 using spot pricing, 6 hours * $.108/hr = $.648 using on demand pricing.

Total c5n.18xlarge cost of Running Benchmarking Suite using ONDEMAND pricing = $238.9


Using 288 cpus on the Parallel Cluster, it would take ~4.832 days to run a full year, using 8 c5n.18xlarge compute nodes.

Table 3. Extrapolated Cost of c5n.18xlarge used for CMAQv5.3.3 Annual Simulation based on 2 day CONUS benchmark

| Benchmark Case | Number of PES |  Number of c5n.18xlarge Nodes | Pricing    |   Cost per node | Time to completion (hour)   | Extrapolate Cost for Annual Simulation                 |  
| -------------  | ------------  |  --------------- | -------    |  -------------- | ------------------          |  --------------------------------------------------    |
| 2 day CONUS    |  288          |          8       |    SPOT    |    1.1732/hour |     2287.64/3600 = .635455  |    .635455/2 * 365 = 115.97 hours/node * 8 nodes = 927.7 * $1.1732 = $1088.4 |
| 2 day CONUS    |  288          |          8       |  ONDEMAND  |    3.888/hour   |     2287.64/3600 = .635455  |    .635455/2 * 365 = 115.97 hours/node * 8 nodes = 927.7 * $3.888 = $3606.9 |



<a href="https://aws.amazon.com/fsx/lustre/pricing/">AWS Lustre Pricing</a>


Table 4. Lustre SSD File System Pricing for us-east-1 region

| Storage Type | Storage options   | 	Pricing with data compression enabled*	| Pricing (monthly)  |  Pricing (hourly) |
| --------     | ----------------  |   ------------------------------------    | -----------------  |  ---------------  |
| Persistant   | 125 MB/s/TB       | 	$0.073                                  |	$0.145/month |                   |
| Persistant   | 250 MB/s/TB       | 	$0.105                                  |	$0.210/month |                   |
| Persistant   | 500 MB/s/TB       | 	$0.170                                  | 	$0.340/month |                   |
| Persistant   | 1,000 MB/s/TB     |   $0.300                                  | 	$0.600/month | .0008333/hour     | 
| Scratch      | 200/MB/s/TiB      |    $0.070 	                               |        $0.140/month | 0.000192/hour     |	

Q. What is the difference between TiB and TB (I obtained the syntax from the AWS Pricing Table see link above)

Scratch SSD 200 MB/s/TB is tier of the storage pricing that we have configured in the yaml for the cmaq parallel cluster.

<a href="https://docs.aws.amazon.com/parallelcluster/latest/ug/SharedStorage-v3.html#SharedStorage-v3-FsxLustreSettings">FSxLustreSettings</a>

Cost example:
    0.14 USD per month / 730 hours in a month = 0.00019178 USD per hour

Note: 1.2 TB is the minimum file size that you can specify for the lustre file system

    1,200 GB x 0.00019178 USD per hour x 24 hours x 5 days = 27.6 USD

Question is 1.2 TB enough for the output of a yearly CMAQ run?

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


Cost for annual simulation

     31,572.5 GB x 0.00019178 USD per hour x 24 hours x 5 days = $726.5 USD


Table 5. Extrapolated Cost of Lustre File system for CMAQv5.3.3 Annual Simulation based on 2 day CONUS benchmark

Need to create table


Also need estimate for S3 Bucket cost for storing an annual simulation


### Recommended Workflow

Post-process monthly save output and/or post-processed outputs to S3 Bucket at the end of each month.

Still need to determine size of post-processed output (combine output, etc).

      86.5 GB * 31 days = 2,681.5 GB  =  2.6815 TB

Cost for lustre storage of a monthly simulation

      2,681.5 GB x 0.00019178 USD per hour x 24 hours x 5 days = $61.7 USD

Goal is to develop a reproducable workflow that does the post processing after every month, and then copies what is required to the S3 Bucket, so that only 1 month of output is stored at a time on the lustre scratch file system.
This workflow will help with preserving the data in case the cluster or scratch file system gets pre-empted.
