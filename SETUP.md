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
Add to `/home/hduser/.bashrc`
`export HADOOP_INSTALL=/usr/local/hadoop`
`export PATH=$PATH:$HADOOP_INSTALL/bin`

**Reboot and test**
`sudo reboot`
`hadoop version`

## Basic Hadoop configuration
**Configure environment**
`nano /usr/local/hadoop/etc/hadoop/hadoop-env.sh`
Set: `export HADOOP_HEAPSIZE=768`

`nano /usr/local/hadoop/etc/hadoop/yarn-env.sh`
Set: `YARN_HEAPSIZE=768`

**Configuration files**
Edit `core-site.xml`, `mapred-site.xml`, `hdfs-site.xml` (see GIT repo)

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

Go to http://IP_ADDR:50070 to see the Hadoop status page.

## Going multinode 
Give each node a hostname in /etc/hostname (eg. node01, node02 etc.)
Comment the line containing 127.0.1.1 with a # in /etc/hosts

Add hostname of the node which should be secondary namenode to masters configuration file in `/usr/local/hadoop/etc/hadoop/masters` (one per line)
Add hostnames of nodes which should be data/task nodes to the slaves configuration file in `/usr/local/hadoop/etc/hadoop/slaves` (one per line)

- [ ] TODO: generate new SSH key for new node as in the beginning of this document

`sudo reboot`

Make sure `/fs/hadoop/tmp` is empty on all nodes and reformat the datanodes from the master node with `hdfs namenode -format`
Start Hadoop from the master node and check the status of the nodes on http://IP_ADDR:50070