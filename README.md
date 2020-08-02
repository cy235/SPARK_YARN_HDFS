# Create a Hadoop Distributed File System ( HDFS) running Spark on YARN  
 In this repo, we introduce how to create a Hadoop cluster with different machines, and run the Apache Spark on YARN to process big data.
 
 First, we create multiple (here we create 4) virtual machines installed with CentOS 8 using VMware, and setup the network adapter as NAT, 
 then we configure the network interface card for each machine
 
```
 vi /etc/sysconfig/network-scripts/ifcfg-ens33
```
```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=29990cdd-1873-4193-a232-5e0fa88072e2
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.226.130
GATEWAY=192.168.226.2
DNS1=192.168.226.2
NETMASK=255.255.255.0
```
