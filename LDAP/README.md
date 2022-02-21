---
title: "What is LDAP"
author: "Dongook Son"
date: "Feb 21, 2022"
---

# LDAP

## What is LDAP

- Lightweight Directory Access Protocol.
- Open, cross-platform protocol for directory services and authentication
- Runs on TCP/IP

## How it is used

- Central node for authentication
- Stores users, passwords, accounts, dot files

## Key Operation Types

[link](https://ldap.com/ldap-operation-types/)

- Bind: authenticate a user & change identity of the client connection
- Unbind: close connection to the directory server


**references**

- [keeping accounts consistent across clusters using LDAP](https://slurm.schedmd.com/SLUG18/ewan_roche_slug18.pdf)

- [slurm tools repo](https://github.com/OleHolmNielsen/Slurm_tools/tree/master/slurmaccounts)

- [ldap blog post](https://blog.hkwon.me/use-openldap-part1/)