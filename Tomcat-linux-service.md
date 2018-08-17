Install Tomcat as a service on Linux
======================================

Download Tomcat from the Apache website.  
Unpack in `/opt/apache-tomcat-x.y.z`. E.g. `/opt/apache-tomcat-8.5.6`.  
You'll need a terminal and root access.  


## Create Tomcat user with restricted permissions

This is the user the Tomcat service will run as.

```sh
groupadd tomcat
useradd -s /sbin/nologin -g tomcat -d /opt/apache-tomcat-8.5.6 tomcat
passwd tomcat
```


Set the `tomcat` user as the owner of the $CATALINA_HOME folder.

```sh
chown -R tomcat.tomcat /opt/apache-tomcat-8.5.6
```


## Configure Tomcat to run as a service

### Using init.d

Add `/etc/init.d/tomcat` init script. Notice there are other init scripts in `/etc/init.d/`.  
The script shown below will have a LSB type header to define dependencies and runlevels.  
Some details here: https://wiki.debian.org/LSBInitScripts  
It will start and stop the server as the `tomcat` user, preserving the existing environment variables, by using `su -p -s /bin/sh tomcat ...`.  


```sh
#!/bin/bash

### BEGIN INIT INFO
# Provides:          tomcat
# Required-Start:    $network $remote_fs $syslog
# Required-Stop:     $network $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start Tomcat at boot time
# Description:       Start Tomcat at boot time
### END INIT INFO

export JAVA_HOME=/usr/lib/jvm/jre
export CATALINA_HOME=/opt/apache-tomcat-8.5.6
export JAVA_OPTS="-Xms250m -Xmx1024m"

RETVAL=$?
case $1 in
start)
    if [ -f $CATALINA_HOME/bin/startup.sh ];
    then
        echo $"Starting Tomcat"
        su -p -s /bin/sh tomcat $CATALINA_HOME/bin/startup.sh
    fi
    ;; 
stop)   
    if [ -f $CATALINA_HOME/bin/shutdown.sh ];
    then
        echo $"Stopping Tomcat"
        su -p -s /bin/sh tomcat $CATALINA_HOME/bin/shutdown.sh
    fi
    ;; 
*)
    echo $"Usage: $0 {start|stop}"
    exit 1
    ;;
esac

exit $RETVAL
```

Make the script executable:

```sh
chmod ug+x /etc/init.d/tomcat
```

Configure the system to run the script at boot:

```sh
sudo update-rc.d tomcat defaults     # Debian, Ubuntu
sudo chkconfig --add tomcat          # Red Hat & co.
```

If you want to remove the service

```sh
sudo update-rc.d -f tomcat remove    # Debian, Ubuntu
```

To start/stop the script manually:

```sh
service tomcat [start | stop]
```

Or the old-fashioned way (Ubuntu):

```sh
/etc/init.d/tomcat [start | stop]
```


### Using systemd

Add `/etc/systemd/system/tomcat.service` init script:  

```sh
# Systemd unit file for tomcat
[Unit]
Description=Apache Tomcat Web Application Container
After=syslog.target network.target

[Service]
Type=forking
User=tomcat
Group=tomcat

Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_PID=/opt/apache-tomcat-8.5.6/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/apache-tomcat-8.5.6
Environment=CATALINA_BASE=/opt/apache-tomcat-8.5.6
Environment=CATALINA_OPTS=
Environment="JAVA_OPTS=-Dfile.encoding=UTF-8 -Dnet.sf.ehcache.skipUpdateCheck=true -XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled -XX:+UseParNewGC -Xms2g -Xmx4g"

ExecStart=/opt/apache-tomcat-8.5.6/bin/startup.sh
ExecStop=/bin/kill -15 $MAINPID

[Install]
WantedBy=multi-user.target
```

The script tells the system to run the service as the `tomcat` user with the specified configs.

Reload Systemd in order to discover and load the new Tomcat service file:

```sh
systemctl daemon-reload
```

Enable the service to start at boot:

```sh
systemctl enable tomcat.service
```

To control the service:

```sh
service tomcat [start | stop | restart | status]
```

Or with Systemd directly:

```sh
systemctl [start | stop | restart | status] tomcat
```


## Configure Tomcat with APR native library

For better performance, scalability and SSL usage, especially on production environments, it is recommended to configure Tomcat to run with the APR library.

Docs: https://tomcat.apache.org/tomcat-8.5-doc/apr.html


