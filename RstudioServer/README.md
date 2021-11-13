---
title: RStudio Server with SLURM for Yonsei Data Science Machine
author: Jongmin Mun, Dongook Son
---

# RStudio Server with SLURM for Yonsei Data Science Machine

## Intro

### RStudio
RStudio is an integrated development environment for the R programming language, with limited support for other programming languages (including Python, bash, and SQL).

The advantages of RStudio can be summarized as follows:
1. a powerful graphical environment for importing data in a number of formats (including CSV, Excel spreadsheets, SAS, and SPSS).
2. Easy manipulating, analyzing, and visualizing data.
3. Version control with git or SVN.
4. A graphical R package manager that provides point/click search/installation/uninstallation of R packages from its substantial ecosystem.
[^fn1]

### RStudio Server
RStudio Server is a client/server version of RStudio that runs on a remote server and is accessed via the client’s web browser. A graphical file manager allows file upload/download from HPC via web browser.

### RStudio Server Installation for Single Server

#### Prerequisites

1. Check for network portforwarding rules and firewall rules. Make sure that port `8787` is accessible since this is the default port that RStudio Server listens to.

2. Add CRAN repository to package manager. For Ubuntu servers the following command will install helper packages, add the gpg key for the cran repository.
```sh
# install helper packages
sudo apt install --no-install-recommends software-properties-common dirmngr

# get gpg key
wget -qO- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc | sudo tee -a /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc

# add R4.0 repo from CRAN
sudo add-apt-repository "deb https://cloud.r-project.org/bin/linux/ubuntu $(lsb_release -cs)-cran40/"
```

3. Install base R without additional dependencies.
```sh
sudo apt install --no-install-recommends r-base 
```

#### Install RStudio Server

1. Install `gdebi` in order to install local `deb` packages.
```sh
sudo apt install gdebi-core
```

2. Download RStudio Server `deb` package and install.
```sh
wget https://download2.rstudio.org/server/bionic/amd64/rstudio-server-2021.09.1-372-amd64.deb
sudo gdebi rstudio-server-2021.09.1-372-amd64.deb
```

#### Post-installation

On a client webbrowser, go to `http://<server-public-ip>:8787`. As the default authenticator is PAM, login with *preexisting system-wide* username and password. 

### Scaling R and Rstudio
There are typiccally three use cases for scaling R in HPC(high performance computing) environment:
1. Scaling for many R Users - for multiple data science researchers or students to seamlessly work on a HPC environment for multiple projects.
2. Scaling for HPC - for parallel tasks like cross validation.
3. Scaling for Big Data - for datasets that can't fit on one machine.

|Use Case|Problem|Solutions|Possible Technology|
|------|---|---|--|
|Scaling for Many R Users|Regular R workflows for a team. Includes loading data subsets from files or warehouses|Create a platform to support large-scale individual interactive R session(s) and jobs|RStudio Server, RStudio Workbench + Load Balancer, RStudio Workbench + Launcher|
|Scaling for HPC|(1) Embarrassingly parallel tasks like: bootstrapping, cross validation, scoring, model fitting on independent groups. (2) Matrix-intensive computations like: Gaussian process regression or MCMC computation for general non-conjugate Bayesian inference. |Develop code in an interactive R session in RStudio. Submit code in batch jobs on compute R processes. R must be installed on all compute nodes.|Local: parallel, Rmpi, snow, Rcpp parallel; Cluster: RStudio Workbench + Launcher, Kubernetes, Slurm, LSF, Torque, Docker; Recommendation: batchtools package|
|Scaling for Big Data|Big data, black box routines that require fitting a model against an entire domain space. Data can’t fit on one machine.|R is an orchestration engine. Heavy lifting is done by a different compute engine on the cluster. R syntax is used to construct pipelines, and R is used to analyze results.|Hadoop, Spark, Tensorflow, Oracle BDA, Microsoft R Server, Aster, H2O.ai|

[^fn2]

## Singularity and Rocker Image
RStudio Server will be available on the cluster using a Docker image(imported into Singularity) provided by the Rocker project[^fn4] and docker image for Microsoft R open(for parallelization).

### Rocker Project
The Rocker project provides a widely-used suite of Docker images with
customized R environments for particular tasks.[^fn5]

### Microsoft R open(MRO)
Microsoft R Open is the enhanced distribution of R from Microsoft Corporation. Its main features include: [^fn10]
1. **MKL library.** Multi-threaded math libraries that brings multi-threaded computations to R.
   - This boosts up the speed of matrix-intensive calculations.
   - For example, in some Bayesian inference for non-conjugate models, at each step of the MCMC chain, we need to repeat matrix operations such as QR decomposition, Cholesky, etc., and in that case wee will see a substantial speedup. [^fn11]
2. **MRAN.** A high-performance default CRAN repository that provide a consistent and static set of packages to all Microsoft R Open users.
   - This stores all the snapshots of the packeges of CRAN repository.
   - This is known to often cause errors, so we think it's better not to use it. We can easily switch to CRAN with chooseCRANmirror().[^fn11]
3. **checkpoint package.** that make it easy to share R code and replicate results using specific R package versions.
   - This enables locking down the versions of packages being used once we release the scripts, thus increasaing the reproducibility.

For a docker image for MRO, we can use a project inspired by Rocker project: [^fn12]
- o2r team member Daniel created a Docker image for MRO including MKL. It is available on Docker Hub as nuest/mro, with Dockerfile on GitHub. It is inspired by the Rocker images and can be used in the same fashion. Please note the extended licenses printed at every startup for MKL. [^fn13]

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
[^fn10]: https://mran.microsoft.com/rro#intelmkl1
[^fn11]: https://community.rstudio.com/t/base-r-vs-microsoft-r-open/1757
[^fn12]: https://www.r-bloggers.com/2016/12/investigating-docker-and-r-2/
[^fn13]: https://hub.docker.com/r/nuest/mro/