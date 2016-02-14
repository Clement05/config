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
$ wget http://www.us.apache.org/dist/tomcat/tomcat-8/v8.0.27/bin/apache-tomcat-8.0.27.tar.gz
$ tar xvzf ./apache-tomcat-8.0.27.tar.gz
$ rm ./apache-tomcat-8.0.27.tar.gz

$ sudo mkdir -p /usr/share/tomcat8
$ sudo mv ~/tmp/apache-tomcat-8.0.27 /usr/share/tomcat8

```


