CMAQv5.4 on Parallel Cluster Advanced Tutorial (optional)

## Configure Parallel Cluster

Use ParallelCluster with default Ubuntu OS using hpc7g.large head node and hpc7g.16xlarge compute node.

Step by step instructions to configuring and running a ParallelCluster for the CMAQ 12US1 benchmark with instructions to install the libraries and software.

```{admonition} Notice
:class: warning

Skip this tutorial if you successfully completed the CMAQv5.4 on Parallel Cluster Intermediate Tutorial.
Unless you need to build the CMAQ libraries and code and run on a different family of compute nodes, such as the c6gn.16xlarge compute nodes AMD Graviton.

```


Activate the virtual environment to use the ParallelCluster command line

```
source ~/apc-ve/bin/activate
source ~/.nvm/nvm.sh
```

Upgrade to get the latest version of ParallelCluster


`python3 -m pip install --upgrade "aws-parallelcluster"`

Verify that the ParallelCluster AWS CLI is installed by checking the version


`pcluster version`

Output:

```
		{
  "version": "3.6.0"
}
```


Use an existing yaml file from the git repo to create a ParallelCluster

`cd /your/local/machine/install/path/`

Use a configuration file from the github repo that was cloned to your local machine

`git clone -b main https://github.com/CMASCenter/pcluster-cmaq.git pcluster-cmaq`


`cd pcluster-cmaq/yaml`

Edit the  hpc7g.16xlarge.ebs_unencrypted_installed_public_ubuntu2004.fsx_import.yaml

`vi hpc7g.16xlarge.ebs_unencrypted_installed_public_ubuntu2004.fsx_import.yaml`

```{note}
1. the hpc7g-16xlarge*.yaml is configured to use ONDEMAND instance pricing for the compute nodes.
2. the hpc7g-16xlarge*.yaml is configured to the the hpc7g.16xlarge as the compute node, with up to 10 compute nodes, specified by MaxCount: 10.
3. the hpc7g-16xlarge*.yaml is configured to disable multithreading (This option restricts the computing to CPUS rather than allowing the use of all virtual CPUS. (192 virtual cpus reduced to 96 cpus)
4. the hpc7g-16xlarge*.yaml is configured to enable the setting of a placement group to allow low inter-node latency
5. the hpc7g-16xlarge*.yaml is configured to enables the elastic fabric adapter
6. given this yaml configuration, the maximum number of PEs that could be used to run CMAQ is 64 cpus x 10 = 640, the max settings for NPCOL, NPROW is NPCOL = 32, NPROW = 20 or NPCOL=20, NPROW=32 in the CMAQ run script. Note: CMAQ does not scale well beyond 2-3 compute nodes.
```

Replace the key pair and subnet ID in the hpc7g.16xlarge*.yaml file with the values created when you configured the demo cluster

```
Region: us-east-1
Image:
  Os: ubuntu2004
HeadNode:
  InstanceType: c7g.large
  Networking:
    SubnetId: subnet-xx-xx-xx           << replace
  DisableSimultaneousMultithreading: true
  Ssh:
    KeyName: your_key                     << replace
Scheduling:
  Scheduler: slurm
  SlurmQueues:
    - Name: queue1
      CapacityType: ONDEMAND
      Networking:
        SubnetIds:
          - subnet-xx-xx-x         x    << replace
        PlacementGroup:
          Enabled: true
      ComputeResources:
        - Name: compute-resource-1
          InstanceType: hpc7g.16xlarge
          MinCount: 0
          MaxCount: 10
          DisableSimultaneousMultithreading: true
          Efa:
            Enabled: true
            GdrSupport: false
SharedStorage:
  - MountDir: /shared
    Name: ebs-shared
    StorageType: Ebs
  - MountDir: /fsx
    Name: name2
    StorageType: FsxLustre
    FsxLustreSettings:
      StorageCapacity: 1200
```

The Yaml file for the hpc7g.16xlarge contains the settings as shown in the following diagram.

Figure 1. Diagram of YAML file used to configure a ParallelCluster with a hpc7g.large head node and hpc7g.16xlarge compute nodes using ONDEMAND
![hpc7g.16xlarge yaml configuration](../../yml_plots/hpc7g.16xlarge-yaml.png)



## Create the hpc7g.16xlarge pcluster

`pcluster create-cluster --cluster-configuration hpc7g.16xlarge.ebs_unencrypted_installed_public_ubuntu2004.fsx_import.yaml --cluster-name cmaq --region us-east-1`

#### Check on status of cluster

`pcluster describe-cluster --region=us-east-1 --cluster-name cmaq`


After 5-10 minutes, you see the following status: "clusterStatus": "CREATE_COMPLETE"


#### Login to cluster
```{note}
Replace the your-key.pem with your Key Pair.
```

`pcluster ssh -v -Y -i ~/your-key.pem --region=us-east-1 --cluster-name cmaq`

```{note}
Notice that the hpc7g.16xlarge yaml configuration file contains a setting for PlacementGroup.
```

```
PlacementGroup:
          Enabled: true
```
 
A placement group is used to get the lowest inter-node latency. 

A placement group guarantees that your instances are on the same networking backbone. 

### Check to make sure elastic network adapter (ENA) is enabled

`modinfo ena`

`lspci`

### Check what modules are available on the cluster

`module avail`

Output:

```
module avail
------------------------------------------------------------------------------------------------ /usr/share/modules/modulefiles -------------------------------------------------------------------------------------------------
armpl/21.0.0  dot  libfabric-aws/1.17.1  module-git  module-info  modules  null  openmpi/4.1.5  use.own  
```


### Load the openmpi module

`module load openmpi/4.1.5`

### Load the Libfabric module

`module load libfabric-aws/1.17.1`

### Verify the gcc compiler version is greater than 8.0

`gcc --version`

output:

```
gcc (Ubuntu 9.4.0-1ubuntu1~20.04.1) 9.4.0
Copyright (C) 2019 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

```

```{seealso}
<a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking-ena.html#test-enhanced-networking-ena">Link to the AWS Enhanced Networking Adapter Documentation</a>
```

```{seealso}
<a href="https://docs.aws.amazon.com/parallelcluster/latest/ug/what-is-aws-parallelcluster.html">ParallelCluster User Manual</a>
```
