% Spark Notes
% 
% 

Spark
=====

Spark Data Locality
-------------------

Q: How can I ensure that the executors come up on the right machines
in a cluster? How can I ensure data locality?

- The executors are assigned to machines when you create the
  `SparkContext`. 

- At that point the application does not know which HDFS files you are
  planning to process, and which machines the blocks of those files
  are located on. 

- So you have to provide this information to the `SparkContext`
  object.


Spark Data Locality
-------------------

- Here is how you do it:

        val locData = 
          InputFormatInfo.computePreferredLocations(
            Seq(
              new InputFormatInfo(conf, 
                classOf[TextInputFormat], new Path("myfile.txt")))
        val sc = new SparkContext(conf, locData)


Spark Data Locality
-------------------

- For more details on this see Sandy Ryza's presentation about Spark
  on YARN at 
  <http://spark-summit.org/wp-content/uploads/2014/07/Spark-on-YARN-A-Deep-Dive-Sandy-Ryza.pdf>.

Spark Streaming Checkpointing
-----------------------------

Q: How can I ensure that if my streaming driver fails it recovers its checkpointed state?

- Here is the code for doing this. 

- You have to create your `StreamingContext` and initialize all the
  receivers in a separate function, that returns the initialized
  `StreamingContext`.

        def initStreamingContext(): StreamingContext = {
            val ssc = new StreamingContext(...)   // Create context
            val lines = ssc.socketTextStream(...) // Create DStreams
            ...
            ssc.checkpoint(checkpointDirectory)   // Set checkpoint directory
            ssc
        }

Spark Streaming Checkpointing
-----------------------------

- Here is how you call this function from your main function:

        // Get StreamingContext from checkpoint data or create a new one
        val context = StreamingContext.getOrCreate(checkpointDirectory, 
          initStreamingContext _)

        // Do additional setup on context that needs to be done,
        // irrespective of whether it is being started or restarted
        context. ...

        // Start the context
        context.start()
        context.awaitTermination()


Spark Streaming Checkpointing
-----------------------------

- When your app runs `StreamingContext.getOrCreate` checks to see if
  your checkpoint directory exists or not.
 
- The first time your app runs this directory will not exist.

- When your directory does not exist it called `initStreamingContext`
  to initialize this directory.


Spark Streaming Checkpointing
-----------------------------

- When your app runs a second time after it terminated, it will check
  to see if this directory exists.

- This time the checkpoint directory will exist.

- When the checkpoint directory exists, Spark will recover the context
  that is stored in it, instead of recreating the context by calling
  the `initStreamingContext` function.


Spark Streaming Checkpointing
-----------------------------

- For more details see <http://spark.apache.org/docs/1.3.1/streaming-programming-guide.html#checkpointing>


Lambda Architecture
-------------------

Q: Can I join non-streaming RDD with a streaming RDD?

- Here is code to do this in a streaming app.

        // RDD containing user data.
        val userData = ssc.sparkContext.textFile(...) 

        // Join it in with incoming RDDs in transform or foreachRDD.
        val detailedTransactionDStream = 
          transactionDStream.transform(rdd => {
            rdd.join(userData).filter(...) 
            ...
          })


Lambda Architecture
-------------------

- For more details see <http://spark.apache.org/docs/1.3.1/streaming-programming-guide.html#transform-operation>. 


Spark Scala Script 
------------------

Q: How can I run a Scala script on Apache Spark without compiling it?

- Type the script in a file. Lets say this is called `myscript.scala`. 

- Make sure you put `System.exit(0)` at the end.

        val result = sc.parallelize(Array(1,2,3,4,5)).map(x => x*x).collect()
        result.foreach(x => println(x))
        System.exit(0)

- Next, submit it using `spark-shell`. 

        spark-shell -i myscript.scala


Spark SQL
---------

Q: How can I use SQL to query Apache Spark RDDs?

- Here are the steps. 

- Try these out by pasting them into a Scala Spark REPL (shell).

Spark SQL
---------

    // Create SQL context.
    val sqlContext = new org.apache.spark.sql.SQLContext(sc)

    // Create RDD containing sequence of JSON records.
    val jsonArray = Array(
      """ {"name":"Liz","address":{"city":"SF","state":"CA"}} """,
      """ {"name":"Bob","address":{"city":"SF","state":"CA"}} """,
      """ {"name":"Che","address":{"city":"LA","state":"CA"}} """)
    val jsonRdd = sc.parallelize(jsonArray)

    // Create Schema RDD and register as table.
    val employeesRdd = sqlContext.jsonRDD(jsonRdd)
    employeesRdd.registerTempTable("employees")

    // View the Schema RDD.
    employeesRdd.printSchema
    employeesRdd.collect

    // Use SQL to query the RDD.
    val sfEmployeesRdd = sqlContext.sql(
      "SELECT * FROM employees WHERE address.city = 'SF'")

    // This RDD will only contain the SF employees.
    sfEmployeesRdd.printSchema
    sfEmployeesRdd.collect


Receiver Flooding
-----------------

Q: In a streaming app if the receiver is getting data faster than the
DStreams are able to process it what happens?

- In this case the receiver will continue to receive data and turn
  them into RDDs every batch interval. 

- In the meantime the DStream will process the RDDs at its pace. 

- Over time the RDD queue in the DStream will build up. 

- This is okay once in a while. 

- But if it is happening continuously you want to fix this. 


Receiver Flooding
-----------------

- Some ways to fix this are: 

    - Increase number of partitions. 

    - Make the batch interval longer. 

    - Make the window's slide duration longer. 

    - Optimize the processing so it happens faster.


Batch Mode Failures
-------------------

Q: How does Spark handle failure of the driver, executors, tasks in batch mode?

- Failed tasks are re-attempted 4 times, like MapReduce. 

- Failed executors have their tasks partitions and tasks reassigned to
  other executors. Some partitions might need to be recomputed. 

- Failed drivers cause the job to fail. It needs to be restarted
  somewhere else. YARN and other frameworks can handle restarts.


Streaming Mode Failures
-----------------------

Q: What happens if the driver, executors, or receivers fail in a
streaming app?

- To understand this we first need to cover:

    - Reliable and unreliable receivers

    - Write ahead logs


Reliable and Unreliable Receivers
---------------------------------

Q: What is the difference between reliable and unreliable receivers? 

- Reliable receivers acknowledge receipt of data. If the receiver
  crashes before acknowledging the data the source resends the data to
  the next receiver.

- Kafka is an example of a reliable receiver.

- Unreliable receivers do not wait for acknowledgements.


Write Ahead Logs
----------------

Q: What are write ahead logs and how do they increase fault-tolerance? 

- Write ahead logs were added to Spark in version 1.2. 

- With write ahead logs the receiver writes incoming data into a write
  ahead log in the checkpoint directory. 

- When the driver fails and recovers it can pick up the data that was
  received but not processed.

- Enable write ahead logs by setting
  `spark.streaming.receiver.writeAheadLog.enable` to `true`.

Write Ahead Logs
----------------

Q: What are the downsides of write ahead logs?

- Write ahead logs lower the throughput of receivers. 

Write Ahead Logs
----------------

Q: What are some best practices to improve the performance of write
ahead logs?

- Run more receivers in parallel to improve throughput.

- Disable replication of received data by setting storage level for
  input stream to `StorageLevel.MEMORY_AND_DISK_SER`.


Persisting Received Data
------------------------

Q: What are the stages that received data goes through?

- Memory --> WAL --> RDD --> Checkpointed RDD


Receiver and Driver Failure
---------------------------

Q: What happens when the receiver or the driver fail?

WAL         Receiver      Receiver Fails          Driver Fails 
---         --------      --------------          ------------ 
Disabled    Unreliable    Unreplicated data lost  Unreplicated + uncheckpointed data lost
Disabled    Reliable      Zero data lost          Uncheckpointed data lost
Enabled     Unreliable    Unreplicated data lost  Data not in WAL lost
Enabled     Reliable      Zero data lost          Zero data lost


Receiver and Driver Failure
---------------------------

- For details see here.

- <https://spark.apache.org/docs/latest/streaming-programming-guide.html#fault-tolerance-semantics>

- <https://spark.apache.org/docs/latest/streaming-custom-receivers.html#receiver-reliability>


Killing Spark Jobs
------------------

Q: How do you kill a long-running Spark Streaming app?

- Through the execution framework. 

- For example in YARN you would type,

        yarn application -kill APP_ID


Misc Questions
==============


Debugging
---------

Q: How can you see the generated spark DAG or perhaps can you use the
generated DAG to help you debug or optimize your code

- You can use `toDebugString` to see the lineage of an RDD.

- There is work in progress to build a nicer visual DAG tool.

- <https://issues.apache.org/jira/browse/SPARK-975> is tracking this
  issue.


Debugging
---------

Q: How can you profile your code to see where the hot spots are?
Looking at the yarn output is to coarse grained 

- The Application UI (at port 4040) shows you the sizes of the
  partitions and which tasks are working harder than others.


Debugging
---------

Q: How can you hook into "Before" and "After", when an executor starts
its part of the job? You might want to have code run on the executor
to take debug info and at the end send them to the driver or write to
HDFS (somewhat like I suspect accumulators work). (equivalent to
Hadoop setup/cleanup). One example: we have had issues where the
executors are not setup the same with respect to default java
settings, it would be good to verify the java setting at the start of
the executor job.

- `mapPartitions` handles this use case.


Code
----

Q: When executing code on the executor, it seems to execute the "main"
part of the program. When does it stop executing and start waiting for
commands from the driver.

- Not sure I understand. 

- The main function is executed on the driver.

- The functions passed to the transformations (such as `map` and
  `filter`) are executed on executors. 


Code
----

Q: How do you read in local files (with respect to sparkSubmit)  in
the executors. If I want to pass in a local file to the spark program
that is on the same machine that did sparkSubmit (i.e. a config file),
how do I get that local file copied to the executors so they can read
it as well. Currently I copy the file to HDFS beforehand and pass in
the HDFS path. I would prefer not to copy to HDFS first.

- You can use `--files` to pass files to the executors.

- If you specify `--files localtest.txt#appSees.txt` and this will
  upload the file you have locally named `localtest.txt` into HDFS but
  this will be linked to by the name `appSees.txt` and your
  application should use the name as `appSees.txt` to reference it
  when running on YARN.

    <https://spark.apache.org/docs/latest/running-on-yarn.html#important-notes>

Code
----

Q: If I create a scala companion object, and create a global variable,
will it be available to the executors.

- The global object will exist in the driver and will not be available
  to the executors.

- However, if you reference it in the functions passed into
  transformations the variable will get serialized as part of a
  closure and will be available in the tasks on the executors.

- Another option is to use *Broadcast Variables*.

Partitioning
------------

Q: How do you pick the best partition size and can it be done
dynamically? For example in our case, too few partitions will result
in GC memory errors. Too many will slow things down. If we let spark
do it, the partitions will grow to very large numbers (greater than
50K partitions) and the processes will die.  Currently we try to do it
dynamically, but our options we use are based on trial and error.

- Consider using `repartition` and `coalesce` to even out the
  partitions.

- `coalesce` is like `repartition` except it is a narrow operation,
  while `repartition` is a wide operation.

- However, `coalesce` only reduces the number of partitions.

- You want your partitions to be roughly the size of blocks, so about
  100MB.

- You can have more partitions than cores. Spark can handle tasks as short as 200 ms.

- For more details see <https://spark.apache.org/docs/latest/tuning.html#level-of-parallelism>.

Partitioning
------------

- If you are doing joins then use `partitionBy` to partition your
  PairRDDs by keys, and then `cache` or `persist` the RDD.
  
- This will significantly reduce the amount of data that is shuffled
  during joins.

- For more details see the *Data Partitioning (Advanced)* section in
  chapter 3 of *Learning Spark*.

Executors
---------

Q: What is the relationship between executors, cores, threads.

- Cores equal threads equal tasks that can run in parallel.

- Cores are also used for receivers. 

- Executors are JVMs, and cores/threads/tasks live inside executors.


Executors
---------

Q: Do cores know about each other and therefore can optimize
operations with out sending data to disk?

- Cores in the same executor are in the same JVM so they can
  communicate with each other through shared global variables. 

- However, this is error-prone and dangerous.

- The safest approach is to imagine each cores in a bubble.

- Cores on different machines do no know about each other.


Executors
---------

Q: How does one determine the best number of cores/executors/threads?
Is there a way to determine the number of executors (approximate) you
would need given the size of data input and the size of executor.

- If you want to maximize parallelism you can set the number of cores
  for a job to be the number of partitions in the input files.

- Think of cores as the number of gates at a train station from which
  people are flowing out. The people in this analogy are the tasks.

- The number of executors does not matter as much as the number of
  cores. 
  
- The number of cores is limited by what is available on each worker
  machine, which the number of executors is limited by how many worker
  machines there are.


Quizzes
=======

Quiz: RDDs
----------

- Question: What is an RDD?

    - An RDD is a *Resilient Distributed Dataset*.

- Question: What is the difference between transformation and action?

    - A transformation defines a new RDD in terms of previously
      defined RDDs. It does not force the evaluation of the RDD.

    - An action forces the RDD to get evaluated, frequently by
      extracting a concrete data object out of it.


- Question: Given `r = sc.parallelize([1,2,3])` is it possible to
  modify the RDD pointed to by `r` to add another element to it?

    - No, it is not possible. RDDs are immutable. You can define a new
      RDD but you cannot modify an existing RDD.

- Question: What will this output?

        sc.parallelize([1,3,2,2,1]).distinct().collect()

    - `[2, 1, 3]`

- Question: What will this output?

        sc.parallelize([1,3,2,2,1]).distinct().sortBy(lambda x: x).collect()

    - `[1, 2, 3]`

- Question: What will this output?

        val lines = Array("hello world", "this is another line", "this is another")
        sc.parallelize(lines).map(line => line.split("\\W+")).count()

    - 3

- Question: What will this output?

        val lines = Array("hello world", "this is another line", "this is another")
        sc.parallelize(lines).flatMap(line => line.split("\\W+")).count()

    - 9

- Suppose you have stock market data from Yahoo! for AAPL
    from <http://finance.yahoo.com/q/hp?s=AAPL+Historical+Prices>. The
    data is in CSV format and has these values.

    Date         Open     High     Low      Close    Volume       Adj Close
    ----         ----     ----     ---      -----    ------       ---------
    11-18-2014   113.94   115.69   113.89   115.47   44,200,300   115.47
    11-17-2014   114.27   117.28   113.30   113.99   46,746,700   113.99

    Here is what the CSV looks like:
    
        csv = [
          "#Date,Open,High,Low,Close,Volume,Adj Close\n",
          "2014-11-18,113.94,115.69,113.89,115.47,44200300,115.47\n",
          "2014-11-17,114.27,117.28,113.30,113.99,46746700,113.99\n",
        ]

    Lets find the date on which the price was the highest. 

- Question: What two fields would you need to extract?

    - *Date* and *Adj Close*.

    - We want to use *Adj Close* instead of *High* so our calculation
      is not affected by stock splits.

- Question: What field would need to be the key?

    - *Adj Close*.

- Question: What are the sequence of operations that you would need to perform.

    - Use `textFile` to read the file.

    - Use `filter` to remove the header line.
        
    - Use `map` to split each row into fields.

    - Use `map` to extract *Adj Close* and *Date*.

    - Use `sortByKey(false)` to sort descending on *Adj Close*.

    - Use `take(1)` to get the highest value.

- Here is full source.

        csv = [
          "#Date,Open,High,Low,Close,Volume,Adj Close\n",
          "2014-11-18,113.94,115.69,113.89,115.47,44200300,115.47\n",
          "2014-11-17,114.27,117.28,113.30,113.99,46746700,113.99\n",
        ]
        sc.parallelize(csv) \
          .filter(lambda line: not line.startswith("#")) \
          .map(lambda line: line.split(",")) \
          .map(lambda fields: (float(fields[-1]),fields[0])) \
          .sortByKey(False) \
          .take(1)

- Here is the program for finding the high of any stock.

        import urllib2
        import re

        def get_stock_high(symbol):
          url = 'http://real-chart.finance.yahoo.com' + \
            '/table.csv?s='+symbol+'&g=d&ignore=.csv'
          csv = urllib2.urlopen(url).read()
          csv_lines = csv.split('\n')
          stock_rdd = sc.parallelize(csv_lines) \
            .filter(lambda line: re.match(r'\d', line)) \
            .map(lambda line: line.split(",")) \
            .map(lambda fields: (float(fields[-1]),fields[0])) \
            .sortByKey(False)
          return stock_rdd.take(1)

        get_stock_high('AAPL')


Quiz: HDFS
----------

- Question: How many times is a block replicated by default?

    - 3

- Question: Suppose you have file that is 200 MB. How many HDFS blocks
  will it be split up into?

    - 2

- Question: Suppose you have a file that is 4K. How many HDFS blocks
  will it be split up into?

    - 1

- Question: Will the 4K block take up as much space as a 128 MB 
  block?

    - A block is stored as a Unix file on a DataNode. 
    - So it will take up precisely as much space as it needs. 
    - This is different from the way file system blocks behave.


Quiz: RDDs and Partitions
-------------------------

- Question: In the following code how many times will the lambda
  passed to `mapPartitions` be called, and what will this output?

        sc.parallelize([10,11,12,13,14], 2) \
          .mapPartitions(lambda iter: iter).count()

    - The lambda passed to `mapPartitions` will be called twice,
      because there are 2 partitions.

    - The output will be 5.

- Question: In the following code how many times will the lambda
  passed to `mapPartitions` be called, and what will this output?

        sc.parallelize([10,11,12,13,14], 2) \
          .mapPartitions(lambda iter: iter).take(1)

    - The lambda passed to `mapPartitions` will be called once,
      because only 1 partition needs to be processed to produce the
      result.

    - The output will be 10.

- Question: In the following code to get the highest price of a stock,
  which transformations are wide and which are narrow? 
  
        import urllib2
        import re

        def get_stock_high(symbol):
          url = 'http://real-chart.finance.yahoo.com' + \
            '/table.csv?s='+symbol+'&g=d&ignore=.csv'
          csv = urllib2.urlopen(url).read()
          csv_lines = csv.split('\n')
          stock_rdd = sc.parallelize(csv_lines) \
            .filter(lambda line: re.match(r'\d', line)) \
            .map(lambda line: line.split(",")) \
            .map(lambda fields: (float(fields[-1]),fields[0])) \
            .sortByKey(False)
          return stock_rdd.take(1)

        get_stock_high('AAPL')


    - The narrow transformations are: `filter`, `map`,
      and the second `map`.

    - The wide transformation is: `sortByKey`.

- Question: How many stages are there in the code?

    - There are two stages.

    - The first stage ends before `sortByKey`.

    - The second stage ends with the action `take`.


Quiz: Persistence
-----------------

- Question: What is the difference between `rdd.cache()` and
  `rdd.persist(MEMORY_ONLY)`?

    - There is no difference.

- Question: What is the difference between `MEMORY_ONLY` and
  `MEMORY_AND_DISK` and `DISK_ONLY`?

    - When you persist an RDD using `MEMORY_ONLY` then the RDD is not
      cached if sufficient memory is not available.

    - When you persist an RDD using `MEMORY_AND_DISK` the RDD spills
      to disk once memory fills up. 
      
- Question: What is the difference between `MEMORY_AND_DISK` and
  `MEMORY_AND_DISK_2`?

    - When you persist an RDD using `MEMORY_AND_DISK_2` the RDD is
      persisted on two nodes. This way if a node goes down you have a
      copy of the partitions on that node.

- Question: What is the difference between `MEMORY_AND_DISK` and
  `MEMORY_AND_DISK_SER`?

    - When you persist an RDD using `MEMORY_AND_DISK` the objects in
      memory are kept as live Java objects.

    - When you persist an RDD using `MEMORY_AND_DISK_SER` the objects
      in memory are serialized.

    - If an object spills to disk it is serialized in both cases.

- Question: In the following code will `rdd` get cached?

        rdd = sc.parallelize([1,2,3,4])
        rdd.cache()

    - `rdd` will not get cached until an *action* such as `count()` or
      `collect()` is called on the RDD.

- Question: What is the difference between `rdd.checkpoint()` and
  `rdd.persist(DISK_ONLY)`? 

    - `rdd.persist(DISK_ONLY)` will save the RDD to disk, while
      `rdd.checkpoint()` will save the RDD to HDFS.

    - `rdd.checkpoint()` discards the lineage, while
      `rdd.persist(DISK_ONLY)` will not discard the lineage. It might
      need the lineage to recreate a partition if a node is lost.

    - `rdd.checkpoint()` evaluates all the partitions in the RDD,
      while `rdd.persist()` will only evaluate the partitions that it
      needs to complete the action.

Quiz: Streaming
---------------

- Question: If you have `dstream.print()` does it print immediately?
  Or does it print later?

    - `dstream.print()` will not print immediately.

    - It will print later after `ssc.start()` is called and when the
      RDDs start entering the streaming app.

    - `dstream.print()` is an instruction to the streaming context to
      print the RDD when it comes in.


- Question: What happens if your streaming app has a batch interval
  duration of 2 seconds, but it takes it 3 seconds to process each RDD.

    - In this case the RDDs will accumulate in the streaming app.

    - This is an undesirable situation and if it occurs you will need
      to resolve it.

    - To resolve it, either lengthen the batch interval to longer than
      the processing time, or shorten the processing interval by
      optimizing the code, or by increasing the number of partitions.

- Question: What is the difference between `transform` and
  `foreachRDD`.

    - `transform` is a generic transform for DStreams.

    - `foreachRDD` is a generic output operation for DStreams.

- Question: How realtime is Spark Streaming? What is its minimum
  latency?

    - Spark Streaming has second scale latency.

    - This means that it will detect an event roughly a second after
      it occurs.

    - Now you can shorten this to some by tuning the batch interval,
      but you can only shorten it down to how long it takes to process
      an RDD.

Quiz: Broadcast Variables
-------------------------

- Question: What is the advantage of using broadcast variables over
  closures?

    - Broadcast variables are transmitted to each executor once per
      application. 
      
    - Closures are transmitted to each task.

    - Broadcast variables are transmitted more efficiently from the
      driver to the executors because they use the Cornet P2P algorithm.

    - Broadcast variables are about 3x faster than the distributed
      cache.

Quiz: Accumulators
------------------

- Question: Is it possible to look at the value of an accumulator
  on the executors?

    - It is not possible to look at the value of an accumulator on the
      executors. 
      
    - If you look at the value of an accumulator in an executor it
      will throw an exception.


Spark Resources 
===============

Spark Reading List
------------------

- Details of how Spark runs on YARN    
  <http://spark-summit.org/wp-content/uploads/2014/07/Spark-on-YARN-A-Deep-Dive-Sandy-Ryza.pdf>    

Books on Spark
--------------

- Learning Spark: Lightning-Fast Big Data Analytics    
  By Holden Karau, Andy Konwinski, Patrick Wendell, Matei Zaharia    
  Publisher: O'Reilly Media, June 2014    
  <http://shop.oreilly.com/product/0636920028512.do>    
  Introduction to Spark APIs and underlying concepts.

- Spark Knowledge Base    
  By Databricks, Vida Ha, Pat McDonough    
  Publisher: Databricks    
  <http://databricks.gitbooks.io/databricks-spark-knowledge-base>    
  Spark tips, tricks, and recipes.    

- Spark Reference Applications    
  By Databricks, Vida Ha, Pat McDonough    
  Publisher: Databricks    
  <http://databricks.gitbooks.io/databricks-spark-reference-applications>    
  Best practices for large-scale Spark application architecture.
  Topics include import, export, machine learning, streaming.

Learning Scala
--------------

- Scala for the Impatient    
  by Cay S. Horstmann    
  Publisher: Addison-Wesley Professional, March 2012    
  <http://www.amazon.com/Scala-Impatient-Cay-S-Horstmann/dp/0321774094>    
  Concise, to the point, and contains good practical tips on using Scala.    

Video Tutorials
---------------

- Spark Internals    
  By Matei Zaharia (Databricks)    
  <https://www.youtube.com/watch?v=49Hr5xZyTEA>    

- Spark on YARN    
  By Sandy Ryza (Cloudera)    
  <https://www.youtube.com/watch?v=N6pJhxCPe-Y>    

- Spark Programming    
  By Pat McDonough (Databricks)    
  <https://www.youtube.com/watch?v=mHF3UPqLOL8>    


Community
---------

- Community    
  <https://spark.apache.org/community.html>    
  Spark's community page lists meetups, mailing-lists, and upcoming
  Spark conferences.

- Meetups    
  <http://spark.meetup.com/>    
  Spark has meetups in the Bay Area, NYC, Seattle, and most major
  cities around the world.

- Mailing Lists    
  <https://spark.apache.org/community.html>    
  The user mailing list covers issues and best practices around using
  Spark. The dev mailing list is for people who want to contribute to
  Spark.


