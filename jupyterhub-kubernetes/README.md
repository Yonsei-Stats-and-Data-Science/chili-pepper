---
title: JupyterHub with Kubernetes for Yonsei Data Science Machine
author: Dongook Son
---

# JupyterHub with Kubernetes for Yonsei Data Science Machine

## Intro

For multiple data science researchers or students to seamlessly work on a HPC(high performance computing) environment for multiple projects, it is much efficient to provide common research tools such as jupyter notebooks and R studio instead of just giving them the ssh login information.

Although this is possible via projects such as JupyterHub[^fn1] and RStudio Workbench[^fn2], the server needs to be carefully configured for a stable and efficient implementation. This document will cover configuration of JupyterHub for a mid-size(100-200 users) academic research environment.


## References

[^fn1]: https://github.com/jupyterhub/jupyterhub
[^fn2]: https://www.rstudio.com/products/workbench/
