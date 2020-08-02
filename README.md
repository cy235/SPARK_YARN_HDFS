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

## Change the host name and map ip 
We change the host name for each machine 
```
vi /etc/hostname
```
In this repo, we name the machines as `master`, `slave1`, `slave2`, `slave3`, respectively. Then map the ip of each machine to its associated name

```
vi /etc/hosts
```
```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.226.130 master
192.168.226.131 slave1
192.168.226.132 slave2
192.168.226.133 slave3
```
Since the file `vi /etc/hosts` should be identical in each machine, we can create it on master machine and copy it to different slave machines

```
scp /etc/hosts  root@slave1:/etc
scp /etc/hosts  root@slave2:/etc
scp /etc/hosts  root@slave3:/etc
```

You can find each time we copy the file from the master to the slaves, we need to enter the password, while the Hadoop cluster requires that master can login each slave with out SSH key. In the next, we will configure that the master can login slaves without SSH key.

## Login slaves without SSH key
Now, in the master machine, we create a ssh keypair `id_dsa` and `id_dsa.pub` 
```
ssh-keygen  -t dsa  -P  '' -f  ~/.ssh/id_dsa
```
Then, we rename the public key `id_dsa.pub` as `authorized_keys`
```
cat  ~/.ssh/id_dsa.pub >>  ~/.ssh/authorized_keys
```
Finally, we copy this public key into each slaves
```
scp ~/.ssh/authorized_keys  root@slave1:~/.ssh/
scp ~/.ssh/authorized_keys  root@slave2:~/.ssh/
scp ~/.ssh/authorized_keys  root@slave3:~/.ssh/
```
Also, don't forget to change mode of this public key by executing the following script on each machine.

```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
Further, the firewall of all machines need to be shutdown or disabled

```
systemctl status firewalld
systemctl stop firewalld
systemctl disable firewalld

```

## Install java virtual machine
Download `jdk-8u161-linux-x64.tar.gz` from ORACLE. Upload the `jdk-8u161-linux-x64.tar.gz` to slave1,slave2 and slave3 (here we use the tool `FileZilla` to upload the file). unzip the file
```
tar -xvf jdk-8u161-linux-x64.tar.gz
```
and configure the environment
```
cd ~
```
```
vi .bash_profile
export JAVA_HOME=/usr/local/lib/jdk1.8.0_161
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin

export PATH
```
```
source ~/.bash_profile
```

