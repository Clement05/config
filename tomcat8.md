* This is the steps that I follow to set up my tomcat8 serveur

* Here it is only dealing with tomcat installation

* I have already installed an apache2 server

* I am an debian-jessie

Install Tomcat 8
=====================
[source](https://wolfpaulus.com/journal/software/tomcat-jessie/)

```
#!cmd
$ mkdir -p ~/tmp
$ cd ~/tmp
$ wget https://archive.apache.org/dist/tomcat/tomcat-8/v8.0.32/bin/apache-tomcat-8.0.32.tar.gz
$ tar xvzf ./apache-tomcat-8.0.32.tar.gz
$ rm ./apache-tomcat-8.0.32.tar.gz

$ sudo mkdir -p /usr/share/tomcat8
$ sudo mv ~/tmp/apache-tomcat-8.0.32 /usr/share/tomcat8
```
Then we make a symbolic link to refer to this folder. When you will move to a new version of tomcat, it will be easiest to move up.
```
#!cmd
$ sudo rm -f /usr/share/tomcat
$ sudo ln -s /usr/share/tomcat8/apache-tomcat-8.0.27 /usr/share/tomcat
```
Then put the right to the folder
```
#!cmd
$ sudo chmod +x /usr/share/tomcat/bin/*.sh
```
Then edit the tomcat-users.xml files
```
#!cmd
$ sudo nano /usr/share/tomcat/conf/tomcat-users.xml
```
Add the following rules
[source](http://stackoverflow.com/questions/19325636/403-access-denied-on-tomcat-7-0-42)
```
#!xml
<tomcat-users>
  <role rolename="manager-script"/>
  <role rolename="manager-gui"/>
  <role rolename="manager-jmx"/>
  <role rolename="manager-status"/>
  <user username="tomcat" password="tomcat" roles="manager-gui,manager-status"/>
</tomcat-users>
```
Start Tomcat 8
=====================
```
#!cmd
$ sudo /bin/su - {User} -c /usr/share/tomcat/bin/startup.sh
```
Stop Tomcat 8
=====================
```
#!cmd
$ sudo /bin/su - {USer} -c /usr/share/tomcat/bin/shutdown.sh
```
Make a restart file
=====================
Create tomcat.sh
```
#!cmd
$ cd
$ sudo nano tomcat.sh
```
Past the following content to tomcat.sh
```
#!/bin/bash

### BEGIN INIT INFO
# Provides:        tomcat
# Required-Start:  $network
# Required-Stop:   $network
# Default-Start:   2 3 4 5
# Default-Stop:    0 1 6
# Short-Description: Start/Stop Tomcat server
### END INIT INFO

PATH=/sbin:/bin:/usr/sbin:/usr/bin

start() {
 /bin/su - tomcat -c /usr/share/tomcat/bin/startup.sh
}

stop() {
 /bin/su - tomcat -c /usr/share/tomcat/bin/shutdown.sh 
}

case $1 in
  start|stop) $1;;
  restart) stop; start;;
  *) echo "Run as $0 <start|stop|restart>"; exit 1;;
esac
```
Put exe right to tomcat.sh
```
#!cmd
$ chmod 755 tomcat.sh
```
You can now run
```
#!cmd
$ sh tomcat.sh restart
```
