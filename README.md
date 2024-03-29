<br/>
<p align="center">
    <a href="https://hpc.stat.yonsei.ac.kr/" target="_blank">
        <img width="50%" src="https://hpc.stat.yonsei.ac.kr/images/logo.svg" alt="Yonsei BK logo">
    </a>
</p>

<br/>
<p align="center">
</p>
<br/>

<p align="center">
    <a href="https://hpc.stat.yonsei.ac.kr">
        <img width="20%" src="https://hpc.stat.yonsei.ac.kr/images/chili.png" alt="Chili Pepper Cluster Logo">
    </a>
</p>
<br/>


|   NodeName  |                                                    OS                                                    | Cores |  RAM  | vRAM |                                       Conda                                      |                                               Cuda                                               |
|:-----------:|:--------------------------------------------------------------------------------------------------------:|:-----:|:-----:|:----:|:--------------------------------------------------------------------------------:|:------------------------------------------------------------------------------------------------:|
| cpu-compute | ![Ubuntu: 18.04](https://img.shields.io/static/v1?label=ubuntu&message=18.04.6&color=e95420&logo=ubuntu) |   32  | 128GB |   -  | ![Conda: 4.11.0](https://img.shields.io/badge/conda%7Cconda--forge-v4.11.0-blue) |                                                 -                                                |
| gpu-compute | ![Ubuntu: 18.04](https://img.shields.io/static/v1?label=ubuntu&message=18.04.6&color=e95420&logo=ubuntu) |   16  |  80GB |  32  | ![Conda: 4.6.14](https://img.shields.io/badge/conda%7Cconda--forge-v4.6.14-blue) | ![Cuda: 10.1](https://img.shields.io/static/v1?label=cuda&message=10.1&color=76b900&logo=nvidia) |

[Chili Pepper Cluster](https://hpc.stat.yonsei.ac.kr/) is a data science computing cluster configured to run science workloads via [SLURM](https://slurm.schedmd.com). The cluster is hosted via Naver Cloud Platform and configured by student administrators. This project is funded by BK21 Foundation and Yonsei University Department of Statistics and Data Science.

This repository contains various documentation for configuring and operating cloud-based HPC clusters and GPU on-premise servers. All documents are markdown files for accessibility. You can commit documentation adhering to the following steps:  


***Creating a new documentation topic***

1. Look for an undocumented topic in this repository(`GitHub > Projects`).

2. Create a new folder for the selected topic and create a `README.md` file under the directory.  

```
.
├── README.md
└── new-topic
    └── README.md
```

3. Write the following meta data in the `README.md` in `yaml` format.  

*Example*

```
---
title: SOME_TITLE
author: SOME_ONE
---
```

***Adding new content for existing topic***

1. Navigate to the existing topic folder and open the associated `README.md` file. 

2. If you are not the author, append your name to the `author` in the metadata section like so.


*Example*

```
---
title: SOME_TITLE
author: OG_Author, NEW_Author
---
```


3. Add and commit changes with a useful commit message. 

*Example*

```bash
# for updates
$ git commit -m "[update]some new feature"

# for deprecation
$ git commit -m "[deprecated]some old feature"
```

***Footnotes***

Markdown supports footnotes with the following syntax(`[^NAME]`). Footnotes will be rendered easily in GitHub.

```markdown
Einstein said "..."[^fn1].

...

# The end of markdown document.
[^fn1]: Some Link or Reference
```

***Feedbacks for documents***

*Comment on the commit*  

To add feedbacks, click the latest commit associated to the document of interest and add a comment. 
