---
layout: post
title: Setting up an Environment to Build the Hadoop Native Library
date: 2014-10-25 07:07:07 +0300
description: Hadoop Native Library Build Environment Setup
img: 2014_10_25-1.jpg
fig-caption:
tags: [Hadoop, Linux]
---
The following are my notes on how to setup a freshly installed Centos 6.5 machine to compile the Hadoop native library. My reason for doing this was to add a modified version of the native library which utilized hardware compression. In order to accomplish that, I needed to recompile the native library. In a previous post, [“Integrating Hardware Accelerated Gzip Compression into Hadoop”](https://swiftdeveloper.ninja/hadoop-hw-gzip/), I described the process for modifying the Hadoop native library to use a [hardware compression accelerator](http://www.aha.com/data-compression/).  

In that [previous post](https://swiftdeveloper.ninja/hadoop-hw-gzip/) I utilized Hadoop 2.2. In this post I’m using Hadoop 2.4 because that’s the version of Hadoop that matches what is utilized by Ambari 1.6.0, the management tool I’ve chosen to administer my current cluster. Additionally, I was using Ubuntu 12.04 in the previous post, whereas in this post I’m using Centos 6.5. Ubuntu is typically my default Linux flavor, but because it is [not compatible with Ambari](https://cwiki.apache.org/confluence/display/AMBARI/Install+Ambari+1.6.0+from+Public+Repositories), I installed Centos 6.5.

*All of these steps assume you are logged in as “root”. If you aren’t, prefix the commands below with “sudo”.*

1. Install the development tools required to build the native library via the package manager. (Note: Some of these may already be installed)
```
# yum -y update
# yum -y groupinstall “Development Tools”
# yum -y install zlib-devel
# yum -y install xz-devel
# yum -y install cmake
# yum -y install openssl-devel
```
2. Optionally install these packages. (Note: These steps aren’t required to build Hadoop’s native library, I just find myself having to install them eventually)

    * Install the kernel development package, vi, and git using the package manager
    ```
    # yum install kernel-devel
    # yum install vi
    # yum install git
    # yum install lynx
    ```

    * Install hexedit manually
    ```
    # wget http://pkgs.repoforge.org/hexedit/hexedit-1.2.10-1.el6.rf.x86_64.rpm
    # rpm -Uvh hexedit-1.2.10-1.el6.rf.x86_64.rpm
    ```
    * If you have an AHA Hardware Compression Accelerator installed, install/load the driver, and build the hardware accelerated zlib variant. See Steps 1-2 of this [post](https://swiftdeveloper.ninja/hadoop-hw-gzip/).
3. Install Oracle Java JDK 1.7  
You might already have this JDK installed, otherwise download Oracle Java JDK 7.1; The file should be named something similar to “jdk-7u71-linux-x64.tar.gz”. On my machine which is also a datanode (installed via Ambari) the JDK was located in /usr/jdk64/jdk1.7.0_67/.  

    * **Without** the JDK already installed do this:
     ```
     # mv jdk-7u71-linux-x64.tar.gz /opt/
     # tar -xvf jdk-7u71-linux-x64.tar.gz
     # echo -e 'export JAVA_HOME=/opt/jdk1.7.0_71\nexport PATH=${JAVA_HOME}/bin:${PATH}' &gt; /etc/profile.d/oraclejava.sh
    ```
 
    * **With** the JDK already installed to /usr/jdk64/jdk1.7.0_67/ do this:
    ```
    # echo -e 'export JAVA_HOME=/usr/jdk64/jdk1.7.0_67\nexport PATH=${JAVA_HOME}/bin:${PATH}' &gt; /etc/profile.d/oraclejava.sh
    ```
4. Install Maven 3.0.5
```
# cd /usr/local/
# wget http://www.us.apache.org/dist/maven/maven-3/3.0.5/binaries/apache-maven-3.0.5-bin.tar.gz
# tar -xvf apache-maven-3.0.5-bin.tar.gz
# ln -s apache-maven-3.0.5 maven
# echo -e 'export M2_HOME=/usr/local/maven\nexport PATH=${M2_HOME}/bin:${PATH}' &gt; /etc/profile.d/maven.sh
```
5. Install Ant 1.9.4
```
# cd /usr/local/
# wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.4-bin.tar.gz
# tar -xvf apache-ant-1.9.4-bin.tar.gz
# ln -s /usr/local/apache-ant-1.9.4 ant
# echo -e 'export ANT_HOME=/usr/local/ant\nexport PATH=${ANT_HOME}/bin:${PATH}' &gt; /etc/profile.d/ant.sh
```
6. Install Protobuf 2.5.0
```
# wget https://github.com/google/protobuf/releases/download/v2.5.0/protobuf-2.5.0.tar.gz
# tar -xvf protobuf-2.5.0.tar.gz
# cd protobuf-2.5.0
# ./configure
# make install
```
7. Download Hadoop 2.4.0 source
```
# mkdir /usr/local/hadoop_src
# cd /usr/local/hadoop_src
# wget --no-check-certificate https://archive.apache.org/dist/hadoop/core/hadoop-2.4.0/hadoop-2.4.0-src.tar.gz
# tar -xvf hadoop-2.4.0-src.tar.gz
```
8. Compile the Hadoop native library. This step takes a while. On my machine it takes around 50 minutes to complete. (Note: Make any changes to the native library, such as adding support for hardware compression, before this step).
```
# cd /usr/local/hadoop_src/hadoop-2.4.0-src
# mvn package -Pdist,native -DskipTests -Dtar
```
     

 9. Backup Hadoop’s native library, and then replace it with the version compiled in step 8 (Note: The path to the hadoop libraries below are the default location that Ambari installs to. Your installation path will vary depending on how you went about your Hadoop installation)
```
# cd /usr/local/hadoop_src/hadoop-2.4.0-src/hadoop-dist/target/hadoop-2.4.0/lib/
# tar -cvf native_NEW.tar.gz native
# cp native_NEW.tar.gz /usr/lib/hadoop/lib/
# cd /usr/lib/hadoop/lib/
# tar -cvf native_ORIGINAL.tar.gz native
# rm -rf native
# tar -xvf native_NEW.tar.gz
```  
That’s it!
