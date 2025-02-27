# Use ParallelCluster with Software and Data pre-installed on hpc7g.16xlarge

Step by step instructions to configuring and running a ParallelCluster for the CMAQ 12US1 benchmark 

```{admonition} Notice
:class: warning

The CMAQ libraries were installed using the gcc compiler on c6g.large.

```


## Configure the ParallelCluster

Use an existing yaml file from the git repo to create a ParallelCluster

`cd /your/local/machine/install/path/`

Use a configuration file from the github repo that was cloned to your local machine

`git clone -b main https://github.com/CMASCenter/pcluster-cmaq.git pcluster-cmaq`


`cd pcluster-cmaq/yaml`

Edit the hpc7g.16xlarge.ebs_unencrypted_installed_public_ubuntu2004.fsx_import.yaml

`vi hpc7g.16xlarge.ebs_unencrypted_installed_public_ubuntu2004.fsx_import.yaml`

```{note}
1. the hpc7g.16xlarge*.yaml is configured to use ONDEMAND instance pricing for the compute nodes.
2. the hpc7g.16xlarge*.yaml is configured to the the hpc7g.16xlarge as the compute node for the compute-resource-1 queue, with up to 12 compute nodes, specified by MaxCount: 12.
3. the hpc7g.16xlarge*.yaml is configured to the the hpc7g.8xlarge as the compute node for the compute-resource-1 queue, with up to 7 compute nodes.
3. the hpc7g.16xlarge*.yaml is configured to disable multithreading (This option restricts the computing to CPUS rather than allowing the use of all virtual CPUS. (128 virtual cpus reduced to 64 cpus)
4. the hpc7g.16xlarge*.yaml is configured to enable the setting of a placement group to allow low inter-node latency
5. the hpc7g.16xlarge*.yaml is configured to enables the elastic fabric adapter
6. given this yaml configuration, the maximum number of PEs that could be used to run CMAQ is 64 cpus x 12 = 768, the max settings for NPCOL, NPROW is NPCOL = 24, NPROW = 32 or NPCOL=32, NPROW=24 in the CMAQ run script. Note: CMAQ will need to be benchmarked using the 12US1 to determine the optimal number of compute nodes to use.
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
  LocalStorage:
    RootVolume:
      Encrypted: true
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
          MaxCount: 12
          DisableSimultaneousMultithreading: true
          Efa:
            Enabled: true
            GdrSupport: false
        - Name: compute-resource-2
          InstanceType: hpc7g.8xlarge
          MinCount: 0
          MaxCount: 7
          DisableSimultaneousMultithreading: true
          Efa:
            Enabled: true
            GdrSupport: false
SharedStorage:
  - MountDir: /shared
    Name: ebs-shared
    StorageType: Ebs
    EbsSettings:
      Encrypted: true
      SnapshotId: snap-0049a7c309f238500
  - MountDir: /fsx
    Name: name2
    StorageType: FsxLustre
    FsxLustreSettings:
      StorageCapacity: 1200
      ImportPath: s3://cmas-cmaq/
```

The Yaml file for the hpc7g.16xlarge contains the settings as shown in the following diagram.

Figure 1. Diagram of YAML file used to configure a ParallelCluster with a c6g.large head node and hpc7g.16xlarge compute nodes using ONDEMAND pricing
![hpc7g.16xlarge yaml configuration](../../../yml_plots/hpc7g.16xlarge.png)

(to do!)


## Create the hpc7g pcluster

Note, this yaml file is configured to have 12 nodes of the hpc7g.16xlarge (64 pe per node) and 7 nodes of the hpc7g.8xlarge (32 pe per node).

`pcluster create-cluster --cluster-configuration hpc7g.16xlarge.ebs_unencrypted_installed_public_ubuntu2004.fsx_import.yaml --cluster-name cmaq --region us-east-1`

Check on status of cluster

`pcluster describe-cluster --region=us-east-1 --cluster-name cmaq`


After 5-10 minutes, you see the following status: "clusterStatus": "CREATE_COMPLETE"

If the cluster fails to start, use the following command to check for an error

`pcluster get-cluster-stack-events --cluster-name cmaq --region us-east-1 --query 'events[?resourceStatus==`CREATE_FAILED`]'`
