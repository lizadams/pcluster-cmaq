## Run CMAQv5.4 on c6a.2xlarge

Obtain IP address 

Obtain IP address from AWS Web Console or use the following AWS CLI command to obtain the public IP address of the machine.

`aws ec2 describe-instances --region=us-east-1 --filters "Name=image-id,Values=ami-051ba52c157e4070c" | grep PublicIpAddress`

Login to the ec2 instance

`ssh -v -Y -i ~/downloads/your-pem.pem ubuntu@ip.address`


Login to the ec2 instance again, so that you have two windows logged into the machine.

`ssh -Y -i ~/downloads/your-pem.pem ubuntu@your-ip-address` 


Load the environment modules

`module avail`

`module load ioapi-3.2/gcc-11.3.0-netcdf  mpi/openmpi-4.1.2  netcdf-4.8.1/gcc-11.3 `

Update the pcluster-cmaq repo using git (optional)

`cd /shared/pcluster-cmaq`

`git pull`

Verify that the input data

Input Data for the benchmark

`ls -lrt /shared/data/12US1_LISTOS/*`


Run CMAQv5.4 for 12US1 Listos Training 3 Day benchmark Case on 4 pe

Input data is available for a subdomain of the 12km 12US1 case.

```
GRIDDESC

'2018_12Listos'
'LamCon_40N_97W'   1812000.000    240000.000     12000.000     12000.000   25   25    1
```

Use command line to submit the job. 

This single virtual machine does not have a job scheduler such as slurm installed.


```
cd /shared/build/openmpi_gcc/CMAQ_v54+/CCTM/scripts
./run_cctm_2018_12US1_listos.csh | & tee ./run_cctm_2018_12US1_listos.c6a.2xlarge.log
```

Use HTOP to view performance.

`htop`

output


If the ec2 instance was created without specifying 1 thread per core in the Advanced Settings, then it will have 8 vcpus.

![Screenshot of HTOP with hyperthreading on](htop_c6a.2xlarge_8vcpus_hyperthreading_on_by_default.png)

If the ec2 instance is configured to use 1 thread per core in the advanced setting, then it will have 4 cores

![Screenshot of HTOP with hyperthreading off](htop_c6a.2xlarge_hyperthreading_off.png)

Successful output using the gp3 volume with hyperthreading on (8vcpus)

After the benchmark is complete, use the following command to view the timing results.

`tail -n 20 run_cctm_2018_12US1_listos.c6a.2xlarge.log`

```
==================================
  ***** CMAQ TIMING REPORT *****
==================================
Start Day: 2018-08-05
End Day:   2018-08-07
Number of Simulation Days: 3
Domain Name:               2018_12Listos
Number of Grid Cells:      21875  (ROW x COL x LAY)
Number of Layers:          35
Number of Processes:       4
   All times are in seconds.

Num  Day        Wall Time
01   2018-08-05   227.7
02   2018-08-06   213.0
03   2018-08-07   216.2
     Total Time = 656.90
      Avg. Time = 218.96
```

Use lscpu to view number of cores

Cconfirm that there are 4 cores on the c6a.2xlarge ec2 instance that was created with hyperthreading turned off (1 thread per core).

`lscpu`

Output:

```
lscpu
Architecture:            x86_64
  CPU op-mode(s):        32-bit, 64-bit
  Address sizes:         48 bits physical, 48 bits virtual
  Byte Order:            Little Endian
CPU(s):                  4
  On-line CPU(s) list:   0-3
Vendor ID:               AuthenticAMD
  Model name:            AMD EPYC 7R13 Processor
    CPU family:          25
    Model:               1
    Thread(s) per core:  1
    Core(s) per socket:  4
    Socket(s):           1
    Stepping:            1
    BogoMIPS:            5299.98
    Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx mmxext fxsr_opt pdpe1gb rdt
                         scp lm constant_tsc rep_good nopl nonstop_tsc cpuid extd_apicid aperfmperf tsc_known_freq pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x
                         2apic movbe popcnt aes xsave avx f16c rdrand hypervisor lahf_lm cmp_legacy cr8_legacy abm sse4a misalignsse 3dnowprefetch topoext invpcid_s
                         ingle ssbd ibrs ibpb stibp vmmcall fsgsbase bmi1 avx2 smep bmi2 invpcid rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 clze
                         ro xsaveerptr rdpru wbnoinvd arat npt nrip_save vaes vpclmulqdq rdpid
Virtualization features: 
  Hypervisor vendor:     KVM
  Virtualization type:   full
Caches (sum of all):     
  L1d:                   128 KiB (4 instances)
  L1i:                   128 KiB (4 instances)
  L2:                    2 MiB (4 instances)
  L3:                    16 MiB (1 instance)
NUMA:                    
  NUMA node(s):          1
  NUMA node0 CPU(s):     0-3
Vulnerabilities:         
  Itlb multihit:         Not affected
  L1tf:                  Not affected
  Mds:                   Not affected
  Meltdown:              Not affected
  Mmio stale data:       Not affected
  Retbleed:              Not affected
  Spec store bypass:     Mitigation; Speculative Store Bypass disabled via prctl
  Spectre v1:            Mitigation; usercopy/swapgs barriers and __user pointer sanitization
  Spectre v2:            Mitigation; Retpolines, IBPB conditional, IBRS_FW, RSB filling, PBRSB-eIBRS Not affected
  Srbds:                 Not affected
  Tsx async abort:       Not affected

```

Once you have successfully run the benchmark, terminate the instance.

Terminate the c6a.2xlarge either thru the Web Console or using the CLI

Find the InstanceID using the following command on your local machine.

`aws ec2 describe-instances --region=us-east-1 | grep InstanceId`

Output

i-xxxx

Terminate Instance

`aws ec2 terminate-instances --region=us-east-1 --instance-ids i-xxxx`

Verify that the instance is being shut down.

`aws ec2 describe-instances --region=us-east-1`
