---
title: RStudio Server with SLURM for Yonsei Data Science Machine
author: Jongmin Mun
---

# RStudio Server with SLURM for Yonsei Data Science Machine

## Intro

### RStudio
RStudio is an integrated development environment for the R programming language, with limited support for other programming languages (including Python, bash, and SQL). RStudio provides a powerful graphical environment for importing data in a number of formats (including CSV, Excel spreadsheets, SAS, and SPSS); manipulating, analyzing, and visualizing data; version control with git or SVN; a graphical R package manager that provides point/click search/installation/uninstallation of R packages from its substantial ecosystem (including the Bioconductor repository, which provides almost 1500 software tools “for the analysis and comprehension of high-throughput genomic data.”); and many other features.[^fn1]

### RStudio Server
RStudio Server is a client/server version of RStudio that runs on a remote server and is accessed via the client’s web browser. A graphical file manager allows file upload/download from hpc-class via web browser.

### Scaling R and Rstudio
There are typiccally three use cases for scaling R:
1. Scaling for many R Users - for multiple data science researchers or students to seamlessly work on a HPC(high performance computing) environment for multiple projects.
2. Scaling for HPC - for parallel tasks cross validation.
3. Scaling for Big Data - for data that can't fit on one machine.

|Use Case|Problem|Solutions|Possible Technology|
|------|---|---|--|
|Scaling for Many R Users|Regular R workflows for a team. Includes loading data subsets from files or warehouses|Create a platform to support large-scale individual interactive R session(s) and jobs|RStudio Server, RStudio Workbench + Load Balancer, RStudio Workbench + Launcher|
|Scaling for HPC|Embarrassingly parallel tasks like: bootstrapping, cross validation, scoring, model fitting on independent groups|Develop code in an interactive R session in RStudio. Submit code in batch jobs on compute R processes. R must be installed on all compute nodes.|Local: parallel, Rmpi, snow, Rcpp parallel; Cluster: RStudio Workbench + Launcher, Kubernetes, Slurm, LSF, Torque, Docker; Recommendation: batchtools package|
|Scaling for Big Data|Big data, black box routines that require fitting a model against an entire domain space. Data can’t fit on one machine.|R is an orchestration engine. Heavy lifting is done by a different compute engine on the cluster. R syntax is used to construct pipelines, and R is used to analyze results.|Hadoop, Spark, Tensorflow, Oracle BDA, Microsoft R Server, Aster, H2O.ai|
[^fn2]

## Singularity and Rocker Image
RStudio Server will be available on the cluster using a Docker image(imported into Singularity) provided by the Rocker project.[^fn4]

### Rocker Project
The Rocker project provides a widely-used suite of Docker images with
customized R environments for particular tasks.[^fn5]

### Singularity
SingularityCE is a container platform. It allows you to create and run containers that package up pieces of software in a way that is portable and reproducible. You can build a container using SingularityCE on your laptop, and then run it on many of the largest HPC clusters in the world, local university or company clusters, a single server, in the cloud, or on a workstation down the hall. Your container is a single file, and you don’t have to worry about how to install all the software you need on each different operating system. 

SingularityCE was created to run complex applications on HPC clusters in a simple, portable, and reproducible way. First developed at Lawrence Berkeley National Laboratory, it quickly became popular at other HPC sites, academic sites, and beyond. SingularityCE is an open-source project, with a friendly community of developers and users. The user base continues to expand, with SingularityCE now used across industry and academia in many areas of work. [^fn6]

Singularity is useful for running containers as an unprivileged user, especially in multi-user environments like High-Performance Computing clusters. Rocker images can be imported and run using Singularity, with optional custom password support.[^fn4]

## SLURM

The cluster is managed by the SLURM queueing software. SLURM provides a standard batch queueing system through which users submit jobs to the cluster. Jobs are typically submitted to SLURM using a user-defined shell script that executes one's application code. Interactive use is also an option. Users may also query the cluster to see job status.[^fn3]

To use RStudio Server on the cluster, a user submits a SLURM job script. This allows RStudio Server to run on any available cluster compute resources (including large-memory nodes).[^fn4]

A default job script that should suffice for most users would be provided.[^fn3]

### Submitting Parallel Jobs
One can use SLURM to submit a variety of types of parallel code. A set of potentially useful templates would be provided such that we expect will account for most user needs.[^fn3]

## Use case
Many institutions run RStudio server managed by SLURM, 
### Department of Statistics, University of California Berkely [^fn3]
### High Performance Computing Cluster, The Iowa State University [^fn4]
### Princeton Research Computing [^fn9]

Especially, the Iowa State University uses the combination of Rocker image, Singularity and SLRUM.[^fn4]


[^fn1]: https://www.hpc.iastate.edu/guides/containers/rstudio
[^fn2]: https://support.rstudio.com/hc/en-us/articles/236226087-Scaling-R-and-RStudio
[^fn3]: https://statistics.berkeley.edu/computing/servers/cluster
[^fn4]: https://www.hpc.iastate.edu/guides/containers/rstudio
[^fn5]: https://journal.r-project.org/archive/2017/RJ-2017-065/RJ-2017-065.pdf
[^fn6]: https://sylabs.io/guides/latest/user-guide/introduction.html#why-use-singularityce
[^fn7]: https://www.rocker-project.org/use/singularity/
[^fn9]: https://researchcomputing.princeton.edu/support/knowledge-base/rrstudio#ondemand
