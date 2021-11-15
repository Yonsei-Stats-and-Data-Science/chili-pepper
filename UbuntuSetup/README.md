---
title: Ubuntu Setup
author: Dongook Son
---

## Intro

Assuming `ssh` connection is enabled with the remote instance(cloud or on-premise server), the following instructions are additional steps for configuring a minimal data science envrionment. Please note that on cloud servers `ssh` will be preinstalled. If you are configuring an on-premise server, consider installing `ssh` prior to other configurations for remote access.

### Login Configuration

To add protection from brute-force attacks, on the master node(the machine that is directly connected to the internet) create a user with `sudo` privileges and disable `root` user login.

```sh
# Create user(USERNAME) and add to sudo group
sudo adduser USERNAME
usermod -aG sudo USERNAME
```

You can disable `root` user login by editing the `/etc/passwd` file and the `/etc/ssh/sshd_config` file. First, in `/etc/passwd` file, replace the `root` user shell to `/sbin/nologin`.
```sh
# /etc/passwd
root:x:0:0:root:/root:/sbin/nologin
```

Disable `root` ssh access by configuring the `/etc/ssh/sshd_config` file. Change the `PermitRootLogin` option to `no`.
```sh
# /etc/ssh/sshd_config
PermitRootLogin no
```

### Network Configuration

After logging in to the newly created account that has `sudo` privileges, install firewall(`ufw`) and allow ssh.
```sh
sudo apt install ufw
sudo ufw enable
sudo ufw allow ssh
```

For additional security, install `fail2ban` for brute-force login attacks. This software will listen on incoming connections and *jail* ip address that fails to login multiple times. Settings can be configured to the system administrators preference.
```sh
sudo apt install fail2ban
sudo systemctl enable fail2ban
```

### Container Software Configuration

Install docker(or podman) to run or build docker images. The details for installation will be defered to the [official documentation](https://docs.docker.com/engine/install/ubuntu/).

### GPU Related

#### NVIDIA driver
For GPU equipped machines, the proper Nvidia driver must be installed to take advantage of the fast computing performance. The instructions for driver installation is defered to the [official document](https://docs.nvidia.com/datacenter/tesla/tesla-installation-notes/index.html). 


#### CUDA
CUDA is a parallel computing platform and programming model invented by NVIDIA.[^1] Note that fast computation for tabular data can be implemented easily by using the [CUDF framework](https://github.com/rapidsai/cudf) since it uses similar APIs with the Python Pandas library.


### Backend Software Configuration

#### Conda

Install conda from the [official repository](https://repo.anaconda.com/miniconda/).
```bash
# download conda for x86_64 machines with Python 3.9 support
wget https://repo.anaconda.com/miniconda/Miniconda3-py39_4.10.3-Linux-x86_64.sh
bash Miniconda3-py39_4.10.3-Linux-x86_64.sh
```

#### Node.js

Install `node.js` from the snap store on Ubuntu / Debian.
```bash
sudo snap install node
```


[^1]: https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#introduction