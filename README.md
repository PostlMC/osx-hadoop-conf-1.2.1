# Hadoop 1.2.1 Setup on sMACk

(where sMACk == my MacBook Pro)

These are my notes on setting up Hadoop 1.2.1 on my MacBook for experimention.  The tutorial I used to get started (see below) suggests storing Hadoop's config directory in Git, which seemed like a Really Good Idea&trade; to me, and every decent Git repo needs some kind of README file, so here we are.


## Credit Where Due
This is all based -- _some parts very nearly verbatim_ -- on [Hadoop initial configuration on OSX](http://borrelli.org/2012/05/03/hadoop-initial-configuration/) at [borrelli.org](http://borrelli.org/).  Versions and paths have been changed to suit my local environment and curious habits.


## Base Install
There's not a lot of detail in this section because it's all boring lead up bits.

1. Just use [Homebrew](http://brew.sh/) to get Hadoop itself.
2. Make sure it's possible to SSH to `localhost` [without a password prompt](https://www.google.com/search?q=ssh+passwordless+login) using whatever account will run Hadoop.
3. Make all the data directories.  A previous life left me with a predilection for sticking non-OS-ey things in `/opt`, so here goes.:

```bash
sudo mkdir -p /opt/var/hadoop /opt/var/hadoop/nn /opt/var/hadoop/dfs /opt/var/hadoop/mapred
sudo chown -R $(whoami) /opt/var/hadoop
```


## Initial Configuration

### Kerberos Fix
This looks important.  Let's make sure this is in `hadoop-env.sh`:

```bash
export HADOOP_OPTS="-Djava.security.krb5.realm= -Djava.security.krb.kdc="
```

### Edit `core-site.xml`
```xml
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/tmp/hdfs-${user.name}</value>
        <description>A base for other temporary directories.</description>
    </property>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://localhost:54310</value>
        <description>The name of the default file system.  A URI whose
          scheme and authority determine the FileSystem implementation.  The
          uri's scheme determines the config property (fs.SCHEME.impl) naming
          the FileSystem implementation class.  The uri's authority is used to
          determine the host, port, etc. for a filesystem.</description>
    </property>
</configuration>
```

### Edit `hdfs-site.xml`
```
<configuration>
    <property>
        <name>dfs.data.dir</name>
        <value>/opt/var/hadoop/dfs</value>
    </property>
    <property>
        <name>dfs.name.dir</name>
        <value>/opt/var/hadoop/nn</value>
    </property>
</configuration>
```

### Edit `mapred-site.xml`
```xml
<configuration>
    <property>
        <name>mapred.job.tracker</name>
        <value>localhost:54311</value>
        <description>The host and port that the MapReduce job tracker runs
          at.  If &quot;local&quot;, then jobs are run in-process as a single map
          and reduce task.</description>
    </property>
    <property>
        <name>mapred.local.dir</name>
        <value>/opt/var/hadoop/mapred/</value>
    </property>
</configuration>
```

### Format the [namenode](http://wiki.apache.org/hadoop/NameNode)
Do this only once:

```
hadoop namenode -format
```

The output will go a little something like this:

	14/01/02 16:04:09 INFO namenode.NameNode: STARTUP_MSG:
	/************************************************************
	STARTUP_MSG: Starting NameNode
	STARTUP_MSG:   host = sMACk.local/192.168.42.199
	STARTUP_MSG:   args = [-format]
	STARTUP_MSG:   version = 1.2.1
	STARTUP_MSG:   build = https://svn.apache.org/repos/asf/hadoop/common/branches/branch-1.2 -r 1503152; compiled by 'mattf' on Mon Jul 22 15:23:09 PDT 2013
	STARTUP_MSG:   java = 1.6.0_65
	************************************************************/
	Re-format filesystem in /opt/var/hadoop/nn ? (Y or N) Y
	14/01/02 16:04:19 INFO util.GSet: Computing capacity for map BlocksMap
	14/01/02 16:04:19 INFO util.GSet: VM type       = 64-bit
	14/01/02 16:04:19 INFO util.GSet: 2.0% max memory = 1039859712
	14/01/02 16:04:19 INFO util.GSet: capacity      = 2^21 = 2097152 entries
	14/01/02 16:04:19 INFO util.GSet: recommended=2097152, actual=2097152
	14/01/02 16:04:19 INFO namenode.FSNamesystem: fsOwner=scott
	14/01/02 16:04:19 INFO namenode.FSNamesystem: supergroup=supergroup
	14/01/02 16:04:19 INFO namenode.FSNamesystem: isPermissionEnabled=true
	14/01/02 16:04:19 INFO namenode.FSNamesystem: dfs.block.invalidate.limit=100
	14/01/02 16:04:19 INFO namenode.FSNamesystem: isAccessTokenEnabled=false accessKeyUpdateInterval=0 min(s), accessTokenLifetime=0 min(s)
	14/01/02 16:04:19 INFO namenode.FSEditLog: dfs.namenode.edits.toleration.length = 0
	14/01/02 16:04:19 INFO namenode.NameNode: Caching file names occuring more than 10 times
	14/01/02 16:04:20 INFO common.Storage: Image file /opt/var/hadoop/nn/current/fsimage of size 111 bytes saved in 0 seconds.
	14/01/02 16:04:20 INFO namenode.FSEditLog: closing edit log: position=4, editlog=/opt/var/hadoop/nn/current/edits
	14/01/02 16:04:20 INFO namenode.FSEditLog: close success: truncate to 4, editlog=/opt/var/hadoop/nn/current/edits
	14/01/02 16:04:20 INFO common.Storage: Storage directory /opt/var/hadoop/nn has been successfully formatted.
	14/01/02 16:04:20 INFO namenode.NameNode: SHUTDOWN_MSG:
	/************************************************************
	SHUTDOWN_MSG: Shutting down NameNode at sMACk.local/192.168.42.199
	************************************************************/

### Annoyance: Java Apps and Dock Icons on OSX
Since I'm running an Apple JVM, OSX places these really unattractive little gray icons in the Dock for each of the processes started with Hadoop:


To get rid of them, add `-Dapple.awt.UIElement` to the `HADOOP-OPTS` variable in `hadoop-env.sh`.  With this and the Kerberos fix, mine now reads:

    export HADOOP_OPTS="-Dapple.awt.UIElement=true -Djava.security.krb5.realm= -Djava.security.krb.kdc="

 Here's a [Stack Overflow discussion](http://stackoverflow.com/questions/8246766/how-to-hide-the-java-swt-program-icon-in-the-dock-when-the-application-is-in-the) on the matter, and more info on that particular Apple JVM property can be found [here](https://developer.apple.com/library/mac/documentation/java/Reference/Java_PropertiesRef/Articles/JavaSystemProperties.html).


## Testing
Start it up:

```bash
start-all.sh
```

Output:

	starting namenode, logging to /usr/local/Cellar/hadoop/1.2.1/libexec/bin/../logs/hadoop-scott-namenode-sMACk.local.out
	localhost: starting datanode, logging to /usr/local/Cellar/hadoop/1.2.1/libexec/bin/../logs/hadoop-scott-datanode-sMACk.local.out
	localhost: starting secondarynamenode, logging to /usr/local/Cellar/hadoop/1.2.1/libexec/bin/../logs/hadoop-scott-secondarynamenode-sMACk.local.out
	starting jobtracker, logging to /usr/local/Cellar/hadoop/1.2.1/libexec/bin/../logs/hadoop-scott-jobtracker-sMACk.local.out
	localhost: starting tasktracker, logging to /usr/local/Cellar/hadoop/1.2.1/libexec/bin/../logs/hadoop-scott-tasktracker-sMACk.local.out

Make a place in hdfs for the test work:

```bash
hadoop fs -mkdir /gutenberg
hadoop fs -ls /
```

Put something to work on in the new location:

```bash
curl http://mirrors.xmission.com/gutenberg/1/3/4/1342/1342.txt | hadoop fs -put - /gutenberg/1342.txt
hadoop fs -ls /gutenberg
```

...and, test:

```bash
time /usr/local/Cellar/hadoop/1.2.1/libexec/hadoop jar hadoop-examples*.jar wordcount /gutenberg /output
```

Output: 

	14/01/02 16:23:50 INFO input.FileInputFormat: Total input paths to process : 1
	14/01/02 16:23:50 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
	14/01/02 16:23:50 WARN snappy.LoadSnappy: Snappy native library not loaded
	14/01/02 16:23:50 INFO mapred.JobClient: Running job: job_201401021609_0004
	14/01/02 16:23:51 INFO mapred.JobClient:  map 0% reduce 0%
	14/01/02 16:23:56 INFO mapred.JobClient:  map 100% reduce 0%
	14/01/02 16:24:03 INFO mapred.JobClient:  map 100% reduce 33%
	14/01/02 16:24:05 INFO mapred.JobClient:  map 100% reduce 100%
	14/01/02 16:24:05 INFO mapred.JobClient: Job complete: job_201401021609_0004
	14/01/02 16:24:05 INFO mapred.JobClient: Counters: 26
	14/01/02 16:24:05 INFO mapred.JobClient:   Job Counters
	14/01/02 16:24:05 INFO mapred.JobClient:     Launched reduce tasks=1
	14/01/02 16:24:05 INFO mapred.JobClient:     SLOTS_MILLIS_MAPS=5196
	14/01/02 16:24:05 INFO mapred.JobClient:     Total time spent by all reduces waiting after reserving slots (ms)=0
	14/01/02 16:24:05 INFO mapred.JobClient:     Total time spent by all maps waiting after reserving slots (ms)=0
	14/01/02 16:24:05 INFO mapred.JobClient:     Launched map tasks=1
	14/01/02 16:24:05 INFO mapred.JobClient:     Data-local map tasks=1
	14/01/02 16:24:05 INFO mapred.JobClient:     SLOTS_MILLIS_REDUCES=8759
	14/01/02 16:24:05 INFO mapred.JobClient:   File Output Format Counters
	14/01/02 16:24:05 INFO mapred.JobClient:     Bytes Written=148338
	14/01/02 16:24:05 INFO mapred.JobClient:   FileSystemCounters
	14/01/02 16:24:05 INFO mapred.JobClient:     FILE_BYTES_READ=201438
	14/01/02 16:24:05 INFO mapred.JobClient:     HDFS_BYTES_READ=717703
	14/01/02 16:24:05 INFO mapred.JobClient:     FILE_BYTES_WRITTEN=508699
	14/01/02 16:24:05 INFO mapred.JobClient:     HDFS_BYTES_WRITTEN=148338
	14/01/02 16:24:05 INFO mapred.JobClient:   File Input Format Counters
	14/01/02 16:24:05 INFO mapred.JobClient:     Bytes Read=717597
	14/01/02 16:24:05 INFO mapred.JobClient:   Map-Reduce Framework
	14/01/02 16:24:05 INFO mapred.JobClient:     Map output materialized bytes=201438
	14/01/02 16:24:05 INFO mapred.JobClient:     Map input records=13427
	14/01/02 16:24:05 INFO mapred.JobClient:     Reduce shuffle bytes=201438
	14/01/02 16:24:05 INFO mapred.JobClient:     Spilled Records=27280
	14/01/02 16:24:05 INFO mapred.JobClient:     Map output bytes=1199776
	14/01/02 16:24:05 INFO mapred.JobClient:     Total committed heap usage (bytes)=281808896
	14/01/02 16:24:05 INFO mapred.JobClient:     Combine input records=124592
	14/01/02 16:24:05 INFO mapred.JobClient:     SPLIT_RAW_BYTES=106
	14/01/02 16:24:05 INFO mapred.JobClient:     Reduce input records=13640
	14/01/02 16:24:05 INFO mapred.JobClient:     Reduce input groups=13640
	14/01/02 16:24:05 INFO mapred.JobClient:     Combine output records=13640
	14/01/02 16:24:05 INFO mapred.JobClient:     Reduce output records=13640
	14/01/02 16:24:05 INFO mapred.JobClient:     Map output records=124592

	real	0m16.472s
	user	0m2.486s
	sys	0m0.186s

I have no frame of reference to tell me if the time is good, but having satisfies my curiousity.  Also: Yay! No shrapnel!  

Here's what came out:

	scott@sMACk:/usr/local/Cellar/hadoop/1.2.1/libexec> hadoop fs -ls /output
	Found 3 items
	-rw-r--r--   3 scott supergroup          0 2014-01-02 16:24 /output/_SUCCESS
	drwxr-xr-x   - scott supergroup          0 2014-01-02 16:23 /output/_logs
	-rw-r--r--   3 scott supergroup     148338 2014-01-02 16:24 /output/part-r-00000

	scott@sMACk:/usr/local/Cellar/hadoop/1.2.1/libexec> hadoop fs -cat /output/part-r-00000 | sort -rnk2 | head
	the	4205
	to	4121
	of	3660
	and	3309
	a	1945
	her	1858
	in	1813
	was	1796
	I	1740
	that	1419

