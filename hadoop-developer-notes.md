% Hadoop Developer Notes
% May, 2015
% Asim Jalis

Hadoop
======

Big Data
--------

Q: How big is Big Data?

- Here are some numbers on how much data companies store on Hadoop
  clusters from 2014.

    Company     Hadoop Cluster
    -------     --------------
    Yahoo!      350 PB
    Facebook    300 PB
    eBay        150 PB

    <http://lintool.github.io/my-data-is-bigger-than-your-data/>


Real-World Applications of Hadoop
---------------------------------

Q: What are some real-world applications of Hadoop?

- Yahoo! uses Hadoop to detect spam.

    <http://www.slideshare.net/hadoopusergroup/mail-antispam>

- Orbitz found Mac users will more for hotels than PC users. 

    <https://gigaom.com/2011/11/22/big-data-reveals-mac-users-book-pricier-hotels/>

- Case studies of how banks, telcos, and other companies use Hadoop:

    <http://blog.cloudera.com/wp-content/uploads/2011/03/ten_common_hadoopable_problems_final.pdf>

- New York Times converted 11 million TIFF files to PDF in less than 24 hours. 

    <http://open.blogs.nytimes.com/2007/11/01/self-service-prorated-super-computing-fun>

- Netflix uses Hadoop to make recommendations. 

    <http://gigaom.com/2012/06/14/netflix-analyzes-a-lot-of-data-about-your-viewing-habits>

- Target uses Data Science to predict pregnancies. 

    <http://www.nytimes.com/2012/02/19/magazine/shopping-habits.html>

- Capital One fraud researchers themselves committed fraud using Hive and Hadoop.

    <http://www.bloombergview.com/articles/2015-01-23/capital-one-fraud-researchers-may-also-have-done-some-fraud>


Mounting HDFS
-------------

Q: How can I mount HDFS like a regular file system and then do
autocomplete on it?

- You can do this through FUSE DFS. Here are the steps.

        # Install hadoop-hdfs-fuse
        sudo yum install hadoop-hdfs-fuse

        # Find NameNode URI from /etc/hadoop/conf/core-site.xml

        # Mkdir mount point.
        sudo mkdir -p /mnt/hdfs

        # Mount HDFS using NameNode URI.
        sudo hadoop-fuse-dfs hdfs://0.0.0.0:8020 /mnt/hdfs

        # You should see HDFS underneat /mnt/hdfs.
        cd /mnt/hdfs
        ls -l


Cloudera Manager Tutorial
-------------------------

Q: Can I get more information and training on Cloudera Manager?

- Cloudera has a 10-video free online class on Cloudera Manager.

    <http://www.cloudera.com/content/cloudera/en/training/library/cloudera-manager-training.html>


Benchmarks
----------

Q: How can I benchmark my cluster?

- *Terasort* is a commonly used benchmark. Terasort can generate large
  amounts of data and then sort it using MapReduce.

- Set up the path.

        EX_JAR=/usr/lib/hadoop-mapreduce/hadoop-map-reduce-examples.jar

- Generate 50 100-byte rows of data with 10 mappers (default is 2).

        hadoop -D mapreduce.job.maps 10 \
            jar $EX_JAR teragen 50 out-gen

- Sort data.

        hadoop jar $EX_JAR terasort out-gen out-sort

- Validate data.

        hadoop jar $EX_JAR teravalidate \
            out-sort out-validate

- For information on the current winners see:

    <http://sortbenchmark.org>


Cloudera QuickStart VM
----------------------

Q: How can I get a Cloudera QuickStart VM to experiment with Cloudera
Manager?

- See here for instructions and the download link.

    <http://www.cloudera.com/content/cloudera-content/cloudera-docs/DemoVMs/Cloudera-QuickStart-VM/cloudera_quickstart_vm.html>


Setting Up Eclipse For Hadoop
-----------------------------

Q: How can I create a project from scratch and pull in all the
required Hadoop dependencies?

- This QuickStart video shows how to create a brand new MapReduce
  project in Eclipse from scratch. 
  
- The steps start at 4:51.

    <http://blog.cloudera.com/blog/2013/08/how-to-use-eclipse-with-mapreduce-in-clouderas-quickstart-vm/>


Local Mode in Eclipse
---------------------

Q: How can I run my MapReduce job in Eclipse in local mode so that I
can debug it?

- By default Hadoop runs in local mode. 

- So if you follow the steps in the quickstart video you will run
  Eclipse in the local mode. This requires no special
  configuration.

- The reason is that the configuration in `/etc/hadoop/conf` is
  read by the `hadoop jar` command because it is in the classpath.
  If you keep it out of the classpath in Eclipse then it will run
  in local mode.


Testing Streaming Jobs
----------------------

Q: How can I test streaming API mappers and reducers without Hadoop?

- Here is how to do it:

        cat shakespeare/* | ./mapper.py | sort | ./reducer.py > output


Python MRJob
------------

Q: Is there an easy way to write MapReduce in Python without dealing
with all the low-level details?

- Take a look at the *MRJob* project.

    <https://github.com/Yelp/mrjob>


Large Data Sets
---------------

Q: Where can I get my hands on large data sets to play with?

- <http://kaggle.com> runs Data Science competitions many with prizes
  anywhere between a few hundred dollars to $25k and more.

- The site also publishes datasets for these competitions.


Configuration Parameters
------------------------

Q: Where can I get a list of all the default Hadoop configuration
parameters?

- Scroll down to the bottom of this page. 

- See bottom left corner for default Hadoop configuration values.

    <http://archive-primary.cloudera.com/cdh5/cdh/5/hadoop/>


Translate SQL to Hive
---------------------

Q: Is there a way to translate SQL to Java code that runs MapReduce?

- The YSmart project from Ohio State will do this translation for you.

    <https://github.com/YSmart/YSmart>


How to Shorten Oozie XML
------------------------

Q: Can I break down a big Oozie job into smaller XML files and then
include them into a larger XML workflow file?

- Here are the details for how to do this:

    <http://blog.cloudera.com/blog/2013/11/how-to-shorten-your-oozie-workflow-definitions/>


Quizzes
=======

Quiz: HDFS, MapReduce, Pseudo Mode
----------------------------------

> - Question: If you have a file that is 100 MB how many blocks will it
>   contain?
> 
>     - 2 blocks.
> 
> - Question: If you run a MapReduce job on a 100 MB file how many
>   mappers will run? How many reducers will run (by default)?
> 
>     - 2 mappers and 1 reducer.
> 
> - Question: Are the intermediate output files of the Mappers stored in
>   HDFS or on the local disk? Why?
> 
>     - The intermediate output files are stored on local disk. HDFS
>       would be expensive in terms of performance.
> 
> - Question: In a heterogenous cluster is it a good idea to have
>   Speculative Execution enabled?
> 
>     - Not a good idea because any task assigned to a slow machine
>       is likely to get done twice--the second time by a faster machine.
>   
> - Question: What are the 3 Java files that the developer has to
>   provide to specify a MapReduce job?
>
>     - The driver, the mapper, and the reducer.
> 
> - Question: How many JVMs/daemons are there in a pseudo-distributed
>   cluster vs full cluster?
> 
>     - All the daemons in both.
> 
> - Question: How many machines are there in a pseudo-distributed vs
>   full cluster?
> 
>     - 1 in pseudo and N in full.
> 
> - Question: Should we increase the replication factor on large files
>   since the odds of a block failure are higher?
> 
>     - No. The failures will occur at different times.
> 
> - Question: You have to write a MapReduce job that counts cards to 
>   find the number of hearts, diamonds, clubs, and spades. What should
>   the intermediate key and value be?
> 
>     - The key should be *suit*, the value *1*, for each card.
> 
> - Question: You have to write a MapReduce job that processes sales 
>   records and sorts them by amount. What should the intermediate 
>   key and value be?
> 
>     - The key should be the *sales amount* (or whatever you want to 
>       sort on), the value should be the *sales record*.
> 
> - Question: What is the role of the InputFormat? What is the 
>   behavior of the default InputFormat?
> 
>     - The InputFormat determines the InputSplits that the mappers
>       will process, and converts the bytes into key-value
>       pair records.
>     - The default InputFormat parses its input into a sequence of
>       byte offsets and lines of text.
> 
> - Question: What is the role of the OutputFormat? What is the 
>   behavior of the default OutputFormat?
> 
>     - The OutputFormat determines how the output of MapReduce 
>       is written out. 
>     - By default, it writes out the string representations of 
>       the key and value, separated by tabs.


Quiz: MapReduce
---------------

> - Question: How can you tell a class belong to the Old API and not the
>   New API?
> 
>     - It has *mapred* instead of *mapreduce* in its package name.
> 
> - Question: Can the *ToolRunner* be used in a non-MapReduce program?
> 
>     - Yes. For example `hadoop fs` uses ToolRunner to parse options
>       such as `-D` and `-fs`. 
>       
>     - Use the ToolRunner API any time you want a program to parse
>       standard Hadoop options and feed them into a Hadoop
>       Configuration object.
> 
> - Question: To create a combiner class do I have to extend the
>   `Combiner` base class or the `Reducer` base class?
> 
>     - The `Reducer` base class. The combiner is a reducer that runs on
>       the mapper machine.
> 
> - Question: What is the benefit of using the Combiner?
> 
>     - The combiner shrinks the intermediate data and saves disk space
>       as well as network bandwidth when the intermediate data is
>       copied to the reducers.
> 
> - Question: True or False: HDFS cannot be accessed from within a
>   `map` function.
> 
>     - False. HDFS can be accessed from outside the cluster and from
>       within the cluster, by using the `FileSystem` API.
> 
> - Question: How many machines and JVMs are used when running the
>   Hadoop MapReduce program in LocalJobRunner mode?
> 
>     - 1 machine and 1 JVM. All the daemons spin up in the same JVM and
>       then terminate when the program ends.
> 
> - Question: How can we configure Eclipse to come up in LocalJobRunner
>   mode?
> 
>     - By default Hadoop comes up in LocalJobRunner mode if it cannot
>       find its config files. To ensure this happens on Eclipse, make
>       sure that `/etc/hadoop/conf` is not in Eclipse's classpath.
> 
> 
> - Question: Can I set a combiner on MRUnit's `MapReduceDriver`?
> 
>     - Yes. MRUnit's `MapReduceDriver` is a full-fledged test harness
>       for running MapReduce programs. It lets you specify the combiner
>       and other configuration options that you specify on the `Job`
>       object.
> 

Quiz: Logging, Counters, Map-Only Jobs
--------------------------------------

> - Question: How do you configure a job to run as a map-only job?
>
>     - Set the number of reducers to 0.
> 
> - Question: How many output files does a map-only job produce?
>
>     - The number of output files will equal the number of mappers 
>       which equals the number of input blocks.
>
> - Question: How many output files does a regular MapReduce job produce?
>
>     - The number of output files will equal the number of reducers
>       which is a configuration parameter. By default this is 1.
>
> - Question: To add a new custom command-line option to your program do you need to edit the Driver code?
>
>     - If you use the ToolRunner API that passes the configuration 
>       parameters directly from the command line to the mappers and 
>       reducers. The Driver code does not need to be modified.
>
> - Question: Why is it a better to use logging instead `println`? Why
>   is it important to not to be too chatty in logs in Big Data programs?
> 
>     - It is better to use logging because you can disable logging from
>       the command line or through properties. It is important to not
>       be too chatty in logs to avoid producing logs that are as large
>       as the Big Data input datasets.
> 
> - Question: Why is it a best practice to reuse Writable objects in
>   MapReduce programs?
> 
>     - Reusing Writable objects in MapReduce programs delays triggering
>       garbage collection which improves the performance of mappers and
>       reducers.
> 
> - Question: Why can't we use counters for Word Count?
> 
>     - Counters can only be used to count 120 different things. Word
>       Count does not know in advance how many unique words it will
>       need to count the frequencies of.
> 
> - Question: Does Shuffle and Sort run in map-only jobs?
> 
>     - No, Shuffle and Sort does not run in map-only jobs.

Quiz: Partitioners and Reducers
-------------------------------

> - Question: Does a partitioner have to send all records with the same
>   key to the same reducer?
> 
>     - No. A partitioner can send a key-value pair to any reducer
>       depending on the logic in its *getPartition* function.
> 
> - Question: What are the benefits of using multiple reducers? What are
>   the downsides?
> 
>     - Benefits:
>         
>         - Each reducer has to store less intermediate data on its
>           local disk.
> 
>         - The reduce step is run in parallel.
> 
>     - Downsides:
> 
>         - When you have multiple reducers the output of each reducer
>           will be sorted but if the reducer output files are
>           concatenated together the result will not be sorted.
> 
>         - If the partitioner is not fair some reducers might get hit
>           with more data skew than others.
> 
> - Question: True or false: Both the key and the value types have to
>   implement the *WritableComparable* interface.
> 
>       - False. The key has to implement *WritableComparable*. The
>         value only has to implement *Writable*.
> 
> - Question: True or false: If the mappers output key-value pair contains JSON in the
>   value, this requires a custom Writable.
> 
>       - False. JSON is represented as a string and can be stored in
>         Hadoop's built-in *Text* object.

Quiz: Data Types
----------------

> - Question: Does the split size have to be exactly equal to an HDFS
>   block or can it be smaller or larger?
> 
>     - No, the split size can be smaller or larger than the HDFS block
>       size.
> 
> - Question: Does a compression block equal 128MB?
> 
>     - No, the compression block is 1,000,000 bytes or slightly larger.
> 
> - Question: How does the `LineRecordReader` handle records split
>   across input splits? If a record is split across mappers M1 and M2,
>   which one will read it?
> 
>     - Each record reader reads past the end of its input split to
>       complete the record. Each record reader, except for the first,
>       skips its first record since that is read by the previous record
>       reader.
> 
>     - M1 will read the record. M2 will skip it.
> 
> - Question: Why is gzip not a good compression format for Hadoop? What
>   alternative is preferred?
> 
>     - Gzip is not a good compression format for Hadoop because it is
>       not splittable. Snappy which is splittable is a better
>       compression format.
> 

Class Notes
===========

Grouping Sales
--------------

Q: How can MapReduce be used to group sales by region?

    112,CA,200
    113,NY,150
    114,CA,300
    115,CA,400
    116,NY,250
      |
      | RecordReader
      v
    0,112,CA,200
    100,113,NY,150
    200,114,CA,300
    300,115,CA,400
    400,116,NY,250
      |
      | Map
      v
    (CA,200)
    (NY,150)
    (CA,300)
    (CA,400)
    (NY,250)
      +--------------+
      | Shuffle+Sort | 
      +--------------+
      v              v
    (CA,200)     (NY,150)
    (CA,300)     (NY,250)
    (CA,400)       |
      |            |
      | Reduce     | Reduce
      v            v
    (CA,900)     (NY,400)



IP Frequency
------------

Q: Find the IP addresses with the highest frequency in a web log.

    1.2.3.5 - - - - 
    1.2.3.6 - - - - 
    1.2.3.7 - - - - 
    1.2.3.5 - - - - 
      | 
      | Map
      v
    (1.2.3.5,1)
    (1.2.3.5,1)
    (1.2.3.6,1)
    (1.2.3.7,1)
      |
      | Reduce
      v
    (1.2.3.5,2)
    (1.2.3.6,1)
    (1.2.3.7,1)


Highest Sales
-------------

Q: Find the highest sales transaction.

    Alice 300
    Bob 200
    Alice 500
    Alice 600
    Bob 400
    Charlie 200
      |
      | Map
      v
    (200,Bob)
    (200,Charlie)
    (300,Alice)
    (400,Bob)
    (500,Alice)
    (600,Alice)
      |
      | Reduce
      v
    (200,Bob)
    (200,Charlie)
    (300,Alice)
    (400,Bob)
    (500,Alice)
    (600,Alice)

MapReduce Data Types
--------------------

Q: What is the relationship between the key and value types used
through the MapReduce pipeline?

    HDFS: Raw Bytes
      |
      | InputFormat/RecordReader
      v
    K1,V1
      |
      | Map 
      v                 
    K2,V2 -->Combiner -+
      |                |
      | Reduce<--------+
      v
    K3,V3
      | 
      | OutputFormat
      v
    HDFS: Raw Bytes


How Many Reducers
-----------------

Q: How can you estimate how many reducers to use for a job?

- Look at the size of the input file.

        Input Data Size   = 268140 

- Look at the size of the intermediate data (*Map Output Bytes*).

        Intermediate Data = 301272

- Find the ratio between them. In this example, the ratio is about
  1:1.

- This means the number of reducers should equal the number of
  mappers.

- If the ratio between input data and intermediate data was 10:1, then
  this would mean that for 10 mappers you would need 1 reducer.


Sequence Files
--------------

Q: What is a common use case for sequence files? 

- If you have a chain of MapReduce jobs you can use the sequence file
  format for the files produced by Job 1 and consumed by Job 2.

- This way Job 2 does not have to parse the data a second time.


        Text
          |
          v
        Map 1
          |
          v
        Reduce 1 (write as Seq File)
          |
        Seq File on HDFS
          |
          v
        Map 2 (read as Seq File)
          |
          v
        Reduce 2


Chained Jobs
------------

Q: How can jobs be chained together?

- The simplest way is to run the jobs one after the other using
  separate `hadoop jar` commands. 
  
        hadoop jar foo.jar Driver1 input1 output1
        hadoop jar foo.jar Driver2 output1 output2


Input Splits
------------

Q: What does an input split contain?

- Input Split:
    - Path
    - Start 
    - Length


Partial Records
---------------

Q: How does the default record reader handle records that are split
across multiple input splits?

- Each mapper except the first one skips its first record.

- Each mapper completes its record even if it has to go past the end
  of its input split.

- If a mapper's record ends at the end of the input split it still
  reads the next record from the next mapper's input split.

- For example,

        Mappers [   1       2       3   ]
        Blocks  [       |       |       ]
        Records [1 |2 |3|4 |5 |6 | 7 | 8]

- Mapper 1 will read records 1, 2, 3, 4. 

- Mapper 2 will read records 5, 6.

- Mapper 3 will read records 7, 8.


Compression
-----------

Q: What is the difference between record and block compression?

- There are 2 types of compression:

    - Record: Compress each key-value pair.

    - Block: Compress all the key-value pairs in 1,000,000 of data and
      complete last record.

Q: What is the default compression code, and what is the recommended
compression codec?

- The default compression codec is `DefaultCodec`.

- This uses zlib for compression.

- The recommended compression code is `SnappyCodec`.


TF IDF
------

Q: Demonstrate the TF-IDF algorithm using a concrete example.

    Books
    H: Hadoop Definitive Guide
    B: Intro to Beekeeping

    Words

    N = Total number of documents = 2
    n = Number of documents containing word
    TF-IDF = TF * IDF

    TF       H        B    n   log2(N/n)     H     B
    hive    10      100    2           0     0     0  
    bees     0      200    1           1     0     200
    hadoop 100        0    1           1     100   0

    Job 1:

    Mapper:
    (H,hive),1

    Reducer:
    (hive,H),10
    (hadoop,H),100
    (hive,B),100
    (bees,B),200

    Job 2:

    Mapper:
    (bees,(B,200,1))
    (hadoop,(H,100,1))
    (hive,(H,10,1))
    (hive,(B,100,1))

    Reducer:
    (bees,B),(200,1))
    (hadoop,H),(100,1))
    (hive,H),(10,2))
    (hive,B),(100,2))

    Job 3:

    (bees,B),(200*(log(2/1)))
    (hadoop,H),(100*log(2/1))
    (hive,H),(10*log(2/2))
    (hive,B),(100*log(2/2))


TF-IDF Implementation
---------------------

Q: Where can I find a TF-IDF implementation?

- Marcello de Sales implemented TF-IDF as a MapReduce job after he
  took the Cloudera developer course.

- You can view his code here.

    <https://marcellodesales.wordpress.com/2009/12/31/tf-idf-in-hadoop-part-1-word-frequency-in-doc/>


Grocery Store Sales and Co-occurrence
-------------------------------------

Q: How can MapReduce be used to find items that users frequently
purchase together?

    Receipt:
    - Chips
    - Dip
    - Diapers
    - Coffee
    - Coke
      |
      | Map
      v
    (Chips,Dip),1
      |
      | Reduce
      v
    (Chips,Dip),300
    (Chips,Coffee),1
    (Dip,Coffee),2


Find Mutuals Friends
--------------------

Q: Assume that the input is a collection of pairs which represents
friends. So in the following data 1 is a friend of 2 and 3, 3 is a
friend of 5 and so on.

    1,2
    1,3
    3,5
    5,7
    4,7
    1,4
    4,5

- If 1 is a friend of 2, 2 is a friend of 1
- If 1 is a friend of 2, then we don't care for mutual friends between 1 and 2
- For each pair of users find out how many mutual friends they have.

For example:

- 1 and 5 have mutual friends 3 and 4
- So the output for (1,5) is (1,5),2 
- The output must be (A,B),[Count of mutual friends of A,B]

In this case the output will be:

    (1,5),2
    (3,7),1


Join
----

Q: Illustrate reduce-side join with a simple example.

    Emp
    ---
    ID  Name   LocID
    --  ----   -----
    101 Alice  14
    102 Bob    15
    103 Chad   14

    Loc
    ---
    LocID  LocName
    -----  -------
    14     SF
    15     LA

    EmpLoc
    ------

    T   LocID  ID  Name   LocName
    -   -----  --  ----   -------
    L   14                SF
    E   14     101 Alice  
    E   14     103 Chad   
    L   15                LA
    E   15     102 Bob    


    EmpLoc
    ------
    ID  Name   LocID   LocName
    --  ----   -----   -------
    101 Alice  14      SF
    102 Bob    15      LA
    103 Chad   14      SF



Certification Details
=====================

Certification and Practice Exams
--------------------------------

Q: Where can I get more information about certification? 

- Details for certification are at:

    <http://cloudera.com/content/cloudera/en/training/certification.html>

Q: Besides this class what else can I do to prepare for the
certification exams?

- You can review the study guide for the exam which includes what
  topics are covered.

- Go to,

    <http://cloudera.com/content/cloudera/en/training/certification.html>

- Drill down into the certification you are interested in.

- Click on *Exam Study Guide*.

- Go through *Exam Sections* and review the material under each bullet
  point.


Hadoop Puzzlers
---------------

Q: Are there any online Hadoop puzzlers that I can use to prep for the
certification exam?

- This slide deck has some interesting Hadoop puzzlers. 
  
    <http://www.slideshare.net/Hadoop_Summit/hadoop-puzzlers>


Additional Resources
====================

Books and Papers
----------------

Cloudera University has arranged for a discount for our students at
O’Reilly. Use this URL to get the discount.

<http://shop.oreilly.com/category/deals/cloudera.do>

- *Hadoop: The Definitive Guide (4th Edition): Storage and Analysis at
  Internet Scale* by Tom White. O'Reilly Media. March 2015.

    <http://shop.oreilly.com/product/0636920033448.do>

- *Hadoop Operations* by Eric Sammer. O'Reilly Media. October 16, 2012
  (1st ed).

    <http://www.amazon.com/Hadoop-Operations-Eric-Sammer/dp/1449327052>

- *Hadoop Security: Protecting Your Big Data Platform* by Ben
  Spivey, Joey Echeverria. O'Reilly Media. August 2014.

    <http://shop.oreilly.com/product/0636920033332.do>

- *Data Availability and Durability with the Hadoop Distributed File
  System* by Robert J. Chansler. Usenix. Login Magazine. Feb 2012
  (Vol. 37 No. 1).

    <https://www.usenix.org/system/files/login/articles/chansler_0.pdf>

- *HDFS scalability: the limits to growth* by Konstantin Shvachko.
  Usenix. Login Magazine. April 2010 (Vol. 35 No. 2).

    <https://www.usenix.org/legacy/publications/login/2010-04/openpdfs/shvachko.pdf>

- *Learning Spark: Lightning-Fast Big Data Analytics* by Holden Karau,
  Andy Konwinski, Patrick Wendell, Matei Zaharia. Publisher: O'Reilly
  Media, June 2014. 
  
    <http://shop.oreilly.com/product/0636920028512.do>


- *The Google File System* by Sanjay Ghemawat, Howard Gobioff, Shun-Tak Leung. 
  SOSP’03, October 19–22, 2003.

    <http://research.google.com/archive/gfs-sosp2003.pdf>

- *MapReduce: Simplified Data Processing on Large Clusters* by Jeffrey
  Dean, Sanjay Ghemawat. OSDI 2004. 
  
    <http://research.google.com/archive/mapreduce-osdi04.pdf>

- *Bigtable: A Distributed Storage System for Structured Data* by Fay
  Chang, Jeffrey Dean, Sanjay Ghemawat, Wilson C. Hsieh, Deborah A.
  Wallach Mike Burrows, Tushar Chandra, Andrew Fikes, Robert E.
  Gruber. OSDI 2006. 
  
    <http://static.googleusercontent.com/media/research.google.com/en/us/archive/bigtable-osdi06.pdf>

- *Data-Intensive Text Processing with MapReduce* by Jimmy Lin, Chris
  Dyer. Available for $34 on Amazon or as a free download from Jimmy
  Lin's website.
  
    <http://mapreduce.me>.

- *MapReduce Patterns, Algorithms, and Use Cases* by Ilya Katsov.

    <http://highlyscalable.wordpress.com/2012/02/01/mapreduce-patterns/>

- *MapReduce Design Patterns: Building Effective Algorithms and
  Analytics for Hadoop and Other Systems*. by Donald Miner, Adam
  Shook. O'Reilly Media. December 22, 2012 (1st ed).

    <http://www.amazon.com/MapReduce-Design-Patterns-Effective-Algorithms/dp/1449327176>

- *Examples included in Hadoop source code*. Hadoop GitHub Repository. 

    <https://github.com/apache/hadoop-mapreduce/tree/HDFS-641/src/examples/org/apache/hadoop/examples>


Mailing Lists
-------------

Q: Is there a mailing-list in which I can ask technical questions
related to managing a Hadoop cluster?

<http://mail-archives.apache.org/mod_mbox/hadoop-user/>

