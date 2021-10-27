---
title: How to write documentation for this repo
author: Dongook Son
---

## Intro

This repository contains various documentation for configuring and operating cloud-based HPC clusters and GPU on-premise servers. All documents should be in a `.md` format. You can commit documentation adhering to the following steps:  

## Creating a new documentation topic

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

## Adding new content for existing topic

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

## Footnotes

Markdown supports footnotes with the following syntax(`[^NAME]`). Footnotes will be rendered easily in GitHub.

```markdown
Einstein said "..."[^fn1].

...

# The end of markdown document.
[^fn1]: Some Link or Reference
```

## Feedbacks for documents
To add feedbacks, click the latest commit associated to the document of interest and add a comment. 