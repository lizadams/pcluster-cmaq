## System Requirements

### Software Requirements

* Git
* Compilers (C, C++, and Fortran)
   * GNU compilers version ≥ 8.3
* MPI (Message Passing Interface)
   * OpenMPI ≥ 3.0
* NetCDF (with C, C++, and Fortran support)
* I/O API



### Hardware Requirements

#### Recommended Minimum Requirements

The size of hardware depends on the domain size and resolution for  your CMAQ case, and how quickly your turn-around requirements are.
Larger hardware and memory configurations are also required for instrumented versions of CMAQ incuding CMAQ-ISAM and CMAQ-DDM3D.


* c4.large instance running RHEL with 3.75 GiB memory, 500 Mbps Network Bandwidth and 2 virtual cpus is used for the CMAQ Training Case 

<a href="https://aws.amazon.com/blogs/aws/now-available-new-c4-instances/">AWS c4 Instance Pricing</a>

The 12km case study has a ColxRowxLayer = 67x59x35 and takes 2 hours to complete.
Note, that we run the tutorial instances for 4 days, 24 hours a day, and much of the time the node is idle (evenings) but we pay for the instances to be available 24/7 so that we do not shut down the instances, and then need to provide new IP addresses to login to.

The Parallel Cluster allows you to run the compute nodes only as long as the job requires, and you can also update the compute nodes as needed for your domain


#### Recommended Parallel Cluster Requirement for CONUS Domain

* c5n.18xlarge running Ubuntu with 192 GiB memory, 14 Gbps EBS Bandwidth, and 100 Gbps Network Bandwitdth

<a href="https://aws.amazon.com/blogs/aws/new-c5n-instances-with-100-gbps-networking/">AWS c5n Pricing</a>

| Instance Name	| vCPUs |  RAM      |  EBS Bandwidth	| Network Bandwidth | Linux On-Demand Price | Linux Spot Price | 
| ------------  | ----- | --------  | ---------------   | ---------------   | --------------------  | ---------------  |
| c4.large	| 2	| 3.75 GiB  |   Moderate	|  500 Mbps         | 	$0.116/hour         | $0.0191/hour     |
| c4.8xlarge	| 36	| 60 GiB    |	10 Gbps	        |  4,000 Mbps       | 	$1.856/hour         | $0.3190/hour     |
| c5n.large	| 2	| 5.25 GiB  |	Up to 3.5 Gbps	| Up to 25 Gbps     |   $0.108/hour         | $0.0190/hour     |
| c5n.xlarge	| 4	| 10.5 GiB  |	Up to 3.5 Gbps	| Up to 25 Gbps     |   $0.216/hour         | $0.0380/hour     |
| c5n.2xlarge	| 8	| 21 GiB    |	Up to 3.5 Gbps	| Up to 25 Gbps     |   $0.432/hour         | $0.0830/hour     |
| c5n.4xlarge	| 16	| 42 GiB    | 	3.5 Gbps	| Up to 25 Gbps     |   $0.864/hour         | $0.1667/hour     |
| c5n.9xlarge	| 36	| 96 GiB    |	7 Gbps	        | 50 Gbps           |   $1.944/hour         | $0.4494/hour     |
| c5n.18xlarge	| 72	| 192 GiB   |	14 Gbps	        | 100 Gbps          |   $3.888/hour         | $0.6997/hour     |


#### Performance of C5n.18xlarge Parallel Cluster for CONUS2 Domain

| Number of PEs | Number of Nodes| NPCOL x NPROW | 1st day Timing (sec) | 2nd day Timing (sec) | Total Time(2days) (sec)    | SBATCH --exclusive | Data Imported or Copied | DisableSimultaneousMultithreading(yaml)| Answers Matched |
| ------------- | -----------    | -----------   | ----------------     | ---------------      | -------------------        | ------------------ | --------------          | ---------                              |   -------- |
| 180           |  5x36          | 10x18         | 2481.55              | 2225.34              |    4706.89                 |  no                | copied                  |  false                                 |            |
| 180           |  5x36          | 10x18         | 2378.73              | 2378.73              |    4588.92                 |  no                | copied                  |  true                     | 10x18 did not match 16x18 |
| 180           |  5x36          | 10x18         | 1585.67        | 1394.52         |    2980.19           |  yes                | imported    |  true        |            |
| 256           |  8x36          | 16x16         |  1289.59       | 1164.53         |    2454.12           |  no                 |  copied           |  true    |            |
| 256           |  8x36          | 16x16         |  1305.99       | 1165.30         |    2471.29           |  no                |   copied    |   true    |            |
| 256           |  8x36          | 16x16         |  1564.90       | 1381.80         |    2946.70           |  no                |   imported  | true   |            |
| 288           |  8x36          | 16x18         | 1873.00        | 1699.24         |     3572.2           |  no                |  copied     |    false       |            |
| 288           |  8x36          |  16x18        |  1976.35       | 1871.61         |     3847.96          |  no                |  Copied     |  true         |            |
| 288           |  8x36          | 16x18         |  1197.19       | 1090.45         |     2287.64          |  yes               |  Copied     |  true         |             16x18 matched 16x16 |
| 288           |  8x36          | 18x16         | 1206.01        | 1095.76         |     2301.77          |  yes               |  imported   |  true        |             |


Using 288 cpus on the Parallel Cluster, it would take ~4.832 days to run a full year, as spot pricing cost of $81.00 for ec2 pricing.

<a href="https://www.amazonaws.cn/en/fsx/lustre/pricing/">AWS Lustre Pricing</a>

Persistant file system pricing:
| Storage options   | 	Pricing with data compression enabled*	| Pricing (monthly)  | 
| ----------------  |   ------------------------------------    | -----------------  |
| 125 MB/s/TB       | 	$0.073                                  |	$0.145/month |
| 250 MB/s/TB       | 	$0.105                                  |	$0.210/month |
| 500 MB/s/TB       | 	$0.170                                  | 	$0.340/month |  
| 1,000 MB/s/TB     |   $0.300                                  | 	$0.600/month | .0008333/hour  |   ??? is this the storage pricing that is used on parallel cluster?


Cost example:
    0.600 USD per month / 730 hours in a month = 0.00046658 USD per hour
    1,200 GB x 0.00046658 USD per hour x 24 hours x 5 days = 67.18 USD

Question is 1.2 TB enough for the output of a yearly CMAQ run?

#### For the output data, assuming 2 day CONUS Run, all 35 layers, all 244 variables in CONC output

cd /fsx/data/output/output_CCTM_v532_gcc_2016_CONUS_16x8pe_full
du -sh

173G .

So we need 86.5 GB per day

#### Storage requirement for an annual simulation if you assumed you would keep all data on lustre filesystem

     86.5 GB * 365 days = 31,572.5 GB  = 31.5 TB


Cost for annual simulation

     31,572.5 GB x 0.00046658 USD per hour x 24 hours x 5 days = 1,767.7 USD

### Assume that you do post-processing every month, and only save the combine output to S3 Bucket at the end of each month

      86.5 GB * 31 days = 2,681.5 GB  =  2.6815 TB

Cost for lustre storage of a monthly simulation

      2,681.5 GB x 0.00046658 USD per hour x 24 hours x 5 days = $150 USD

Need to develop a reproducable workflow that does the post processing after every month, and then copies what is required to the S3 Bucket
