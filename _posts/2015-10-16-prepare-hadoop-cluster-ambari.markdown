---
layout: post
title: Preparing a Hadoop Cluster Node for Ambari
date: 2015-10-16 07:07:07 +0300
description: Ambari with Hadoop Cluster Setup Notes
img: 2015_10_216-1.jpg
fig-caption:
tags: [Hadoop, Linux]
---
The following are my notes on preparing a freshly installed Centos 6.5 for inclusion into a Hadoop cluster via Amabari.

The goal here is to get a system prepared so that when Ambari does its preliminary check before assimilation, there are no warnings.

![Ambari Success]({{site.baseurl}}/assets/img/2016_10_216-2.png)



All of these steps assume you are logged in as “root”. If you aren’t, prefix the commands below with “sudo”.

1. Turn on the ssh server.  
   ```
   # service sshd start
   # chkconfig sshd on
   ```


1. Update the system  
   ```
   # yum update
   ```

1. Turn off iptables  
   ```
   # service iptables stop  
   # chkconfig iptables off
   ```


1. Turn on Network Time Protocol daemon (ntpd)  
   ```
   # service ntpd start
   # chkconfig ntpd on
   ```

1. Turn off SELinux  
   ```
   # setenforce 0
   ```  
   In **/etc/selinux/config** set the *SELINUX* variable to “disabled”
      **SELINUX=disabled**


1. Turn off Transparent Huge Pages (THP)  
In **/etc/grub.conf**, append **“transparent_hugepage=never”** to the boot options, like so:  

   ```
   kernel /vmlinuz-2.6.32-573.7.1.el6.x86_64 ro root=/dev/mapper/vg_hdp1-lv_root rd_NO_LUKS  KEYBOARDTYPE=pc KEYTABLE=us LANG=en_US.UTF-8 rd_LVM_LV=vg_hdp1/lv_root rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto rd_LVM_LV=vg_hdp1/lv_swap rd_NO_DM rhgb quiet transparent_hugepage=never
   ```
1. Disable IPV6  
   In **/etc/sysctl.conf** append the following lines (or create it with the following lines, if it doesn't already exist):

   ```
   net.ipv6.conf.all.disable_ipv6 = 1
   net.ipv6.conf.default.disable_ipv6 = 1
   net.ipv6.conf.lo.disable_ipv6 = 1
   ```

   Then load the changes:  

   ```
   # sysctl -p
   ```
