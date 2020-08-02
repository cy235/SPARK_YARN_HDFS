# Create a Hadoop Distributed File System ( HDFS) with Spark running on YARN  
 In this repo, we introduce how to create a Hadoop cluster with different nodes, and run the Apache Spark on YARN to process big data.
 
 First, we need to create multiple (here we create 4) virtual machines installed with CentOS 8 using VMware, and setup the network adapter as NAT.
 
 ## Network interface card setting
We configure the network interface card for each node in the following
 
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
change `BOOTPROTO=dhcp` to `BOOTPROTO=static`, add your own `IPADDR`, `GATEWAY`, `DNS1`, `NETMASK`, and set different `IPADDR` for different nodes. In this repo, the ip of four nodes are set as `192.168.226.130`, `192.168.226.131`, `192.168.226.132`, `192.168.226.133` respectively. Then we restart the network.
```
service network restart
```

## Change the host name and map ip 
We change the host name for each node
```
vi /etc/hostname
```
In this repo, we name the nodes as `master`, `slave1`, `slave2`, `slave3`, respectively. Then map the ip of each node to its associated name

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
Since the file `vi /etc/hosts` should be identical in each node, we can create it on master node and copy it to different slave nodes

```
scp /etc/hosts  root@slave1:/etc
scp /etc/hosts  root@slave2:/etc
scp /etc/hosts  root@slave3:/etc
```

You can find each time we copy the file from the master to the slaves, we need to enter the password, while the Hadoop cluster requires that master can login each slave with out SSH key. In the next, we will configure that the master can login slaves without SSH key.

## Login slaves without SSH key
Now, in the master node, we create a ssh keypair `id_dsa` and `id_dsa.pub` 
```
ssh-keygen  -t dsa  -P  '' -f  ~/.ssh/id_dsa
```
Then, we rename the public key `id_dsa.pub` as `authorized_keys`
```
cat  ~/.ssh/id_dsa.pub >>  ~/.ssh/authorized_keys
```
Also, don't forget to change mode of this public key by executing the following script.

```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
Finally, we copy this public key into each slaves
```
scp -p ~/.ssh/authorized_keys  root@slave1:~/.ssh/
scp -p ~/.ssh/authorized_keys  root@slave2:~/.ssh/
scp -p ~/.ssh/authorized_keys  root@slave3:~/.ssh/
```

Further, the firewall of all nodes need to be shutdown or disabled

```
systemctl status firewalld
systemctl stop firewalld
systemctl disable firewalld

```

## Install java virtual machine
In the home/root directory, download `jdk-8u161-linux-x64.tar.gz` from ORACLE. Upload the `jdk-8u161-linux-x64.tar.gz` to the home/root directory of slave1,slave2 and slave3 (here we use the tool `FileZilla` to upload the file). unzip the file
```
tar -xvf jdk-8u161-linux-x64.tar.gz
```
and configure the environment
```
vi .bash_profile
export JAVA_HOME=/usr/local/lib/jdk1.8.0_161
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin

export PATH
```
```
source ~/.bash_profile
```

## HDFS configuration
In the home/root directory, download and unzip `hadoop-3.2.1.tar.gz`, create a `bigdata` folder, 
```
mkdir bigdata
cd bigdata
```
then unzip the file `hadoop-3.2.1.tar.gz` into the `bigdata` folder.
Now we configure `core-site.xml`

```
vi /root/bigdata/hadoop-3.2.1/etc/hadoop/core-site.xml
```
```
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://master:9000</value>
                <description>HDFS basic path</description>
        </property>
</configuration>
```
where `hdfs://master:9000/` is the HDFS basic path, all the data will be stored under this path. 

Also, we create required files for namenode and datanode.

```
mkdir -p /root/bigdata/dfs/name
mkdir -p /root/bigdata/dfs/data
```
and configure `hdfs-site.xml`

```
vi /root/bigdata/hadoop-3.2.1/etc/hadoop/hdfs-site.xml
```
```
<configuration>
        <property>
                <name>dfs.replication</name>
                <value>2</value>
                <description>the number of backup blocks, which is not larger than number of DataNodes</description>
        </property>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>/root/bigdata/dfs/name</value>
                <description>the data storage place for NameNodes</description>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>/root/bigdata/dfs/data</value>
                <description>the data storage place for DataNodes</description>
        </property>
</configuration>
```
Then, we configure the hadoop environment

```
vi /root/bigdata/hadoop-3.2.1/etc/hadoop/hadoop-env.sh
```
```
export JAVA_HOME=/usr/local/lib/jdk1.8.0_161
export HDFS_DATANODE_USER=root
export HDFS_NAMENODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
```
```
vi  /root/bigdata/hadoop-3.2.1/etc/hadoop/workers
```
and add the name of all workers/slaves
```
slave1
slave2
slave3
```
Finally, we copy the folder `bigdata` into each slaves
```
scp -r /root/bigdata root@slave1:/root
scp -r /root/bigdata root@slave2:/root
scp -r /root/bigdata root@slave3:/root
```
In the master node, configure `.bash_profile`
```
vi /root/.bash_profile
```
```
export HADOOP_HOME=/root/bigdata/hadoop-3.2.1
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

export PATH
```
```
source /root/.bash_profile
```

## Start HDFS
Before we start HDFS, we need to format the name node by executing the following command in the master node
```
hdfs namenode -format
```
WARNING: format ONLY ONE time (do not do it multiple times), otherwise the namenodes can not successfully start; if you encounter the errors, you can clean up the `data` and `name` folders of all the nodes and format in the master again.

Now, we can start the HDFS
```
start-dfs.sh
```

You can verify whether your HDFS successfully start by executing `jps` (Java Virtual Machine Process Status Tool) in each node,
```
[root@master hadoop]# jps
2179 Jps
2055 SecondaryNameNode
1769 NameNode
```
```
[root@slave1 hadoop]# jps
1698 Jps
1635 DataNode
```
namenode and datanode are running in the master node and slave nodes, respectively. You can also verify it by visiting http://master:9870 or  http://192.168.226.130:9870
![image](https://github.com/cy235/SPARK_YARN_HDFS/blob/master/data_nodes.png)

## Big data practice- create a folder in the HDFS
```
hadoop fs -mkdir hdfs://master:9000/user
```

## generate a big data file
Create a test file to create big data
```
vi test.sh
```
```
#!/bin/bash

for ((i=0;i<=$1;i++))
do
        echo "I love Bigdata, I love Hadoop">>big_file.txt
done
```
Create a big data file of 10000000 lines
```
bash test.sh 10000000
```
Check the file size
```
ll -h
```

Upload the big_file.txt
```
hadoop fs -put big_file.txt /user     
```

You can check this uploaded file on dashboard [image](https://github.com/cy235/SPARK_YARN_HDFS/blob/master/big_data_file.png)




## Configuration of YARN
Now, we configure the hadoop resourcemanager YARN. In the master node, 
```
mkdir -p /root/bigdata/yarn/local-dir
```
```
vi /root/bigdata/hadoop-3.2.1/etc/hadoop/yarn-site.xml
```

```
<configuration>
        <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>master</value>
                <description>the machine where the Resourcemanager is installed</description>
        </property>
        <property>
                <name>yarn.resourcemanager.address</name>
                <value>master:8032</value>
                <description>resourcemanager listen to this port</description>
        </property>
        <property>
                <name>yarn.nodemanager.local-dirs</name>
                <value>/root/bigdata/yarn/local-dir</value>
                <description>the data storage place for nodeManager</description>
        </property>

        <property>
                <name>yarn.nodemanager.resource.memory-mb</name>
                <value>1630</value>
                <description>the memory size the nodemanager manage</description>
        </property>

        <property>
                <name>yarn.nodemanager.resource.cpu-vcores</name>
                <value>2</value>
                <description>the number of CPUs the nodemanager manage</description>
        </property>

</configuration>
```
Then, copy this configurations into slaves

```
scp /root/bigdata/hadoop-3.2.1/etc/hadoop/yarn-site.xml root@slave1:~/bigdata/hadoop-3.2.1/etc/hadoop/
scp /root/bigdata/hadoop-3.2.1/etc/hadoop/yarn-site.xml root@slave2:~/bigdata/hadoop-3.2.1/etc/hadoop/
scp /root/bigdata/hadoop-3.2.1/etc/hadoop/yarn-site.xml root@slave3:~/bigdata/hadoop-3.2.1/etc/hadoop/

scp -r /root/bigdata/yarn/local-dir root@slave1:/root/bigdata/yarn
scp -r /root/bigdata/yarn/local-dir root@slave2:/root/bigdata/yarn
scp -r /root/bigdata/yarn/local-dir root@slave3:/root/bigdata/yarn
```
In `hadoop-env.sh`,
```
vi /root/bigdata/hadoop-3.2.1/etc/hadoop/hadoop-env.sh
```
add the following
```
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
```

Now, start YARN
```
start-yarn.sh
```
you can visit it via http://master:8088 or http://192.168.226.130:8088/ 

![image](https://github.com/cy235/SPARK_YARN_HDFS/blob/master/hadoop_nodes.png)


## Configuration of Apache Spark
Download and unzip file spark-3.0.0-preview2-bin-hadoop3.2.tgz under the fold `/root/bigdata/`.

Add the slave names
```
cd /root/bigdata/spark-3.0.0-preview2-bin-hadoop3.2/conf
cp slaves.template slaves
vi slaves
```
```
slave1
slave2
slave3
```
Set Spark environmnt 
```
cp spark-env.sh.template spark-env.sh
vi spark-evc.sh
```
add the following
```
export JAVA_HOME=/usr/local/lib/jdk1.8.0_161
```

Copy Spark configuration to slaves.
```
scp -r /root/bigdata/spark-3.0.0-preview2-bin-hadoop3.2 root@slave1:/root/bigdata
scp -r /root/bigdata/spark-3.0.0-preview2-bin-hadoop3.2 root@slave2:/root/bigdata
scp -r /root/bigdata/spark-3.0.0-preview2-bin-hadoop3.2 root@slave3:/root/bigdata
```

Then, modify the `.bash_profile`
```
vi .bash_profile
```
```
export JAVA_HOME=/usr/local/lib/jdk1.8.0_161
export HADOOP_HOME=/root/bigdata/hadoop-3.2.1
export SPARK_HOME=/root/bigdata/spark-3.0.0-preview2-bin-hadoop3.2
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$SPARK_HOME/bin

export PATH
```
```
source .bash_profile
```
## Spark applys resource from YARN
```
export YARN_CONF_DIR=/root/bigdata/hadoop-3.2.1/etc/hadoop
spark-shell  --master yarn  --num-executors 2 --executor-memory 1048M --executor-cores 1
```
Now, you can enter the Spark
scala>  val data=sc.textFile("hdfs://master:9000/user/big_file.txt")
scala> data.count()

