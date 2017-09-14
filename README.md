# JVisualVM tomcat EC2
The objective is to connect a Java profiler like JVisualVm or JConsole to a remote tomcat running on AWS EC2 (Amazon Linux AMI).



## Environment information
 * Tomcat: tomcat 8.0.x (through yum)
 * Instance type: EC2 t2.micro
 * OS: Red Hat 4.8.3-9

After reading different tutorials and trying different approaches which didn’t work, here is what it worked for me:

 
 

## Software already installed
The machine already has the following software installed from my previous tutorials:
 
 1. [Tomcat 8.x](https://youtu.be/lCex88J-fIo): This video explains how to install tomcat in an AWS EC2 instance.
 * Port: 8080
 * User: admin
 * Password: adminadmin
 
 
 
 ## Configuration steps
 **1. Find the environment file for tomcat**
 * This path is correct if you installed tomcat8 through yum command. 
```
whereis tomcat8
vim /etc/tomcat8/tomcat8.conf
```
* If not you should create the setenv.sh file if you don’t have a set environment (setenv.sh) file yet. In your tomcat folder/bin: create a sentenv.sh file and add execution rights.
```
cd /var/local/apache-tomcat-8.0.x/bin
touch setenv.sh
chmod +x setenv.sh
```


**2. Set the JMX configuration**
In the tomcat8.conf or setenv.sh file you need to add the following configuration to the CATALINA_OPTS variable. In my configuration I have chosen randomly the 7091 for my jmx port.


* For the tomcat8.conf file:
```
CATALINA_OPTS="-Dcom.sun.management.jmxremote.port=7091 -Dcom.sun.management.jmxremote.rmi.port=7091 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=127.0.0.1 "
```

* For the setenv.sh file:
```
CATALINA_OPTS="-Dcom.sun.management.jmxremote.port=7091 -Dcom.sun.management.jmxremote.rmi.port=7091 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=127.0.0.1 $CATALINA_OPTS"
```

* Note: Use CATALINA_OPTS instead of JAVA_OPTS. If you use JAVA_OPTS you might get a java.net.BindException: Address already in use.
This is because with this JAVA_OPTS option will try to start a jmx server when you start and when you shutdown tomcat. Nevertheless, if you use CATALINA_OPTS will only run when you start tomcat.


**3. Restart tomcat**
```
service tomcat8 restart
```


**4. Find the RMI and JMX ports**
When you start tomcat in the server, Java will open a TCP port for RMI calls. This port by default is 1099 but every time you start a new Java process it will open randomly a different port. This port is important because it will allow us to connect to the JVM in the server, and later on we will need to redirect it via SSH tunnel.
```
sudo netstat -lp | grep java
```
![jmx](http://corporacionkristalia.com/jvisualvm-sources/1-jmx.png)
As you can see the jmx port is 7091.



