This is a quick guide to setting up Cloudera's CDH-2.0.0-cdh4 in local mode for CS246 and is largely a cleaned up and modified version of [this blog article](http://practicalcloudcomputing.com/post/26448910436/install-and-run-hadoop-yarn-in-10-easy-steps). 

Pull requests/corrections/recommendations welcome.

**NOTE**: This is *not* a guide to setting up Eclipse locally, which you will also have to do if you want to use Hadoop's Java API outside of the provided VM. The short version of how to set up Eclipse is to download it and then install the m2e plugin (Maven for Eclipse). Then you should generally be able to follow the HW0 guidelines for setting up a Hadoop project.

#### Requirements

* Java 1.6
* A Mac (not tested on anything else)

#### Directories
Before we start you should pick a directory where you want to put your hadoop installation. I put mine in `~/dev/cs246`, but feel free to pick your own directory. Just be careful to change the commands and paths properly when you do. I'll try to give you a heads up when you should do this.

###Step 1 - Download and Setup

Download Cloudera's Hadoop distribution, untar it, and set up a link to the directory so things are a bit cleaner. If you don't have `wget`, just download the tar.gz file and put it in your chosen directory.

```
> cd ~/dev/cs246
> wget http://archive.cloudera.com/cdh4/cdh/4/hadoop-2.0.0-cdh4.0.0.tar.gz
> tar -xvzf hadoop-2.0.0-cdh4.0.0.tar.gz
> ln -sf hadoop-2.0.0-cdh4.0.0 hadoop
> rm hadoop-2.0.0-cdh4.0.0.tar.gz
```
We also need to create 2 directories that will hold the data for the namenode's filesystem and the datanode's filesystem. I chose to put these as siblings to my hadoop directory as follows.

```
> mkdir -p yarn_data/hdfs/namenode
> mkdir yarn_data/hdfs/datanode
```

After this your directory should look something like this.

```
> ls -la
total 8
drwxr-xr-x   4 dlaw  wheel  136 Jan 13 14:54 .
drwxrwxrwt  21 root  wheel  714 Jan 13 14:49 ..
lrwxr-xr-x   1 dlaw  wheel   21 Jan 13 14:51 hadoop -> hadoop-2.0.0-cdh4.0.0
drwxr-xr-x  12 dlaw  wheel  408 Jun  4  2012 hadoop-2.0.0-cdh4.0.0
drwxr-xr-x   3 dlaw  wheel  102 Jan 13 15:05 yarn_data
```
Add a line to your `.bash_profile` to export a `YARN_HOME` environment variable. `YARN_HOME` should point to the `hadoop` link you made, which in turn points to the `hadoop-2.0.0-cdh4.0.0` folder. Don't forget to source it afterwards and change "`$HOME/dev/cs246`" to your chosen directory if you need to.

```
> echo 'export YARN_HOME=$HOME/dev/cs246/hadoop' >> ~/.bash_profile
> source ~/.bash_profile
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

Note that you should include the *absolute* paths to the directory that contains your hadoop installation as the values for `dfs.namenode.name.dir` and `dfs.datanode.data.dir`. My hadoop installation is in `~/dev/cs246` so I used `/Users/dlaw/dev/cs246/...`

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
    <value>file:/Users/dlaw/dev/cs246/yarn_data/hdfs/namenode</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/Users/dlaw/dev/cs246/yarn_data/hdfs/datanode</value>
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

Edit `$YARN_HOME/etc/hadoop/yarn-env.sh` by adding the following lines under the `export YARN_CONF_DIR` line:

```
export HADOOP_CONF_DIR="${HADOOP_CONF_DIR:-$YARN_HOME/etc/hadoop}"
export HADOOP_COMMON_HOME="${HADOOP_COMMON_HOME:-$YARN_HOME}"
export HADOOP_HDFS_HOME="${HADOOP_HDFS_HOME:-$YARN_HOME}"
```

###Step 3 - Format The Namenode

This sets up the HDFS file system for the namenode before we bring it online

```
> cd $YARN_HOME
> ./bin/hdfs namenode -format
```

You should see a ton of text, but somewhere in there you should see a line that says something like:

```
14/01/13 19:52:50 INFO namenode.NNStorage: Storage directory /Users/dlaw/dev/cs246/yarn_data/hdfs/namenode has been successfully formatted
```

###Step 4 - Start Your Engines

####4.1 Start the HDFS (namenode, secondarynamenode, and datanode)
First, start the HDFS services. This is just a one-liner that does it all at once but you can do it however you want (different terminal windows, etc...) because you might get a lot of log lines scrolling:

```
> cd $YARN_HOME
>./bin/hdfs namenode & ./bin/hdfs secondarynamenode & ./bin/hdfs datanode &
```

At this point you should be able to go to [http://localhost:50070/dfshealth.jsp](http://localhost:50070/dfshealth.jsp) and see something like this:

![DFS picture](https://raw2.github.com/dieterichlawson/hadoop_setup/master/dfs.png "DFS Healthy")

####4.2 Start the Yarn services

Now we need to start the Yarn services. For those of you familiar with older versions of Hadoop this is essentially the tasktracker, jobtracker, etc... Your terminal will probably have been taken over by the HDFS processes' logging, so you should probably start a new terminal tab. Again, this is a one-liner that can be messed with if necessary.

```
> cd $YARN_HOME
> ./bin/yarn resourcemanager & ./bin/yarn nodemanager &
```
You should now be able to go to [http://localhost:8088/cluster](http://localhost:8088/cluster) and see this:

![jobtracker](https://raw2.github.com/dieterichlawson/hadoop_setup/master/jobtracker.png "Jobtracker")

Congratulations! You now have a working Hadoop setup.

### Step 5 Final Details

First, we want to be able to run the `hadoop` command so you should add your `$YARN_HOME/bin` directory to your path like this (changing `$HOME/dev/cs246` if you need to):

```
> echo 'export PATH="$PATH:$HOME/dev/cs246/hadoop/bin"' >> ~/.bash_profile
> source ~/.bash_profile
```

You should now be able to run:

```
hadoop fs -ls /
```

from any directory, but it shouldn't return anything because you haven't put anything on the HDFS yet.



#### Starting and Stopping: An Option

We also have to manage starting and stopping the cluster. Here is one option:

```
> cd $YARN_HOME
> ./bin/hdfs namenode & ./bin/hdfs secondarynamenode & ./bin/hdfs datanode & ./bin/yarn resourcemanager & ./bin/yarn nodemanager &
```
This line starts all the processes at the same time, and will give you one window flooded with the logging output. This works just fine, but to stop it, you could either `grep` through `ps` and kill the processes or find their `pids` with the `jps` command (lists all Java processes). I honestly don't know of a good way to gracefully bring down the cluster if you start it this way, but if you do then go for it.

There is another option that I prefer called 'Foreman'.

#### Option Two: Foreman

Foreman is a useful tool that lets you start and stop groups of services. I **highly**  recommend you use it because it provides a nice output and a robust way to easily start and stop your local Hadoop cluster. You can find out more about Foreman and it install it [here](https://github.com/ddollar/foreman).

Foreman uses a file called a "Procfile" that defines the processes you are managing. In your `$YARN_HOME` you can create a file called `Procfile` that contains the following:

```
namenode: bin/hdfs namenode
secondarynn: bin/hdfs secondarynamenode
datanode: bin/hdfs datanode
resourcemgr: bin/yarn resourcemanager
nodemgr: bin/yarn nodemanager
```

Then, to start up the Hadoop cluster you only need `cd` into the directory with your Procfile in it and run:

```
cd $YARN_HOME
foreman start
```

You will see something that looks like this:

![foreman](https://raw2.github.com/dieterichlawson/hadoop_setup/master/foreman.png "foreman")

Pretty, huh?

To stop the cluster, just Ctrl-c foreman.