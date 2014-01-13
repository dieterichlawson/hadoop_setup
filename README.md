###Step 0 - Requirements and Directories


###Step 1 - Download and Setup
Download Cloudera's Hadoop distribution, untar it, and set up a link to the directory so things are a bit cleaner.

In `$HDP_DIR`:

```
> wget http://archive.cloudera.com/cdh4/cdh/4/hadoop-2.0.0-cdh4.0.0.tar.gz
> tar -xvzf hadoop-2.0.0-cdh4.0.0.tar.gz
> ln -sf hadoop-2.0.0-cdh4.0.0 hadoop
> rm hadoop-2.0.0-cdh4.0.0.tar.gz
```
We also need to create 2 directories that will hold the data for the namenode's filesystem and the datanode's filesystem. I chose to put these as siblings to my hadoop directory as follows.

In `$HDP_DIR`:

```
> mkdir -p yarn_data/hdfs/namenode
> mkdir yarn_data/hdfs/datanode
```

After this your `$HDP_DIR` should look something like this.

```
> ls -la
total 8
drwxr-xr-x   4 dlaw  wheel  136 Jan 13 14:54 .
drwxrwxrwt  21 root  wheel  714 Jan 13 14:49 ..
lrwxr-xr-x   1 dlaw  wheel   21 Jan 13 14:51 hadoop -> hadoop-2.0.0-cdh4.0.0
drwxr-xr-x  12 dlaw  wheel  408 Jun  4  2012 hadoop-2.0.0-cdh4.0.0
drwxr-xr-x   3 dlaw  wheel  102 Jan 13 15:05 yarn_data
```
Add a line to your `.bash_profile` to export a `YARN_HOME` environment variable, and don't forget to source it afterwards.

```
> echo 'export YARN_HOME=$HDP_DIR/hadoop' >> ~/.bash_profile
> source ~/.bash_profile
```

mine looks like this:

```
export YARN_HOME=$HOME/dev/cs246/hadoop
```

###Step 2 - Edit Config Files

Now we have to edit a few XML config files and export a few more environment variables to make sure Hadoop knows where everything is.

####2.1 core-site.xml

Edit `$YARN_HOME/etc/hadoop/core-site.xml` so that it looks like this:

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://localhost:9000</value>
  </property>
</configuration>
```

####2.2 hdfs-site.xml

Edit `$YARN_HOME/etc/hadoop/hdfs-site.xml` so that it looks like this:

Note that `$HDP_DIR` should be replaced by the *absolute* path to your `$HDP_DIR`.

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:$HDP_DIR/yarn_data/hdfs/namenode</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:$HDP_DIR/yarn_data/hdfs/datanode</value>
  </property>
</configuration>
```

####2.3 yarn-site.xml

Edit `$YARN_HOME/etc/hadoop/yarn-site.xml` so that it looks like this:

```
<?xml version="1.0"?>
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce.shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>
</configuration>
```
####2.4 mapred-site.xml

Edit `$YARN_HOME/etc/hadoop/mapred-site.xml` so that it looks like this:

```
<?xml version="1.0"?>
<?xml-stylesheet href="configuration.xsl"?>
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
```

####2.5 yarn-env.sh

Edit `$YARN_HOME/etc/hadoop/mapred-site.xml` by adding the following lines under the `export YARN_CONF_DIR` line:

```
export HADOOP_CONF_DIR="${HADOOP_CONF_DIR:-$YARN_HOME/etc/hadoop}"
export HADOOP_COMMON_HOME="${HADOOP_COMMON_HOME:-$YARN_HOME}"
export HADOOP_HDFS_HOME="${HADOOP_HDFS_HOME:-$YARN_HOME}"
```

###Step 3 - Format The Namenode

This sets up the HDFS file system for the namenode before we bring it online

```
> cd $YARN_HOME
> bin/hdfs namenode -format
```

You should see a ton of text, but somewhere in there you should see a line that says something like:

```
LINE GOES HERE
```

###Step 4 - Start Your Engines

First, start the HDFS services. This is just a one-liner that does it all at once but you can do it however you want (different terminal windows, etc...) because you might get a lot of log lines scrolling:

```
> cd $YARN_HOME
>./bin/hdfs namenode & ./bin/hdfs secondarynamenode & ./bin/hdfs datanode &
```

At this point you should be able to go to `http://localhost:50070/dfshealth.jsp` and see something like this:

![alt text](https://github.com/dieterichlawson/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")

