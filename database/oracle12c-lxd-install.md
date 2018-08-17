Oracle 12c LXD install
============================

Sources:

 - https://linuxcontainers.org/lxd/getting-started-cli/
 - https://www.cyberciti.biz/faq/how-to-install-lxd-container-hypervisor-on-ubuntu-16-04-lts-server/
 - http://www.resetlogs.com/2016/01/oracle-db12c-linux-lxc.html


Install

    sudo apt install lxd lxd-client


After install

    # Init storage, network
    sudo lxd init
    
    # User needs to be in lxd group. Logout,login or for a quick test:
    newgrp lxd
    
    # May need to add user to lxd group if not done by `lxd init`. Log out and back in.
    sudo usermod -aG lxd ${USER}


Mount local folders into container:

    lxc config device add oracle-db hostshare disk path=/mnt/hostshare source=/home/username/Downloads/oracle
    lxc config device add oracle-db u01 disk path=/u01/app source=/home/username/Soft/oracle/database

The local folders must have write permissions for `others` group.


**Copy files from host to container**  

LXC containers share the same kernel as the host, so any filesystem they mount should be accessible from outside.

If you do a `cat /proc/mounts` on the host, can you see the container filesystems?
If you see a line like `/dev/mapper/... /var/lib/lxc/o1/rootfs ext4` ... then you should be able to access `/var/lib/lxc/o1/rootfs` from the host, without any further commands.
