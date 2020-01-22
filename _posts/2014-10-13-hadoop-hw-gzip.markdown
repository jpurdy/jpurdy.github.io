---
layout: post
title: Integrating Hardware Accelerated Gzip Compression into Hadoop
date: 2014-10-13 07:07:07 +0300
description: AHA Gzip Hardware Integration into Hadoop
img: 2014_10_13-1.jpg
fig-caption:
tags: [Hadoop, Gzip, Linux]
---
The following is a set of instruction for integrating HW GZIP compression into a Hadoop DataNode. The compression enabled in this set of instructions is activated on the map phase’s output data, as well as the reduce phase’s output. Compression at the map output’s stage reduces the number of the bytes written to disk and then transferred across to network during the shuffle and sort stage of a job. Compression at the end of the reduce phase reduces the size of the data on the HDFS disk, as well as the time spent writing to the disk.

Sounds great, why wouldn’t one utilize GZIP compression? Why isn’t it enabled out of the box?

Well, the problem with GZIP compression is that it creates a high load on processors. As a result, it’s bypassed in favor of other “lighter” compression algorithms. The trade off is that these other algorithms are plagued by lower resultant compression ratios. (see : [GZIP or Snappy](http://geekmantra.wordpress.com/2013/03/28/compression-in-kafka-gzip-or-snappy/))

GZIP HW Compression provides the best of both worlds; the CPU load needed to compress data is offloaded to a dedicated piece of hardware, and the compression ratio remains high.

So, without further ado, let’s integrate an AHA HW Compression Accelerator into our Hadoop node.

Note: This procedure was performed using Hadoop 2.2 (single node cluster) on Ubuntu 12.04. The HW Compression utilized for this setup was an [AHA372](http://www.aha.com/data-compression/). Also, for the benefit of full disclosure, I’m a software engineer at AHA Products Group (which makes for easy access to GZIP accelerator cards to experiment with).

 1. Install and load the AHA 3xx hardware compression driver.  
  * After installing the AHA 3xx PCIe compression accelerator into an available PCIe slot, download and unpack the AHA3xx driver to a location of your choosing. I’ve created a folder at /usr/local/aha and placed the tarball in that directory
   ````
   $ mkdir -p /usr/local/aha
   $ cp  AHA3xx_ver_3_1_0_20140711.tgz /usr/local/aha/
   $ tar -xvf AHA3xx_ver_3_1_0_20140711.tgz
   $ cd AHA3xx_ver_3_1_0_20140711
   ````
  * Compile the driver
   ```
   $ sudo ./install_driver
   ```

 2. Compile and install the AHA zlib library  
  * Compile the AHA zlib library
   ````
   $ mkdir /usr/local/aha/zlib
   $ cd /usr/local/aha/AHA3xx_ver_3_1_0_20140711/zlib
   $ ./configure -shared -prefix=/usr/local/aha/zlib --hwdeflate
   $ make install
   ````
  * Add the AHA zlib library’s path to the system’s library search path.
   ```
   $ sudo echo "/usr/local/aha/zlib/lib/" > /etc/ld.so.conf.d/aha.conf
   ```
  * Verify that the AHA zlib library’s path is visible.  
   ```
   $ ldconfig
   $ ldconfig -p | grep aha
   libahaz.so.1 (libc6,x86-64) => /usr/local/aha/zlib/lib/libahaz.so.1
   libahaz.so (libc6,x86-64) => /usr/local/aha/zlib/lib/libahaz.so
   ```

 3. Download and unpack the Hadoop source code from one of the [Apache download mirrors](http://www.apache.org/dyn/closer.cgi/hadoop/core/) to a location of your choosing. In this example, the source code was placed at “/home/hadoop/src/”
 
 4. Install the tools required to build the Hadoop native library (if they aren’t already installed).
  * Install the tools that are available via the package manager.
   ```
   $ sudo apt-get install build-essential
   $ sudo apt-get install autoconf automake cmake g++
   $ sudo apt-get install libtool zlib1g-dev pkg-config libssl-dev
   $ sudo apt-get install maven
   ```
  * Manually install Protobuf 2.5 which is not available via the package manager.
   ```
   $ wget https://protobuf.googlecode.com/files/protobuf-2.5.0.tar.gz
   $ tar -xvf protobuf-2.5.0.tar.gz
   $ cd protobuf-2.5.0
   $ ./configure --prefix=/usr
   $ sudo make install
   ```
 5. Compile and load the Hadoop native library to verify that if functions correctly without AHA zlib modifications.
  * Compile the Hadoop native library. Note: Between the time it takes for Maven to download dependencies and compile the project, this step takes several minutes. On my machine, it takes about 50 minutes. Subsequent compilations (after all dependencies have been downloaded) take about 7 minutes.  
   ```
   $ cd /home/hadoop/src/hadop-2.2.0-src/
   $ mvn package -Pdist,native -DskipTests -Dtar
   ```
  * Copy the native library from the source over to your Hadoop installation. (be sure to backup the existing native library)
   ```
   $ cp /home/hadoop/src/hadoop-2.2.0-src/hadoop-dist/target/hadoop-2.2.0/lib/native/* YOUR_HADOOP_LOCATION/lib/native/
   ```
 6. Edit the Hadoop configuration file in **YOUR_HADOOP_LOCATION/etc/mapred-site.xml** to enable map output, and reduce output compression. Add the following lines before the **</configuration>** line.  
   >&lt;property&gt;  
   >&nbsp;&nbsp;&nbsp;&nbsp;&lt;name&gt;mapreduce.map.output.compress&lt;/name&gt;  
   >&nbsp;&nbsp;&nbsp;&nbsp;&lt;value&gt;true&lt;/value&gt;  
   >&lt;/property&gt;  
   >&lt;property&gt;  
   >&nbsp;&nbsp;&nbsp;&nbsp;&lt;name&gt;mapreduce.map.output.compress.codec&lt;/name&gt;
   >&nbsp;&nbsp;&nbsp;&nbsp;&lt;value&gt;org.apache.hadoop.io.compress.GzipCodec&lt;/value&gt;
   >&lt;/property&gt;  
   >&lt;property&gt;  
   >&nbsp;&nbsp;&nbsp;&nbsp;&lt;name&gt;mapreduce.output.fileoutputformat.compress&lt;/name&gt;  
   >&nbsp;&nbsp;&nbsp;&nbsp;&lt;value&gt;true&lt;/value&gt;  
   >&lt;/property&gt;  
   >&lt;property&gt;  
   >&nbsp;&nbsp;&nbsp;&nbsp;&lt;name&gt;mapreduce.output.fileoutputformat.compress.type&lt;/name&gt;  
   >&nbsp;&nbsp;&nbsp;&nbsp;&lt;value&gt;BLOCK&lt;/value&gt;  
   >&lt;/property&gt;  
   >&lt;property&gt;  
   >&nbsp;&nbsp;&nbsp;&nbsp;&lt;name&gt;mapreduce.output.fileoutputformat.compress.codec&lt;/name&gt;
   >&nbsp;&nbsp;&nbsp;&nbsp;&lt;value&gt;org.apache.hadoop.io.compress.GzipCodec&lt;/value&gt;  
   >&lt;/property&gt;  
 7. Run a Hadoop job that has a reduce stage and verify compression is working for both the map output as well as the reduce output.  
  * *Wordcount* is a Hadoop routine that produces intermediate map output data, and thus can be used to test compression. To set this up, first, copy text files from your local file system to the HDFS file system.  		
   * Run wordcount on the dataset. In the statistics that come back from the job, note the number with the prefix **“Map Output Materialized Bytes:"**    
   ```
   $ YOUR_HADOOP_LOCATION/bin/hadoop jar YOUR_HADOOP_LOCATION/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.2.0.jar wordcount /books /booksoutput
   …
   Map Output Materialized Bytes: 479752
   …
   ```
  * Comment out the properties added in step 6, to disable compression. Rerun the wordcount job. Verify that the **“Map Output Materialized Bytes:”** value is larger than in the previous run. This verifies that compression is being utilized on the map-output data.
   ```
   $ bin/hdfs dfs -rmr /booksoutput
   $ YOUR_HADOOP_LOCATION/bin/hadoop jar YOUR_HADOOP_LOCATION/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.2.0.jar wordcount /books /booksoutput
   …
   Map Output Materialized Bytes: 1459156
   …
   ```
  * To verify that compression is being utilized on the reduce-output data, examine the destination directory in HDFS. The resultant data ‘parts’ should be appended with a gz  
   ```
   bin/hdfs dfs -ls /booksoutput
   -rw-r–r– 1 hadoop supergroup 0 2014-10-16 10:08 /bookouput/_SUCCESS
   -rw-r–r– 1 hadoop supergroup 305714 2014-10-16 10:08 /bookouput/part-r-00000.gz
   ```  
  * Turn compression back on by restoring the properties in step 6.
 8. Modify the native library CMake configuration and source code to point the compression library to the AHA HW compression.
  * Modify the Hadoop native library’s CMake file to point to AHA HW compression’s version of zlib.
   ```
   $vim hadoop-common-project/hadoop-common/target/native/CMakeCache.txt
   ```
   change:  
   `ZLIB_LIBRARY:FILEPATH=/lib/x86_64-linux-gnu/libz.so.1`  
   to:  
   `ZLIB_LIBRARY:FILEPATH=/usr/local/aha/zlib/lib/libahaz.so.1`  
   * Modify the native library’s zlib library macro.
   ```
   $ vim hadoop-common-project/hadoop-common/target/native/config.h
   ```  
   change:  
   `#define HADOOP_ZLIB_LIBRARY “libz.so.1”`  
   to:  
   `#define HADOOP_ZLIB_LIBRARY “libahaz.so.1”`  
   * Edit the native library’s decompression c source such that symbols from AHA zlib library are loaded instead of those from the standard zlib library.  
   ```
   $ vim hadoop-common-project/hadoop-common/src/main/native/src/org/apache/hadoop/io/compress/zlib/ZlibDecompressor.c
   ```
   change:  
   ```
   LOAD_DYNAMIC_SYMBOL(dlsym_inflateInit2_, env, libz, “inflateInit2_”);  
   LOAD_DYNAMIC_SYMBOL(dlsym_inflate, env, libz, “inflate”);  
   LOAD_DYNAMIC_SYMBOL(dlsym_inflateSetDictionary, env, libz, “inflateSetDictionary”);  
   LOAD_DYNAMIC_SYMBOL(dlsym_inflateReset, env, libz, “inflateReset”);  
   LOAD_DYNAMIC_SYMBOL(dlsym_inflateEnd, env, libz, “inflateEnd”);  
   ```
   to:
   ```  
   LOAD_DYNAMIC_SYMBOL(dlsym_inflateInit2_, env, libz, **“AHA_inflateInit2_”**);  
   LOAD_DYNAMIC_SYMBOL(dlsym_inflate, env, libz, **“AHA_inflate”**);  
   LOAD_DYNAMIC_SYMBOL(dlsym_inflateSetDictionary, env, libz, **“AHA_inflateSetDictionary”**);  
   LOAD_DYNAMIC_SYMBOL(dlsym_inflateReset, env, libz, **“AHA_inflateReset**”);  
   LOAD_DYNAMIC_SYMBOL(dlsym_inflateEnd, env, libz, **“AHA_inflateEnd”**);  
   ```
   * Edit the native library’s compression c source such that symbols from AHA zlib library are loaded instead of those from the standard zlib library.
   ```
   $ vim hadoop-common-project/hadoop-common/src/main/native/src/org/apache/hadoop/io/compress/zlib/ZlibCompressor.c
   ```
   change:
   ```
   LOAD_DYNAMIC_SYMBOL(dlsym_deflateInit2_, env, libz, “deflateInit2_”);
   LOAD_DYNAMIC_SYMBOL(dlsym_deflate, env, libz, “deflate”);
   LOAD_DYNAMIC_SYMBOL(dlsym_deflateSetDictionary, env, libz, “deflateSetDictionary”);
   LOAD_DYNAMIC_SYMBOL(dlsym_deflateReset, env, libz, “deflateReset”);
   LOAD_DYNAMIC_SYMBOL(dlsym_deflateEnd, env, libz, “deflateEnd”);
   ```
   to:
   ```
   LOAD_DYNAMIC_SYMBOL(dlsym_deflateInit2_, env, libz, “AHA_deflateInit2_”);
   LOAD_DYNAMIC_SYMBOL(dlsym_deflate, env, libz, “AHA_deflate”);
   LOAD_DYNAMIC_SYMBOL(dlsym_deflateSetDictionary, env, libz, “AHA_deflateSetDictionary”);
   LOAD_DYNAMIC_SYMBOL(dlsym_deflateReset, env, libz, “AHA_deflateReset”);
   LOAD_DYNAMIC_SYMBOL(dlsym_deflateEnd, env, libz, “AHA_deflateEnd”);
   ```
 9. Compile and install the newly modified Hadoop native library.
  * Compile the Hadoop native library. Note: As before, compilation takes several minutes. On my machine, it took about seven minutes.
  ```
  $ cd /home/hadoop/src/hadop-2.2.0-src/
  $ mvn package -Pdist,native -DskipTests -Dtar
  ```
  * Copy the native library from the source over to your Hadoop installation.
  ```
  $ cp /home/hadoop/src/hadoop-2.2.0-src/hadoop-dist/target/hadoop-2.2.0/lib/native/* YOUR_HADOOP_LOCATION/lib/native/
  ```
 10. Rerun the wordcount job from step 7 (making sure the compression properties are enabled in mapred-site.xml) to verify everything is working as expected. Your Hadoop node is now using HW compression to deflate intermediate map output data.
