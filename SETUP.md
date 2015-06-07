# How to: install & configure native Hadoop on RPi2
This guide mostly contains shell commands and tasks in order to succesfully install and configure a basic Hadoop 2.6.0 YARN on a cluster of Raspberry Pi 2 Model B boards.

## Prerequisites for native Hadoop
`apt-get install libssl-dev libsnappy-dev`

## Basic Hadoop installation
**Create a Hadoop user**  
`addgroup hadoop`  
`adduser --ingroup hadoop hduser`  
`adduser hduser sudo`  


**Setup SSH for Hadoop**  
`su - hduser`  
`ssh-keygen -t rsa -P ""`  
`cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`  
`chmod 700 ~/.ssh`  
`chmod 600 ~/.ssh/authorized_keys`  

**Unpack Hadoop**  
`sudo tar vxzf hadoop-2.6.0.tar.gz -C /usr/local`  
`cd /usr/local`  
`sudo mv hadoop-2.6.0 hadoop`  
`sudo chown -R hduser:hadoop hadoop`  

**Add Hadoop environment variables**  
Add the following lines to `/home/hduser/.bashrc`  
`export HADOOP_INSTALL=/usr/local/hadoop`  
`export PATH=$PATH:$HADOOP_INSTALL/bin`  

**Reboot and test**  
`sudo reboot`  
`hadoop version`  

## Basic Hadoop configuration
The basic Hadoop configuration should happen on each node in the cluster. Preconfigured configuration and environment files are supplied in the configuration folder in the repository.

**Configure environment**  
`nano /usr/local/hadoop/etc/hadoop/hadoop-env.sh`  
Set: `export HADOOP_HEAPSIZE=768`  

`nano /usr/local/hadoop/etc/hadoop/yarn-env.sh`  
Set: `YARN_HEAPSIZE=768`  

**Configuration files**
Edit `core-site.xml`, `mapred-site.xml`, `hdfs-site.xml` and `yarn-site.xml` (see configuartion folder in the repository for preconfigured files).

**Basic folder setup**  
Create tmp folder  
`sudo mkdir -p /fs/hadoop/tmp`  
`sudo chown hduser:hadoop /fs/hadoop/tmp`  
`sudo chmod 750 /fs/hadoop/tmp/`  

Format working directory  
`/usr/local/hadoop/bin/hadoop namenode -format`  
## Starting Hadoop
`/usr/local/hadoop/sbin/start-dfs.sh`  
`/usr/local/hadoop/sbin/start-yarn.sh`  
`/usr/local/hadoop/sbin/mr-jobhistory-daemon.sh start historyserver`  

Go to http://IP_ADDR:50070 to see the Hadoop status page.  

## Going multinode 
Give each node a hostname in `/etc/hostname` (eg. node01, node02 etc.)  
Comment the line containing 127.0.1.1 with a # in `/etc/hosts`  

Add the hostname of the node which should be secondary namenode to masters configuration file in `/usr/local/hadoop/etc/hadoop/masters` on the master node (one per line)  
Add the hostnames of nodes which should be data/task nodes to the slaves configuration file in `/usr/local/hadoop/etc/hadoop/slaves` on the master node (one per line)  

- [ ] TODO: generate new SSH key for new node as in the beginning of this document  

Reboot the nodes: `sudo reboot`  

Make sure `/fs/hadoop/tmp` is empty on all nodes and reformat the datanodes from the master node with `hdfs namenode -format`  
Now start Hadoop from the master node and check the status of the nodes on http://IP_ADDR:50070  

## Dynamically adding slave nodes
Make sure the new node(s) are configured the same as the other nodes (up until the namenode format).  
First, add the new nodes hostname to the slaves file on the master node. Then start the datanode by running the following command on the new node: `/usr/local/hadoop/sbin/hadoop-daemon.sh start datanode`  

Check if it's correctly added to the cluster on the Hadoop status page.  

After that start the Yarn nodemanager by executing: `/usr/local/hadoop/sbin/yarn-daemon.sh start nodemanager`  
