### 4.2 Encontrar la temperatura máxima global para un año dado

Vamos a crear un pequeño programa para calcular la temperatura mas alta de cada año . Para ello seguiremos este ejemplo proveniente del libro Hadoop: The Definitive Guide, Fourth Edition

Para aprovechar Hadoop, tenemos que diseñar nuestro código para que funcione en el modelo MapReduce. Tanto la fase de map como la de reduce trabajan con pares clave-valor como entrada y salida, y ambas tienen una función definida por el programador.

Utilizaremos Java, porque es una dependencia que ya tenemos de todos modos, así que también podríamos hacerlo.

Nuestra función de map necesita extraer el año y la temperatura del aire, lo que preparará los datos para su uso posterior (encontrar la temperatura máxima para cada año). También eliminaremos registros incorrectos aquí (si falta la temperatura, es sospechosa o errónea).

Copia o reproduce el siguiente código en un archivo llamado MaxTempMapper.java, utilizando cualquier editor de texto de tu elección:

```java
import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class MaxTempMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    private static final int TEMP_MISSING = 9999;
    private static final String GOOD_QUALITY_RE = "[01459]";

    @Override
    public void map(LongWritable key, Text value, Context context)
            throws IOException, InterruptedException {
        String line = value.toString();
        String year = line.substring(15, 19);
        String temp = line.substring(87, 92).replaceAll("^\\+", "");
        String quality = line.substring(92, 93);

        int airTemperature = Integer.parseInt(temp);
        if (airTemperature != TEMP_MISSING && quality.matches(GOOD_QUALITY_RE)) {
            context.write(new Text(year), new IntWritable(airTemperature));
        }
    }
}
```

Ahora, creemos el archivo MaxTempReducer.java. Su trabajo es reducir los datos de múltiples valores a solo uno. Lo hacemos manteniendo el valor máximo de todos los valores que recibimos:

```java
import java.io.IOException;
import java.util.Iterator;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class MaxTempReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    @Override
    public void reduce(Text key, Iterable<IntWritable> values, Context context)
            throws IOException, InterruptedException {
        Iterator<IntWritable> iter = values.iterator();
        if (iter.hasNext()) {
            int maxValue = iter.next().get();
            while (iter.hasNext()) {
                maxValue = Math.max(maxValue, iter.next().get());
            }
            context.write(key, new IntWritable(maxValue));
        }
    }
}
```

Por último, escribamos el método principal, o de lo contrario no podremos ejecutarlo. En nuestro nuevo archivo MaxTemp.java:

```java
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class MaxTemp {
    public static void main(String[] args) throws Exception {
        if (args.length != 2) {
            System.err.println("uso: java MaxTemp <ruta de entrada> <ruta de salida>");
            System.exit(-1);
        }

        Job job = Job.getInstance();

        job.setJobName("Temperatura máxima");
        job.setJarByClass(MaxTemp.class);
        job.setMapperClass(MaxTempMapper.class);
        job.setReducerClass(MaxTempReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        boolean resultado = job.waitForCompletion(true);

        System.exit(resultado ? 0 : 1);
    }
}

```

Ademas nos vamos a descargar el dataset que vamos a utilizar para el ejemplo :

```bash
curl https://raw.githubusercontent.com/tomwhite/hadoop-book/master/input/ncdc/all/190{1,2}.gz | gunzip > 190x
```

Si listamos el formato de los datos que nos acabamos de descargar , tendra esta pinta :

```bash
0029227070999991902123013004+62167+030650FM-12+010299999V0200501N001019999999N0000001N9-01561+99999097981ADDGF108991999999999999999999
0029227070999991902123020004+62167+030650FM-12+010299999V0209991C000019999999N0000001N9-01501+99999098461ADDGF100991999999999999999999
0029227070999991902123106004+62167+030650FM-12+010299999V0200501N002119999999N0000001N9-01171+99999098641ADDGF108991999999999999999999
0029227070999991902123113004+62167+030650FM-12+010299999V0200501N002119999999N0000001N9-01281+99999099551ADDGF108991999999999999999999
0029227070999991902123120004+62167+030650FM-12+010299999V0200501N002119999999N0000001N9-01831+99999100111ADDGF100991999999999999999999
```

Ahora deberemos de llevarnos estos cuatro ficheros dentro del contenedor docker :

```bash
debian@cruces-k8s-1:~/docker-hadoop$ docker cp 190x  namenode:/tmp
debian@cruces-k8s-1:~/docker-hadoop$ docker cp MaxTemp.java  namenode:/tmp
debian@cruces-k8s-1:~/docker-hadoop$ docker cp MaxTempMapper.java  namenode:/tmp
debian@cruces-k8s-1:~/docker-hadoop$ docker cp MaxTempReducer.java  namenode:/tmp
```

Necesitaremos saber cual es la ruta donde tenemos instalado hadoop asi que vamos a buscarlo :

```bash
root@1886eb9c334b:/hadoop# find / -name "hadoop-3.2.1" 2>/dev/null

/opt/hadoop-3.2.1

```

Una vez que conocemos la ruta de los paquetes de hadoop vamos a ejecutar los .java utilizando las siguientes bibliotecas indicándolas con -cp :

```bash
root@1886eb9c334b:/tmp# java -cp ".:/opt/hadoop-3.2.1/share/hadoop/common/*:/opt/hadoop-3.2.1/share/hadoop/common/lib/*:/opt/hadoop-3.2.1/share/hadoop/mapreduce/*:/opt/hadoop-3.2.1/share/hadoop/mapreduce/lib/*:/opt/hadoop-3.2.1/share/hadoop/yarn/*:/opt/hadoop-3.2.1/share/hadoop/yarn/lib/*:/opt/hadoop-3.2.1/share/hadoop/hdfs/*:/opt/hadoop-3.2.1/share/hadoop/hdfs/lib/*" MaxTemp 190x results
    
2024-04-15 09:45:51,311 WARN  [main] util.NativeCodeLoader (NativeCodeLoader.java:<clinit>(60)) - Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
2024-04-15 09:45:53,281 WARN  [main] impl.MetricsConfig (MetricsConfig.java:loadFirst(134)) - Cannot locate configuration: tried hadoop-metrics2-jobtracker.properties,hadoop-metrics2.properties
2024-04-15 09:45:54,171 INFO  [main] impl.MetricsSystemImpl (MetricsSystemImpl.java:startTimer(374)) - Scheduled Metric snapshot period at 10 second(s).
2024-04-15 09:45:54,171 INFO  [main] impl.MetricsSystemImpl (MetricsSystemImpl.java:start(191)) - JobTracker metrics system started
2024-04-15 09:45:55,046 WARN  [main] mapreduce.JobResourceUploader (JobResourceUploader.java:uploadResourcesInternal(149)) - Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
2024-04-15 09:45:55,209 WARN  [main] mapreduce.JobResourceUploader (JobResourceUploader.java:uploadJobJar(482)) - No job jar file set.  User classes may not be found. See Job or Job#setJar(String).
2024-04-15 09:45:55,634 INFO  [main] input.FileInputFormat (FileInputFormat.java:listStatus(292)) - Total input files to process : 1
2024-04-15 09:45:56,267 INFO  [main] mapreduce.JobSubmitter (JobSubmitter.java:submitJobInternal(202)) - number of splits:1
2024-04-15 09:45:58,020 INFO  [main] mapreduce.JobSubmitter (JobSubmitter.java:printTokens(298)) - Submitting tokens for job: job_local756486036_0001
2024-04-15 09:45:58,032 INFO  [main] mapreduce.JobSubmitter (JobSubmitter.java:printTokens(299)) - Executing with tokens: []
2024-04-15 09:45:59,130 INFO  [main] mapreduce.Job (Job.java:submit(1574)) - The url to track the job: http://localhost:8080/
2024-04-15 09:45:59,149 INFO  [main] mapreduce.Job (Job.java:monitorAndPrintJob(1619)) - Running job: job_local756486036_0001
2024-04-15 09:45:59,182 INFO  [Thread-24] mapred.LocalJobRunner (LocalJobRunner.java:createOutputCommitter(501)) - OutputCommitter set in config null
2024-04-15 09:45:59,265 INFO  [Thread-24] output.FileOutputCommitter (FileOutputCommitter.java:<init>(141)) - File Output Committer Algorithm version is 2
2024-04-15 09:45:59,280 INFO  [Thread-24] output.FileOutputCommitter (FileOutputCommitter.java:<init>(156)) - FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-04-15 09:45:59,319 INFO  [Thread-24] mapred.LocalJobRunner (LocalJobRunner.java:createOutputCommitter(519)) - OutputCommitter is org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter
2024-04-15 09:45:59,630 INFO  [Thread-24] mapred.LocalJobRunner (LocalJobRunner.java:runTasks(478)) - Waiting for map tasks
2024-04-15 09:45:59,641 INFO  [LocalJobRunner Map Task Executor #0] mapred.LocalJobRunner (LocalJobRunner.java:run(252)) - Starting task: attempt_local756486036_0001_m_000000_0
2024-04-15 09:45:59,751 INFO  [LocalJobRunner Map Task Executor #0] output.FileOutputCommitter (FileOutputCommitter.java:<init>(141)) - File Output Committer Algorithm version is 2
2024-04-15 09:45:59,755 INFO  [LocalJobRunner Map Task Executor #0] output.FileOutputCommitter (FileOutputCommitter.java:<init>(156)) - FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-04-15 09:45:59,879 INFO  [LocalJobRunner Map Task Executor #0] mapred.Task (Task.java:initialize(626)) -  Using ResourceCalculatorProcessTree : [ ]
2024-04-15 09:45:59,888 INFO  [LocalJobRunner Map Task Executor #0] mapred.MapTask (MapTask.java:runNewMapper(768)) - Processing split: file:/tmp/190x:0+1777168
2024-04-15 09:46:00,213 INFO  [main] mapreduce.Job (Job.java:monitorAndPrintJob(1640)) - Job job_local756486036_0001 running in uber mode : false
2024-04-15 09:46:00,265 INFO  [main] mapreduce.Job (Job.java:monitorAndPrintJob(1647)) -  map 0% reduce 0%
2024-04-15 09:46:00,426 INFO  [LocalJobRunner Map Task Executor #0] mapred.MapTask (MapTask.java:setEquator(1219)) - (EQUATOR) 0 kvi 26214396(104857584)
2024-04-15 09:46:00,427 INFO  [LocalJobRunner Map Task Executor #0] mapred.MapTask (MapTask.java:init(1012)) - mapreduce.task.io.sort.mb: 100
2024-04-15 09:46:00,427 INFO  [LocalJobRunner Map Task Executor #0] mapred.MapTask (MapTask.java:init(1013)) - soft limit at 83886080
2024-04-15 09:46:00,427 INFO  [LocalJobRunner Map Task Executor #0] mapred.MapTask (MapTask.java:init(1014)) - bufstart = 0; bufvoid = 104857600
2024-04-15 09:46:00,427 INFO  [LocalJobRunner Map Task Executor #0] mapred.MapTask (MapTask.java:init(1015)) - kvstart = 26214396; length = 6553600
2024-04-15 09:46:00,447 INFO  [LocalJobRunner Map Task Executor #0] mapred.MapTask (MapTask.java:createSortingCollector(409)) - Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-04-15 09:46:01,244 INFO  [LocalJobRunner Map Task Executor #0] mapred.LocalJobRunner (LocalJobRunner.java:statusUpdate(634)) - 
2024-04-15 09:46:01,246 INFO  [LocalJobRunner Map Task Executor #0] mapred.MapTask (MapTask.java:flush(1476)) - Starting flush of map output
2024-04-15 09:46:01,249 INFO  [LocalJobRunner Map Task Executor #0] mapred.MapTask (MapTask.java:flush(1498)) - Spilling map output
2024-04-15 09:46:01,249 INFO  [LocalJobRunner Map Task Executor #0] mapred.MapTask (MapTask.java:flush(1499)) - bufstart = 0; bufend = 118161; bufvoid = 104857600
2024-04-15 09:46:01,249 INFO  [LocalJobRunner Map Task Executor #0] mapred.MapTask (MapTask.java:flush(1501)) - kvstart = 26214396(104857584); kvend = 26161884(104647536); length = 52513/6553600
2024-04-15 09:46:01,580 INFO  [LocalJobRunner Map Task Executor #0] mapred.MapTask (MapTask.java:sortAndSpill(1696)) - Finished spill 0
2024-04-15 09:46:01,731 INFO  [LocalJobRunner Map Task Executor #0] mapred.Task (Task.java:done(1244)) - Task:attempt_local756486036_0001_m_000000_0 is done. And is in the process of committing
2024-04-15 09:46:01,750 INFO  [LocalJobRunner Map Task Executor #0] mapred.LocalJobRunner (LocalJobRunner.java:statusUpdate(634)) - map
2024-04-15 09:46:01,752 INFO  [LocalJobRunner Map Task Executor #0] mapred.Task (Task.java:sendDone(1380)) - Task 'attempt_local756486036_0001_m_000000_0' done.
2024-04-15 09:46:01,814 INFO  [LocalJobRunner Map Task Executor #0] mapred.Task (Task.java:done(1276)) - Final Counters for attempt_local756486036_0001_m_000000_0: Counters: 17
	File System Counters
		FILE: Number of bytes read=1777304
		FILE: Number of bytes written=660032
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
	Map-Reduce Framework
		Map input records=13130
		Map output records=13129
		Map output bytes=118161
		Map output materialized bytes=144425
		Input split bytes=79
		Combine input records=0
		Spilled Records=13129
		Failed Shuffles=0
		Merged Map outputs=0
		GC time elapsed (ms)=30
		Total committed heap usage (bytes)=187170816
	File Input Format Counters 
		Bytes Read=1777168
2024-04-15 09:46:01,831 INFO  [LocalJobRunner Map Task Executor #0] mapred.LocalJobRunner (LocalJobRunner.java:run(277)) - Finishing task: attempt_local756486036_0001_m_000000_0
2024-04-15 09:46:01,865 INFO  [Thread-24] mapred.LocalJobRunner (LocalJobRunner.java:runTasks(486)) - map task executor complete.
2024-04-15 09:46:01,886 INFO  [Thread-24] mapred.LocalJobRunner (LocalJobRunner.java:runTasks(478)) - Waiting for reduce tasks
2024-04-15 09:46:01,889 INFO  [pool-4-thread-1] mapred.LocalJobRunner (LocalJobRunner.java:run(330)) - Starting task: attempt_local756486036_0001_r_000000_0
2024-04-15 09:46:01,953 INFO  [pool-4-thread-1] output.FileOutputCommitter (FileOutputCommitter.java:<init>(141)) - File Output Committer Algorithm version is 2
2024-04-15 09:46:01,953 INFO  [pool-4-thread-1] output.FileOutputCommitter (FileOutputCommitter.java:<init>(156)) - FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-04-15 09:46:01,956 INFO  [pool-4-thread-1] mapred.Task (Task.java:initialize(626)) -  Using ResourceCalculatorProcessTree : [ ]
2024-04-15 09:46:02,013 INFO  [pool-4-thread-1] mapred.ReduceTask (ReduceTask.java:run(363)) - Using ShuffleConsumerPlugin: org.apache.hadoop.mapreduce.task.reduce.Shuffle@a908a4b
2024-04-15 09:46:02,058 WARN  [pool-4-thread-1] impl.MetricsSystemImpl (MetricsSystemImpl.java:init(151)) - JobTracker metrics system already initialized!
2024-04-15 09:46:02,177 INFO  [pool-4-thread-1] reduce.MergeManagerImpl (MergeManagerImpl.java:<init>(208)) - MergerManager: memoryLimit=639683776, maxSingleShuffleLimit=159920944, mergeThreshold=422191296, ioSortFactor=10, memToMemMergeOutputsThreshold=10
2024-04-15 09:46:02,196 INFO  [EventFetcher for fetching Map Completion Events] reduce.EventFetcher (EventFetcher.java:run(61)) - attempt_local756486036_0001_r_000000_0 Thread started: EventFetcher for fetching Map Completion Events
2024-04-15 09:46:02,283 INFO  [main] mapreduce.Job (Job.java:monitorAndPrintJob(1647)) -  map 100% reduce 0%
2024-04-15 09:46:02,408 INFO  [localfetcher#1] reduce.LocalFetcher (LocalFetcher.java:copyMapOutput(145)) - localfetcher#1 about to shuffle output of map attempt_local756486036_0001_m_000000_0 decomp: 144421 len: 144425 to MEMORY
2024-04-15 09:46:02,423 INFO  [localfetcher#1] reduce.InMemoryMapOutput (InMemoryMapOutput.java:doShuffle(94)) - Read 144421 bytes from map-output for attempt_local756486036_0001_m_000000_0
2024-04-15 09:46:02,463 INFO  [localfetcher#1] reduce.MergeManagerImpl (MergeManagerImpl.java:closeInMemoryFile(323)) - closeInMemoryFile -> map-output of size: 144421, inMemoryMapOutputs.size() -> 1, commitMemory -> 0, usedMemory ->144421
2024-04-15 09:46:02,467 INFO  [EventFetcher for fetching Map Completion Events] reduce.EventFetcher (EventFetcher.java:run(76)) - EventFetcher is interrupted.. Returning
2024-04-15 09:46:02,469 INFO  [pool-4-thread-1] mapred.LocalJobRunner (LocalJobRunner.java:statusUpdate(634)) - 1 / 1 copied.
2024-04-15 09:46:02,470 INFO  [pool-4-thread-1] reduce.MergeManagerImpl (MergeManagerImpl.java:finalMerge(695)) - finalMerge called with 1 in-memory map-outputs and 0 on-disk map-outputs
2024-04-15 09:46:02,578 INFO  [pool-4-thread-1] mapred.Merger (Merger.java:merge(606)) - Merging 1 sorted segments
2024-04-15 09:46:02,579 INFO  [pool-4-thread-1] mapred.Merger (Merger.java:merge(705)) - Down to the last merge-pass, with 1 segments left of total size: 144414 bytes
2024-04-15 09:46:02,782 INFO  [pool-4-thread-1] reduce.MergeManagerImpl (MergeManagerImpl.java:finalMerge(762)) - Merged 1 segments, 144421 bytes to disk to satisfy reduce memory limit
2024-04-15 09:46:02,784 INFO  [pool-4-thread-1] reduce.MergeManagerImpl (MergeManagerImpl.java:finalMerge(792)) - Merging 1 files, 144425 bytes from disk
2024-04-15 09:46:02,785 INFO  [pool-4-thread-1] reduce.MergeManagerImpl (MergeManagerImpl.java:finalMerge(807)) - Merging 0 segments, 0 bytes from memory into reduce
2024-04-15 09:46:02,785 INFO  [pool-4-thread-1] mapred.Merger (Merger.java:merge(606)) - Merging 1 sorted segments
2024-04-15 09:46:02,790 INFO  [pool-4-thread-1] mapred.Merger (Merger.java:merge(705)) - Down to the last merge-pass, with 1 segments left of total size: 144414 bytes
2024-04-15 09:46:02,801 INFO  [pool-4-thread-1] mapred.LocalJobRunner (LocalJobRunner.java:statusUpdate(634)) - 1 / 1 copied.
2024-04-15 09:46:02,903 INFO  [pool-4-thread-1] Configuration.deprecation (Configuration.java:logDeprecation(1395)) - mapred.skip.on is deprecated. Instead, use mapreduce.job.skiprecords
2024-04-15 09:46:03,066 INFO  [pool-4-thread-1] mapred.Task (Task.java:done(1244)) - Task:attempt_local756486036_0001_r_000000_0 is done. And is in the process of committing
2024-04-15 09:46:03,069 INFO  [pool-4-thread-1] mapred.LocalJobRunner (LocalJobRunner.java:statusUpdate(634)) - 1 / 1 copied.
2024-04-15 09:46:03,070 INFO  [pool-4-thread-1] mapred.Task (Task.java:commit(1421)) - Task attempt_local756486036_0001_r_000000_0 is allowed to commit now
2024-04-15 09:46:03,073 INFO  [pool-4-thread-1] output.FileOutputCommitter (FileOutputCommitter.java:commitTask(606)) - Saved output of task 'attempt_local756486036_0001_r_000000_0' to file:/tmp/results
2024-04-15 09:46:03,075 INFO  [pool-4-thread-1] mapred.LocalJobRunner (LocalJobRunner.java:statusUpdate(634)) - reduce > reduce
2024-04-15 09:46:03,076 INFO  [pool-4-thread-1] mapred.Task (Task.java:sendDone(1380)) - Task 'attempt_local756486036_0001_r_000000_0' done.
2024-04-15 09:46:03,080 INFO  [pool-4-thread-1] mapred.Task (Task.java:done(1276)) - Final Counters for attempt_local756486036_0001_r_000000_0: Counters: 24
	File System Counters
		FILE: Number of bytes read=2066186
		FILE: Number of bytes written=804487
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
	Map-Reduce Framework
		Combine input records=0
		Combine output records=0
		Reduce input groups=2
		Reduce shuffle bytes=144425
		Reduce input records=13129
		Reduce output records=2
		Spilled Records=13129
		Shuffled Maps =1
		Failed Shuffles=0
		Merged Map outputs=1
		GC time elapsed (ms)=88
		Total committed heap usage (bytes)=219152384
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Output Format Counters 
		Bytes Written=30
2024-04-15 09:46:03,092 INFO  [pool-4-thread-1] mapred.LocalJobRunner (LocalJobRunner.java:run(353)) - Finishing task: attempt_local756486036_0001_r_000000_0
2024-04-15 09:46:03,093 INFO  [Thread-24] mapred.LocalJobRunner (LocalJobRunner.java:runTasks(486)) - reduce task executor complete.
2024-04-15 09:46:03,371 INFO  [main] mapreduce.Job (Job.java:monitorAndPrintJob(1647)) -  map 100% reduce 100%
2024-04-15 09:46:03,372 INFO  [main] mapreduce.Job (Job.java:monitorAndPrintJob(1658)) - Job job_local756486036_0001 completed successfully
2024-04-15 09:46:03,400 INFO  [main] mapreduce.Job (Job.java:monitorAndPrintJob(1665)) - Counters: 30
	File System Counters
		FILE: Number of bytes read=3843490
		FILE: Number of bytes written=1464519
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
	Map-Reduce Framework
		Map input records=13130
		Map output records=13129
		Map output bytes=118161
		Map output materialized bytes=144425
		Input split bytes=79
		Combine input records=0
		Combine output records=0
		Reduce input groups=2
		Reduce shuffle bytes=144425
		Reduce input records=13129
		Reduce output records=2
		Spilled Records=26258
		Shuffled Maps =1
		Failed Shuffles=0
		Merged Map outputs=1
		GC time elapsed (ms)=118
		Total committed heap usage (bytes)=406323200
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=1777168
	File Output Format Counters 
		Bytes Written=30
```

Y se nos creara un directorio llamado results el cual tendrá la temperatura maxima de cada año :

```bash
root@1886eb9c334b:/tmp# cat results/part-r-00000
1901	317
1902	244
``` 
