Tomcat log rotation
========================

Small tutorial on how to configure rotating logs for Tomcat.


Catalina.out
------------------

The rotation of `catalina.out` cannot be configured from a web application or the Tomcat server directly.  
The logs written to it are simple redirects of std.out and std.err.  
It is possible, however, to limit the output added to `catalina.out` and use an external tool to rotate it.  

Steps to take are as follows.

### Disable writing the default tomcat logs to std.out

From the docs: "The default logging configuration in Apache Tomcat writes the same messages to the console and to a log file. This is great when using Tomcat for development, but usually is not needed in production."

Open `conf/logging.properties` and remove `java.util.logging.ConsoleHandler`.  
The handlers should look similar to:  

    handlers = 1catalina.org.apache.juli.AsyncFileHandler, 2localhost.org.apache.juli.AsyncFileHandler, 3manager.org.apache.juli.AsyncFileHandler, 4host-manager.org.apache.juli.AsyncFileHandler
    .handlers = 1catalina.org.apache.juli.AsyncFileHandler


### Configure the `logrotate` utility on Linux to rotate `catalina.out`

Logrotate is installed by default on most Linux distros.  
It should be configured to run on a daily cron job.  

Check in a terminal that it is listed

    ls /etc/cron.daily/


Create the config file `/etc/logrotate.d/tomcat` with the contents:

    /var/log/tomcat/catalina.out {
        copytruncate   
        daily   
        rotate 30   
        compress   
        missingok   
        size 50M  
    }

Make sure the path `/var/log/tomcat/catalina.out` points to your Tomcat's actual catalina.out file, otherwise adjust it.  
The config above keeps at maximum 30 files of 50 MB each and will compress them.  


Other logs
------------------

Tomcat's default logs are: catalina.log, host-manager.log, localhost.log, localhost_access_log.txt, manager.log.  
They are rotated daily by default.  

If size based config is desirable, update the `conf/logging.properties` file by replacing the `org.apache.juli.AsyncFileHandler` or `org.apache.juli.FileHandler` with `java.util.logging.FileHandler`.  

Sample config:

    handlers = 1catalina.java.util.logging.FileHandler, ...
    .handlers = 1catalina.java.util.logging.FileHandler, ...

    1catalina.java.util.logging.FileHandler.level = FINE
    1catalina.java.util.logging.FileHandler.pattern = ${catalina.base}/logs/catalina.%g.log
    1catalina.java.util.logging.FileHandler.limit = 50000000  (aprox 50 MB)
    1catalina.java.util.logging.FileHandler.count = 5
    1catalina.java.util.logging.FileHandler.formatter = java.util.logging.SimpleFormatter


Example `logging.properties` file is attached.


