# Create a Hadoop Distributed File System ( HDFS) running Spark on YARN  
 In this repo, we introduce how to create a Hadoop cluster with different machines, and run the Apache Spark on YARN to process big data.
 
 First, we create multiple (here we create 4) virtual machines installed with CentOS 8 using VMware, and setup the network adapter as NAT, 
 then we configure the network interface card for each machine
 
"
 vi /etc/sysconfig/network-scripts/ifcfg-ens33
"
