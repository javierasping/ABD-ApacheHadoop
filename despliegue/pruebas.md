
```bash
debian@cruces-k8s-1:~/docker-hadoop$ docker ps
CONTAINER ID   IMAGE                                                    COMMAND                  CREATED          STATUS                             PORTS                                                                                  NAMES
1886eb9c334b   bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8          "/entrypoint.sh /run…"   26 seconds ago   Up 17 seconds (health: starting)   0.0.0.0:9000->9000/tcp, :::9000->9000/tcp, 0.0.0.0:9870->9870/tcp, :::9870->9870/tcp   namenode
774bdccecf45   bde2020/hadoop-resourcemanager:2.0.0-hadoop3.2.1-java8   "/entrypoint.sh /run…"   26 seconds ago   Up 18 seconds (health: starting)   8088/tcp                                                                               resourcemanager
21256d355687   bde2020/hadoop-nodemanager:2.0.0-hadoop3.2.1-java8       "/entrypoint.sh /run…"   26 seconds ago   Up 17 seconds (health: starting)   8042/tcp                                                                               nodemanager
319cf2cff022   bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8          "/entrypoint.sh /run…"   26 seconds ago   Up 19 seconds (health: starting)   9864/tcp                                                                               datanode
054ef200e9a5   bde2020/hadoop-historyserver:2.0.0-hadoop3.2.1-java8     "/entrypoint.sh /run…"   26 seconds ago   Up 18 seconds (health: starting)   8188/tcp                                                                               historyserver

debian@cruces-k8s-1:~/docker-hadoop$ docker exec -it namenode bash

root@1886eb9c334b:/# ls
KEYS  boot  entrypoint.sh  hadoop	home  lib64  mnt  proc	run	sbin  sys  usr
bin   dev   etc		   hadoop-data	lib   media  opt  root	run.sh	srv   tmp  var

root@1886eb9c334b:/# hdfs dfs -ls /

Found 1 items
drwxr-xr-x   - root supergroup          0 2024-04-14 15:54 /rmstate


root@1886eb9c334b:/# hdfs dfs -mkdir -p /user/root

root@1886eb9c334b:/# hdfs dfs -ls /
Found 2 items
drwxr-xr-x   - root supergroup          0 2024-04-14 15:54 /rmstate
drwxr-xr-x   - root supergroup          0 2024-04-14 15:56 /user


debian@cruces-k8s-1:~/docker-hadoop$ docker cp hadoop-mapreduce-examples-2.7.1-sources.jar  namenode:/tmp
debian@cruces-k8s-1:~/docker-hadoop$ docker cp input1.txt  namenode:/tmp

debian@cruces-k8s-1:~/docker-hadoop$ docker exec -it namenode bash
root@1886eb9c334b:/# ls /tmp/
hadoop-mapreduce-examples-2.7.1-sources.jar  input1.txt
hadoop-root-namenode.pid		     jetty-0.0.0.0-9870-hdfs-_-any-9179735125743414600.dir
hsperfdata_root


root@1886eb9c334b:/# hdfs dfs -mkdir /user/root/input
root@1886eb9c334b:/# hdfs dfs -mkdir /user/root/output
root@1886eb9c334b:/# cd /tmp/         



root@1886eb9c334b:/tmp# hdfs dfs -put input1.txt /user/root/input
2024-04-14 16:10:46,702 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false


root@1886eb9c334b:/tmp# hadoop jar hadoop-mapreduce-examples-2.7.1-sources.jar org.apache.hadoop.examples.WordCount input output
root@1886eb9c334b:/tmp# hadoop jar hadoop-mapreduce-examples-2.7.1-sources.jar org.apache.hadoop.examples.WordCount input output
2024-04-14 16:41:58,130 INFO client.RMProxy: Connecting to ResourceManager at resourcemanager/172.18.0.3:8032
2024-04-14 16:41:59,588 INFO client.AHSProxy: Connecting to Application History server at historyserver/172.18.0.5:10200
2024-04-14 16:42:01,220 INFO mapreduce.JobResourceUploader: Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/root/.staging/job_1713112542500_0001
2024-04-14 16:42:02,128 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
2024-04-14 16:42:02,943 INFO input.FileInputFormat: Total input files to process : 1
2024-04-14 16:42:03,473 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
2024-04-14 16:42:03,701 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
2024-04-14 16:42:04,038 INFO mapreduce.JobSubmitter: number of splits:1
2024-04-14 16:42:05,098 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
2024-04-14 16:42:05,244 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1713112542500_0001
2024-04-14 16:42:05,245 INFO mapreduce.JobSubmitter: Executing with tokens: []
2024-04-14 16:42:06,312 INFO conf.Configuration: resource-types.xml not found
2024-04-14 16:42:06,314 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
2024-04-14 16:42:08,212 INFO impl.YarnClientImpl: Submitted application application_1713112542500_0001
2024-04-14 16:42:08,602 INFO mapreduce.Job: The url to track the job: http://resourcemanager:8088/proxy/application_1713112542500_0001/
2024-04-14 16:42:08,610 INFO mapreduce.Job: Running job: job_1713112542500_0001
2024-04-14 16:42:57,409 INFO mapreduce.Job: Job job_1713112542500_0001 running in uber mode : false
2024-04-14 16:42:57,423 INFO mapreduce.Job:  map 0% reduce 0%
2024-04-14 16:43:27,784 INFO mapreduce.Job:  map 100% reduce 0%
2024-04-14 16:43:53,401 INFO mapreduce.Job:  map 100% reduce 100%
2024-04-14 16:43:57,684 INFO mapreduce.Job: Job job_1713112542500_0001 completed successfully
2024-04-14 16:43:58,173 INFO mapreduce.Job: Counters: 54
	File System Counters
		FILE: Number of bytes read=839
		FILE: Number of bytes written=460223
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=20840319
		HDFS: Number of bytes written=1454
		HDFS: Number of read operations=8
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=2
		HDFS: Number of bytes read erasure-coded=0
	Job Counters 
		Launched map tasks=1
		Launched reduce tasks=1
		Rack-local map tasks=1
		Total time spent by all maps in occupied slots (ms)=100944
		Total time spent by all reduces in occupied slots (ms)=181808
		Total time spent by all map tasks (ms)=25236
		Total time spent by all reduce tasks (ms)=22726
		Total vcore-milliseconds taken by all map tasks=25236
		Total vcore-milliseconds taken by all reduce tasks=22726
		Total megabyte-milliseconds taken by all map tasks=103366656
		Total megabyte-milliseconds taken by all reduce tasks=186171392
	Map-Reduce Framework
		Map input records=475084
		Map output records=1895407
		Map output bytes=28421810
		Map output materialized bytes=831
		Input split bytes=112
		Combine input records=1895407
		Combine output records=144
		Reduce input groups=144
		Reduce shuffle bytes=831
		Reduce input records=144
		Reduce output records=144
		Spilled Records=288
		Shuffled Maps =1
		Failed Shuffles=0
		Merged Map outputs=1
		GC time elapsed (ms)=912
		CPU time spent (ms)=13520
		Physical memory (bytes) snapshot=548347904
		Virtual memory (bytes) snapshot=13501370368
		Total committed heap usage (bytes)=530055168
		Peak Map Physical memory (bytes)=377655296
		Peak Map Virtual memory (bytes)=5078142976
		Peak Reduce Physical memory (bytes)=170692608
		Peak Reduce Virtual memory (bytes)=8423227392
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=20840207
	File Output Format Counters 
		Bytes Written=1454
root@1886eb9c334b:/tmp# 


root@1886eb9c334b:/tmp# hdfs dfs -cat /user/root/output/*
2024-04-14 17:00:40,689 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
Cuerpo	138
Demostrativos	46
Interjecciones	46
Locuciones	46
Pronombres	1851960
PronombresPronombres	9062
Pronombress	7130
a	230
acerca	46
adolescente	46
adulta	46
adulto	46
algo	966
alguien	46
alguna	966
algún	966
ambos	46
anciana	138
anciano	138
aquel	46
aquella	46
aquellas	46
aquello	46
aquellos	46
boca	46
caballero	236
cabeza	46
cara	46
conmigo	92
consigo	276
contigo	92
cualquiera	46
cuerpo	322
cuya	782
cuyo	782
cuál	1380
cuánto	598
cuántoPronombres	782
dado	46
dama	142
damaseñor	93
damavv	1
de	138
decir	46
demás	46
diente	46
don	138
doña	138
ella	46
ellas	46
ello	46
ellos	46
es	46
esa	46
esas	46
ese	46
eso	46
esos	46
espinilla	46
esta	46
estas	46
este	46
esto	46
estos	46
gracias	46
humano	138
indefinidos	230
indefinidosun	736
interrogativos	1380
la	276
labio	46
le	276
lo	322
me	276
mejor	46
menudo	46
misma	46
mismo	46
muslo	46
mí	92
mía	92
mío	92
nada	920
nadie	46
ni	46
ninguna	920
ninguno	920
no	46
nos	92
nosotras	46
nosotros	46
nuestra	92
nuestro	92
o	46
ojuandavi	23
ojuandavid	253
os	92
otra	46
otro	46
pesar	46
pie	322
pierna	322
posesivos	92
propósito	46
que	46
quién	46
qué	46
rodilla	46
s	92
salud	138
se	276
sea	46
señor	143
señora	236
siquiera	46
suya	92
suyo	92
sí	322
talón	46
tan	46
tanta	46
tanto	46
te	276
ti	92
través	46
tuya	92
tuyo	92
tú	46
un	230
una	966
uno	966
usted	46
ustedes	46
varias	782
variasPronombres	138
varios	920
vos	46
vosotras	46
vosotros	46
vuestra	92
vuestro	92
y	138
yo	46
él	46  



https://raw.githubusercontent.com/josephmisiti/hadoop-examples/master/wordcount/gutenberg/pg20417.txt