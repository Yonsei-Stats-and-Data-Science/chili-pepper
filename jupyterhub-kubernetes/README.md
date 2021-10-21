---
title: JupyterHub with Kubernetes for Yonsei Data Science Machine
author: Dongook Son
---

# JupyterHub with Kubernetes for Yonsei Data Science Machine

## Intro

For multiple data science researchers or students to seamlessly work on a HPC(high performance computing) environment for multiple projects, it is much efficient to provide common research tools such as jupyter notebooks and R studio instead of just giving them the ssh login information.

Although this is possible via projects such as JupyterHub[^fn1] and RStudio Workbench[^fn2], the server needs to be carefully configured for a stable and efficient implementation. This document will cover configuration of JupyterHub for a mid-size(100-200 users) academic research environment.

This document will contain similar content with the Readthedocs page of JupyterHub[^fn3] but with additional information regarding the particular setup environment in mind. Since the server environment will be comprised of multiple computers with various CPUs and GPUs, the Zero to JupyterHub on Kubernetes[^fn4] will be the building block and the main reference to this manual.

## JupyterHub

JupyterHub can server multiple notebooks for multiple users comprised of the following four subsystems[^fn3].

1. Hub: tornado(python web framework & asynchronous networking library)[^fn5]
2. Configurable http proxy: node module that provides a way to update and manage a proxy table via CLI or REST API.[^fn6]
3. multiple single-user Jupyter notebook servers monitored by Spawners.
4. Authentication class that manages how users can access the system. Controlled via configuration file. 

![gist of jupyterhub](https://jupyterhub.readthedocs.io/en/stable/_images/jhub-fluxogram.jpeg)

[^fn1]: https://github.com/jupyterhub/jupyterhub
[^fn2]: https://www.rstudio.com/products/workbench/
[^fn3]: https://jupyterhub.readthedocs.io/en/stable/
[^fn4]: https://zero-to-jupyterhub.readthedocs.io/en/latest/
[^fn5]: https://www.tornadoweb.org/en/stable/
[^fn6]: https://www.npmjs.com/package/configurable-http-proxy
