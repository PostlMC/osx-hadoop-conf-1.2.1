# Hadoop 1.2.1 Setup on sMACk

(sMACk == My MacBook Pro)

Based (mostly verbatim) on: __Hadoop initial configuration on OSX__ at [borrelli.org](http://borrelli.org/2012/05/03/hadoop-initial-configuration/)

## Configuration

1. Edit core-site.xml
2. Edit hdfs-site.xml
3. Edit mapred-site.xml

```bash
sudo mkdir -p /opt/var/hadoop /opt/var/hadoop/nn /opt/var/hadoop/dfs /opt/var/hadoop/mapred
sudo chown -R $(whoami) /opt/var/hadoop
```

Format the namenode (once):

    scott@sMACk:/opt/var/hadoop> hadoop namenode -format
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

Start it up:

	scott@sMACk:/Users/scott> start-all.sh
	starting namenode, logging to /usr/local/Cellar/hadoop/1.2.1/libexec/bin/../logs/hadoop-scott-namenode-sMACk.local.out
	localhost: starting datanode, logging to /usr/local/Cellar/hadoop/1.2.1/libexec/bin/../logs/hadoop-scott-datanode-sMACk.local.out
	localhost: starting secondarynamenode, logging to /usr/local/Cellar/hadoop/1.2.1/libexec/bin/../logs/hadoop-scott-secondarynamenode-sMACk.local.out
	starting jobtracker, logging to /usr/local/Cellar/hadoop/1.2.1/libexec/bin/../logs/hadoop-scott-jobtracker-sMACk.local.out
	localhost: starting tasktracker, logging to /usr/local/Cellar/hadoop/1.2.1/libexec/bin/../logs/hadoop-scott-tasktracker-sMACk.local.out

## Testing

Make a place for the test work:

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

	scott@sMACk:/Users/scott> cd /usr/local/Cellar/hadoop/1.2.1/libexec
	scott@sMACk:/usr/local/Cellar/hadoop/1.2.1/libexec> time hadoop jar hadoop-examples*.jar wordcount /gutenberg /output
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
