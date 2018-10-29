Oracle 12c Docker install
============================

This is a short tutorial on silently installing Oracle 12c in a Docker container with Oracle Linux.  
Go with the Docker Hub version if you find the way it's configured works for you. It's the first link in the list of sources below.  
This tutorial will use Ubuntu as a host system.  

The main steps are:

 - Install Docker and the Oracle Linux image
 - Oracle DB preinstall
 - Oracle DB silent install
 - Oracle DB post install configs

Sources:

 - https://hub.docker.com/r/sath89/oracle-12c/
 - https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04
 - https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes
 - https://oracle-base.com/articles/12c/oracle-db-12cr2-installation-on-oracle-linux-6-and-7
 - https://oracle-base.com/articles/misc/oui-silent-installations
 - http://www.resetlogs.com/2016/01/oracle-db12c-linux-lxc.html
 - https://docs.oracle.com/database/121/LADBI/app_nonint.htm#LADBI7832


### Docker

Install Docker by typing in the terminal

```sh
sudo apt install docker.io

sudo systemctl status docker

# Add user to docker group. Log out and back in.
sudo usermod -aG docker ${USER}
```

Create `/etc/docker/daemon.json` file to set the path to store images/containers and to set storage driver.

```json
{
    "graph": "/home/username/apps/docker/images",
    "storage-driver": "overlay2"
}
```

Create and run the Oracle Linux container.  
In the terminal:  

```sh
docker search oracle

docker pull oraclelinux

docker run -td --name oracle_linux oraclelinux

# Optionally mount a host directory into the container
# docker run -td --name oracle_linux -v /host/src/path:/container/path oraclelinux
```


### Main setup

To log into the container's bash shell

```sh
docker exec -it oracle_linux /bin/bash
```

**From inside the `oracle_linux` container**

Edit the `/etc/hosts` file of oracle_linux to contain a fully qualified name for the server.  
Below, the Linux hostname is the same as the Docker image id.  
You will need root access for most of the operations.  

```sh
120.0.0.1   localhost
172.18.0.2  c67ff272b5ef
```

Install Oracle `preinstall` package. In the terminal:

```sh
yum -y install oracle-rdbms-server-12cR1-preinstall
```

As root, create the inventory file and directories in oracle_linux.  
In the terminal:  

```sh
if [ ! -d /u01 ]; then
    mkdir -p /u01/app/oracle/distribs
    mkdir -p /u01/app/oraInventory
    chown -R oracle:dba /u01
fi

if [ ! -f /etc/oraInst.loc ]; then
    cat >/etc/oraInst.loc <<EOF
inventory_loc=/u01/app/oraInventory
inst_group=oinstall
EOF
    chown root:oinstall /etc/oraInst.loc
    chown 640 /etc/oraInst.loc
fi
```

**From the host system**

Download database install archives from Oracle.  
Get the `db_install.rsp` file from the archive. Example `rsp` configs can be found at: https://oracle-base.com/articles/misc/oui-silent-installations  
Configure `rsp` settings to your liking. Mind the passwords. The file is used to provide parameters for the silent install.  
This setup assumes `INSTALL_DB_AND_CONFIG` as the `oracle.install.option` so as to also create the starter DB.  

Copy install files and `rsp` file to container. From the host terminal:  

```sh
docker cp /local/path/file1.zip oracle_linux:/container/path
```

**From the container**

Run silent install:   

```
./runInstaller -ignoreSysPrereqs -ignorePrereq -waitforcompletion -showProgress -silent -responseFile /tmp/12cR1.rsp
```

Run root script `root.sh` as instructed by the installer at the end.  


### Post installation config using a response file

To complete the installation run the configuration assistants. These need a password response file in order to access various components.  
The assistants are started with a script called `configToolAllCommands`.  
The passwords are of the accounts created during install.  

**From the container**

Create config file `cfgpwd.properties`. Write your passwords.  
Some property values will differ from the example below if you want a multitenant install.  
Further details: https://docs.oracle.com/database/121/LADBI/app_nonint.htm#LADBI7849

Config for DB not in a multitenant setup:  

```
oracle.assistants.server|S_SYSPASSWORD=oracle
oracle.assistants.server|S_SYSTEMPASSWORD=oracle
oracle.assistants.server|S_DBSNMPPASSWORD=oracle
oracle.assistants.server|S_EMADMINPASSWORD=oracle
```

Run config script  

```
/u01/app/oracle/product/12.1.0.2/db_1/cfgtoollogs/configToolAllCommands RESPONSE_FILE=cfgpwd.properties
```


### Starting the database

Set environment variables - create `setenv.sh` file. The file can be executed manually before starting the database or appended to `bash_profile` or a Linux service config.  

```sh
cat > /home/oracle/scripts/setEnv.sh <<EOF
# Oracle Settings
export TMP=/tmp
export TMPDIR=\$TMP

export ORACLE_HOSTNAME=localhost
export ORACLE_UNQNAME=orcl
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=\$ORACLE_BASE/product/12.1.0.2/db_1
export ORACLE_SID=orcl

export PATH=/usr/sbin:/usr/local/bin:\$PATH
export PATH=\$ORACLE_HOME/bin:\$PATH

export LD_LIBRARY_PATH=\$ORACLE_HOME/lib:/lib:/usr/lib
export CLASSPATH=\$ORACLE_HOME/jlib:\$ORACLE_HOME/rdbms/jlib
EOF
```

Run with: `dbstart $ORACLE_HOME`  
Stop with: `dbshut $ORACLE_HOME`

Example `start_db.sh`:

```sh
#!/bin/bash
. /home/oracle/scripts/setenv.sh

export ORAENV_ASK=NO
. oraenv
export ORAENV_ASK=YES

dbstart $ORACLE_HOME
```

If Oracle doesn't start:

```
sqlplus /nolog
conn sys/oracle as sysdba
shutdown abort
startup
```