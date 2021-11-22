---
title: NAS configuration
author: Dongook Son
---

## Mount information

Information on connecting NCloud NAS to NCloud server instance.

```yaml
mount point: "169.254.82.85:/n2717169_bkstorage"
```

1. Install `nfs-common` for Ubuntu systems.
2. Mount with the following command.
    ```bash
    sudo mkdir /mnt/nas
    sudo mount -t nfs 169.254.82.85:/n2717169_bkstorage /mnt/nas
    ```

3. Change ownership of `/mnt/nas` to `sudo`.
    ```bash
    sudo chown -R sudo /mnt/nas
    ```

4. Create `admin` folder and `users` folder.
    ```bash
    sudo mkdir /mnt/nas/admin
    sudo mkdir /mnt/nas/users
    ```

5. Change `/etc/fstab` for auto-mount after reboot.
    ```bash
    # /etc/fstab
    169.254.82.85:/n2717169_bkstorage /mnt/nas nfs defaults 0 0
    ```

## User Storage

1. Under `/mnt/nas/users`, create folders for individual users(don't forget to change permissions) and create symlink to that folder with each users' `~/Storage`. 
    ```bash
    sudo mkdir /mnt/nas/users/USERNAME
    sudo chown -R USERNAME /mnt/nas/users/USERNAME
    sudo ln -s /mnt/nas/users/USERNAME /home/USERNAME/Storage
    ```



[NCloud NAS User Guide]: https://guide.ncloud-docs.com/docs/en/storage-storage-4-1