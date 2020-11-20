---
title: Workload Manager
tags: HPC 
---
<!--more-->

# What is a Workload Manager
+ Commonly called a Workload Manager. May also be referred to (sometimes loosely) as:

    - Batch system
    - Batch scheduler
    - Workload scheduler
    - Job scheduler
    - Resource manager (usually considered a component of a Workload Manager) 

+ Tasks commonly performed by a Workload Manager:

    - Provide a means for users to specify and submit work as "jobs"
    - Evaluate, prioritize, schedule and run jobs
    - Provide a means for users to monitor, modify and interact with jobs
    - Manage, allocate and provide access to available machine resources
    - Manage pending work in job queues
    - Monitor and troubleshoot jobs and machine resources
    - Provide accounting and reporting facilities for jobs and machine resources
    - Efficiently balance work over machine resources; minimize wasted resources 

+ Generalized architecture and workflow of a Workload Manager: 

![WorkloadMgrWorkflow](/assets/img/blog/hpc/workloadMgrWorkflow.png)

 **User** 

  - Logs into cluster
  - Creates job script and submits it to workload manager
  - Monitors and interacts with job via workload manager
  - Queries workload manager for job and cluster information 

**Workload Manager**
  - Typically runs on a separate server as multiple processes
  - Receives job submissions, commands, queries from user
  - Matches job requirements to available machine resources
  - Evaluates, prioritizes and queues jobs
  - Schedules jobs for execution on cluster
  - Tracks job and cluster information
  - Sends jobs to compute node daemons for actual execution 

**Cluster**
  - Workload Manager daemons run on compute nodes
  - Daemons manage compute resources and job execution
  - Daemons communicate with Workload Manager server processes 

+ Some popular Workload Managers include:
  - Slurm from SchedMD
  - Spectrum LSF from IBM
  - Tivoli Workload Scheduler (LoadLeveler) from IBM
  - PBS from Altair Engineering
  - TORQUE, Maui, Moab from Adaptive Computing
  - Univa Grid Engine
  - OpenLava 



[Workload Manager](https://computing.llnl.gov/tutorials/moab/)