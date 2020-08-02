# Create a Hadoop Distributed File System ( HDFS) running Spark on YARN  
 In this repo, we introduce how to create a Hadoop cluster with different machines, and run the Apache Spark on YARN to process big data.
 
 First, we need to create multiple (here we create 4) virtual machines installed with CentOS 8 using VMware, and setup the network adapter as NAT.
 
 ## Network interface card setting
We configure the network interface card for each machine in the following
 
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
change `BOOTPROTO=dhcp` to `BOOTPROTO=static`, add your own `IPADDR`, `GATEWAY`, `DNS1`, `NETMASK`, and set different `IPADDR` for different machines. In this repo, the ip of four machines are set as `192.168.226.130`, `192.168.226.131`, `192.168.226.132`, `192.168.226.133` respectively.  Restart the network.
```
service network restart
```

## Change the host name
We change the host name for each machine 
``
vi /etc/hostname
``
