---
title: User Management
author: Dongook Son
---

## Prerequisite

- NAS must be mounted before user creation.

- Enroll student to this github repo or Slack group.

- Create instruction files in `/etc/skel`.

- Create default group for users(`bkusers`).
    ```bash
    sudo groupadd bkusers
    ```


## Enrollment for CPU Server

Since we start as a beta-test, collect people who wants to participate and use the compute machine. 

1. Create user via `useradd` and create a password.

```bash
UNAME=someusername
sudo useradd -m -d /mnt/nas/users/$UNAME -G bkusers $UNAME && sudo passwd $UNAME
```

2. Notify the user of login details.

*UPDATE*

In `/mnt/nas/admin` use `create-bkuser.sh` in the following way.
```bash
./create-bkuser.sh USERNAME
```
