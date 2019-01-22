# Farnam

![Louise](/img/Louise-Whitman-Farnam.jpg){: .cluster-portrait}

The Farnam Cluster is named for [Louise Whitman Farnam](http://archives.yalealumnimagazine.com/issues/2006_09/old_yale.html), the first woman to graduate from the Yale School of Medicine, class of 1916. It consists of a variety of compute nodes networked over ethernet and mounts several shared filesystems.

- - -

## Hardware

Farnam is made up of several kinds of compute nodes. The Features column below lists the features that can be used to request different node types using the `--constraints` flag (see our [Slurm documentation](/clusters-at-yale/job-scheduling/resource-requests#features-and-constraints) for more details). The RAM listed below is the amount of memory available for jobs.

!!! Warning
    Care should be taken if when scheduling your job if you are running programs/libraries optimized for specific hardware.
    See the [guide on how to compile software](/clusters-at-yale/applications/compile) for specific guidance.

### Compute Node Configurations

| Count | Node Type                     | CPU                 | CPU Cores | RAM   |         GPU        | Features                                     |
|-------|-------------------------------|---------------------|-----------|-------|--------------------|----------------------------------------------|
| 59    | Dell PowerEdge M620           | 2x E5-2670          | 16        | 121G  |                    | sandybridge, sse4_2, avx, E5-2670            |
| 14    | Dell PowerEdge M915           | 4x AMD Opteron 6276 | 32        | 507G  |                    | bulldozer, sse4_2, avx, opteron-6276         |
| 117   | Lenovo nx360h                 | 2x E5-2660 v3       | 20        | 121G  |                    | haswell, v3, sse4_2, avx, avx2, E5-2660_v3   |
| 5     | Lenovo nx360h w/GPUs          | 2x E5-2660 v3       | 20        | 121G  | 2x K80 (2GPUs/K80) | haswell, v3, sse4_2, avx, avx2, E5-2660_v3   |
| 38    | Lenovo nx360b                 | 2x E5-2680 v4       | 28        | 246G  |                    | broadwell, v4, sse4_2, avx, avx2, E5-2680_v4 |
| 3     | Lenovo nx360b w/GPUs          | 2x E5-2660 v4       | 28        | 246G  | 2x P100            | broadwell, v4, sse4_2, avx, avx2, E5-2660_v4 |
| 2     | Lenovo sd530                  | 2x Gold 6132        | 28        | 184G  |                    | skylake, sse4_2, avx, avx2, avx512, 6132     |
| 1     | Lenovo sd530                  | 2x Gold 6132        | 28        | 751G  |                    | skylake, sse4_2, avx, avx2, avx512, 6132     |
| 1     | Thinkmate GPX XT4 (gpu_devel) | 2x E5-2623 v4       | 8         | 58G   | 4x 1080Ti          | broadwell, v4, sse4_2, avx, avx2, E5-2623_v4 |
| 20    | Thinkmate GPX XT4             | 2x E5-2637 v4       | 8         | 121G  | 4x 1080Ti          | broadwell, v4, sse4_2, avx, avx2, E5-2637_v4 |
| 1     | Thinkmate GPX XT4             | 2x E5-2637 v4       | 8         | 121G  | 4x TITAN V         | broadwell, v4, sse4_2, avx, avx2, E5-2637_v4 |
| 2     | Lenovo 3850X6                 | 4x E7-4809 v3       | 32        | 1507G |                    | haswell, v3, sse4_2, avx, avx2, E7-4809_v3   |
| 1     | Lenovo 3850X6                 | 4x E7-4820 v4       | 40        | 1507G |                    | broadwell, v3, sse4_2, avx, avx2, E7-4820_v4 |

## Slurm Partitions

Nodes on the clusters are organized into partitions, to which you submit your jobs with [Slurm](/clusters-at-yale/job-scheduling/slurm).

### Public Partitions

The general partition is where most batch jobs should run, and is the default if you don't specify a partition. The interactive partition is dedicated to jobs with which you need ongoing interaction. The bigmem partition contains our largest memory nodes; only jobs that cannot be satisfied by general should run here. The gpu_devel partition is a single node meant for testing or compiling GPU accelerated code, and the gpu partition is where normal GPU jobs should run. The scavenge partition allows you to run preemptable jobs on more resources than normally allowed. For more information about scavenge, see the [Slurm documentation](/clusters-at-yale/job-scheduling/scavenge).

All the node types listed are described in more detail in the [hardware](#hardware) table.

| Partition   | User Limits                 | Walltime default/max | Node type (number)                  |
|-------------|-----------------------------|----------------------|-------------------------------------|
| interactive | 20 CPUs, 256 G RAM          | 1d/2d                | m620 (34), nx360h (94)              |
| general*    | 200 CPUs, 1280 G RAM        | 1d/30d               | m620 (34), nx360h (94)              |
| scavenge    | 800 CPUs, 5120 G RAM        | 1d/7d                | all                                 |
| gpu_devel   | 1 job                       | 10min/2hr            | GPX XT4 1080Ti (1)                  |
| gpu         | 32 CPUs, 256 G RAM          | 1d/2d                | nx360h K80 (2), GPX XT4 1080Ti (10) |
| bigmem      | 2 jobs, 32 CPUs, 1532 G RAM | 1d/7d                | m915 (9), 3850X6 (2)                |

*default

### Private Partitions

Private partitions contain nodes acquired by specific research groups. Full access to these partitions is granted at the discretion of the owner. Contact us if your group would like to purchase nodes.

| Partition       | Walltime default/max | Node type (number)                                   |
|-----------------|----------------------|------------------------------------------------------|
| pi_breaker      | 1d/14d               | nx360b (24)                                          |
| pi_cryoem       | 1d/∞                 | GPX XT4 1080Ti (10)                                  |
| pi_deng         | 1d/14d               | nx360b (1)                                           |
| pi_gerstein     | 1d/14d               | m915 (2), nx360b (11), sd530 (3), 3850X6 (1)         |
| pi_gerstein_gpu | 1d/14d               | nx360h K80 (3), nx360b P100 (2), GPX XT4 TITAN V (1) |
| pi_gruen        | 1d/14d               | nx360b (1)                                           |
| pi_jadi         | 1d/14d               | nx360b (2)                                           |
| pi_kleinstein   | 1d/14d               | m915 (1), nx360b (3)                                 |
| pi_krauthammer  | 1d/14d               | nx360h (1)                                           |
| pi_ma           | 1d/14d               | nx360h (2)                                           |
| pi_ohern        | 1d/14d               | m620 (6), nx360h (3)                                 |
| pi_sigworth     | 1d/14d               | nx360h (1)                                           |
| pi_sindelar     | 1d/14d               | m620 (4), m915 (1), nx360h (1)                       |
| pi_strobel      | 1d/14d               | m915 (1)                                             |
| pi_townsend     | 1d/14d               | nx360h (5)                                           |
| pi_zhao         | 1d/14d               | m620 (17), m915 (1)                                  |
 
## Public Datasets

We host datasets of general interest in a loosely organized directory tree in `/gpfs/ysm/datasets`:

```
├── cryoem
├── db
│   ├── annovar
│   └── blast
└── genomes
    ├── 1000Genomes
    ├── 10xgenomics
    ├── Aedes_aegypti
    ├── Chelonoidis_nigra
    ├── Danio_rerio
    ├── hisat2
    ├── Homo_sapiens
    ├── Mus_musculus
    ├── PhiX
    └── Saccharomyces_cerevisiae
```

If you would like us to host a dataset or questions about what is currently available, please email hpc@yale.edu.

## Storage

Farnam has access to a number of GPFS filesystems. `/gpfs/ysm` is Farnam's primary filesystem where home, project, and scratch60 directories are located. For more details on the different storage spaces, see our [Cluster Storage](/clusters-at-yale/data/cluster-storage) documentation.

You can check your current storage usage & limits by running the `getquota` command. Note that the per-user usage details only update once daily.

!!! Warning
    Files stored in `scratch60` are purged if they are older than 60 days. You will receive an email alert one week before they are deleted.

|Partition  | Root Directory        | Storage     | File Count | Backups |
|-----------|-----------------------|-------------|------------|---------|
| home      | `/gpfs/ysm/home`      | 125G/user   | 500,000    | Yes     |
| project   | `/gpfs/ysm/project`   | 4T/group    | 5,000,000  | No      |
| scratch60 | `/gpfs/ysm/scratch60` | 10T/group   | 5,000,000  | No      |