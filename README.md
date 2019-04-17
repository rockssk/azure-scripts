Exxam 70-475: Azure Data Engineering notes

Areas to practice

Batch and Interactive
Streaming data analytics
End to End operationalizing

Batch and Interactive
================
ARM - Azure Resource Management(json based)
Azure SQLDB
Azure SQL Datawarehouse
Polybase
DMG(Data Management Gateway)
ADF(Azure Data Factory-Json based)
Lambda(developed by Azmazon)
Polybase
PDW
CosmosDB(Document DB)
Event Hub
Storm/spark streaming/Azure stream analytics
try to take MeasureUp exam practice test
AzureML
TDE(Transparent Data Encryption)
PDW(Parallel Datawarehouse)
cold store ,hot store
egress/ingress
sqldw(similar to aps n pdw,born in the cloud)


Design bigdata realtime processing
======================
Storm(Streams,Queue-kafka/eventhub/azure service bus queues)
Enabling Intusion detection



Operationalizing end-to-end cloud analytics
==========================
Dont share primary storage key, share only the secondary storage access key with partners or vendors
setting up hybrid env
DMG(Can connect Netezza,teradata,hadoop on-prem n cloud)
SSIS
Alerts and ADF event monitoring
Try some hands-on powershell 
SQL Datawarehouse was an early storage for hadoop before ADLS

Session 1: Processing Big Data with Hadoop in Azure HDInsight
=======================================

https://courses.edx.org/courses/course-v1:Microsoft+DAT202.1x+1T2019/course/

Azure SQL DB : Used to store metadata for hive/oozie in HDI Cluster

Module 1: Getting Started with HDI-> HDFS,MapReduce,Provisioning HDI Cluster
Module 2: HDInsight Client tools
- Azure CLI
- Powershell
-Visual Studio
- Storage explorer
wasb(s)://container@storageaccount.blob.core.windows.net/data/logs/file.txt
native access to blobs from default storage account and container :/data/logs/file.txt

             - hadoop jat <jarfile.jar> classfile <source_data_dir> <output_data_dir>
             - A Sample powershell with HDI
- Azure powershell can be used to work with Azzure SQL DB,Azure Storage ,provision a cluster,upload/download file,run a mapreduce job
cmdlet


Module 3: Hive 
To Access hive  : Hive shell,Hue,visual Studio,Powershell,ODBC Client(Excel,PowerBI)
Data types: TINYINT,SMALLINT,INT,BIGINT
STRING,VARCHAR,CHAR
BOOLEAN,BINARY,ARRAY,MAP,STRUCT,UNIONTYPE
DATE,TIMESTAMP
while reading data, type is layered on top of data, nulls if data and type doesnt fit
Views : Named queries that abstracts underlying tables
unix_timestamp("16/12/2018 14:55:25",'dd/MM/yyy hh:mm:ss') - gives numerical  value of a date
from_unixtime(unix_timestamp)  - Gives standard timestamp value in YYYY-MM-DD hh:mm:ss


  Skewed table creation : create table skewed_table(col1 int,col2 string,col3 string) skewed by (col3) ON ('A') [stored as directories];
insert into table ksewed_table select col1,col2,col3 from staging_table; //Mainly used to create a single partition on col3 for value A where high proportionate of data goes into
clustered Table: imporves mutiple table joins , as it creates a hash of the key column and stores that many number of partitions
PIG: Relations , need not to  have defined rows and columns. suitable for processing xml,json files
  Bag(collection of tuple),Tuple(ordered set of fields),field(a data item). a field can contain an outer bag(d,{(4,5),(6,7)})
PIG latin operations:
    Load,foreach .. generate,filter,order,JOIN,group,flatten,limit,dump(dumping output to console),store(storing a relation intoa file)

UDF: can be written in  Java,C#,Python(simple scripting,intuitive syntax,dynamic typing,interpreted execution)
        python udfs in pig can be invoked through jython interpretor( REGISTER  'wasbs://scripts/myscript.py' usign jython  as myscript;)
Python udfs in hive can be invoked using streaming technique(rows from hive data are passed to python in STDIN and output sent to STDOUT)->(hive>add filewasbs://script/myscript.py) use TRANSFORM to invoke UDF

Oozie: ooozie job -oozie <oozie_URL> -config job.properties -run        
Sqoop : 

Course 2: Implementing Real-Time Analytics with Hadoop in Azure HDInsight
==================================================
HBase-NoSQL
Storm-Streaming Data
Spark-Interactive Data Analysis

HBase: LowLatent,NOSQL,Key value pairs( key,columnfamily,column qualifier). Columns are row specific. Cells are version controlled, temporal feature
create 'readings' 'sensor', 'reading'
put 'readings' , '1','sensor:id' .'sensor1'
put 'readings', '1','reading:datetime','2018-01-01'  ( to insert a new row or update value of a cell in an existing row)
get 'readings' ,'1'
get 'readings' '1' {columns=>'sensor'}
delete
scan
drop 

Bulk load data into hbase table : 
1. upload data into hdfs(azure storage)
2. IMport into store file
3. load the store file into hbase table(LoadIncrementalHFiles)-> (hbase org.apache.hadoop.hbase.mapreduce.LoadIncrementalHFiles /data/storefile stocks)


Create hive table to get data from hbase
create external table hive_hbase(key string,col1 string, col2 string)stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
with serdeproperties
('hbase.columns,mapping'=':key,cf:col1,cf:col2')
tblproperties('hbase.table.name'='htable')
Phoenix (SQL on HBase)
==============
/usr/hdp/2.3/phoenix/bin/sqlline.py zk.hdp.windows.net:2181:/hbase-unsecure -> to connect to hbase using phoenix , we need to connect on zk node

create views in phoenix for tables in hbase
Client (JAva and .net wrapper) also available
Java client for hbase table(HBaseConfiguration,HTable,Scan,put,ResultScanner)

Storm:
====
stream: unbounded , constant stream of data, event data. Aggregation is based on temporal windows. Ex. watch for hashtag from stream of tweets
Storm : Event processer for data streams, written in java, microsoft scp.net package enables development in C#
Topology -> spouts(component in storm topology that consumes data from sources and sends them as streams collection of tuples for processing) 
- It can obtain data from eventhub,kafka or a network port
- OutputSchemaDeclarer ,declares the types of data in the stream(ex.int,string,string)
-Spout class extends BaseRichSpout
Bolts -> logic that Operates on tuples in streams,InputSchemaDeclarer
->Bolt class extends BaseBasicBolt
Topology class extends TopologyBuilder class that connects the components, develop ,compile and submit the code to the cluster
Storm Cluster (Supervisor node-1, numbus node-2,zookeeper node-3 nodes)
How to submit a storm code into the cluster
storm jar <app.jar> <topology_class_name> <topology_streamname_to_submit_to_cluster>
Storm parallel processing and grouping
Master node runs Nimbus(allocates task),worker nodes run supervisor(manages task within the slave)-worker process spawns executors
setBolt.shuffleGrouping_> to distribute processing
fieldsGrouping("spout","f1") ->similar to hashpartitioning,if aggregation on a particular fields , then shuffleGrouping requires re aggregation of distributed results 
By Default TopologyBuilder is a non-transactional ones, for resiliency , we can use acker to maintain acknowldgements of data being processed
or use TransactionalTopologyBuilder that maintains a committer bolt all the way at the end .

How aggregation happens in Storm 
- Tumbling window - to aggregate events in a fixed temporal window
ex. Every hour process the events happened the previous hour
- Sliding window - to aggregate in overlapping timespans
ex. Every 5 minutes aggregate the events for prev 1 hour

Apache Spark:
=======
Fast general purpose computing engine supports in-memory operations
RDD - Data in spark is RDD,represents a collection of data that can be distributed,Java,Scala n Python API
SparkContext-Driver node uses to connect to cluster
spark-shell- brings sparkcontext by default, but writing our own driver code requires a sparkcontext to be initiated
cfg=SparkConf().setMaster("yARN").setAppName("myapp")
 sc=SparkContext(conf=cfg)
to create an RDD , create it from a textfile
txtRDD=sc.textFile("/tmp/a.txt")
lstRDD=sc.parallelize(["A","B","C"])
RDD Transformation ->Create a new RDD by transpiformign an existing RDD RDD.filter(lambda)

owltxt=rdd.filter(lambda t: "owl" in t) ->filter creates a filtered RDD
words=owltxt.flatMap(lambda t:t.split(" ")) - > flatMap is called to flatten, or applies a function against each element in an RDD to obtain multiple elements from it
kv=words.map(lambda key:{key,1)))-> map is used when one-to-one transformation is needed
counts=kv.reduceByKey(lambda a,b:a+b) ->reduceByKey aggregates the value for a given key

RDD Actions->Returns results to the driver program(rdd.count())
rdd.persist()-> since RDD are lazy evaluated, to improve perfromance on a repeatedly used RDDs persist them 
rdd.reduce(lambda x, y:x+y) - > reduce - aggregates the elements of an rdd using function that takes 2 arguments
collect() -> sends the result to driver program
first() ->prints the first element
count()->gets the count of elements in an RDD
saveAsTextFile->saves the results in a text file 

Notebooks: experimentation and collaboration
Spark Dataframes and SparkSQL
Dataframe -  is an untyped dataset, always yields generic row, where as dataset(only in Java and Scala) is a typed (val ds= df.as [myclass]);ds.filter(_.price>2))
An RDD can be saved as a temporary view to be able to queried as a table
temporaryviews->df.createOrReplaceTempView("t1");df=spark.sql("Select price from t1")
sharedtables
a dataframe can have simple sql equivalent functions like join,filter,select etc but for complex sql statements on a dataframe, we need to use spark sql after saving the dtaframe into a view or a table
spark.sql.("show tables").show() - will provie all list of temp view, hive tables


Spark Streaming
==========
-streaming module built in spark
-Data is presented as sequence of RDDS in discretized streams(DStreams)- reduceByKeyAndWindow(lambda a,c:a+b,lambda x,y:x-y,60,10) -check every 10 seconds  and aggreagate values occured in the last 60 secs
- Many sources supported (TCP Socket,File,flume,kafka,Azure EventHUb)
-Spark2.x supports dataframes in streams instead of RDDs(Structured Streaming), perpetual query on stream of data
-from pyspark.streaming import StreamingContext
ssc=StreamingContext(sc,1)
ssc.checkpoint("wasbs:///abc")
streamTxt=ssc.textFileStream("wasbs:///stream")

StructiredStreaming
===========
df=spark.readStream.schema(jsconSchema).option("maxFilesPerTrigger",1).json(inputpath)
countDF.writeStream("memory").queryName("counts").outputMode("complete").start()


Spark Structured streaming and Azure EventHub
================================
same like dstreams, just that the option filed includes the eventHubparameters


Kafka:
====
KAfka is a cluster to get messages in real-time stores them in topics

Kafka broker - Worker node
how to create a kafka topic
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh  --create --replication-factor  1 --partitions 8 --topic sensors --zookeeper <zk_host1:2181>, <zk_host2:2181>, <zk_host3:2181>,

start the producer
=========
kafka-console-producer.sh --broker-list $KAFKABROKERS --topic sensors

Start the consumer
==========
 kafka-console-consumer.sh --bootstrap-server $KAFKABROKERS --topic sensors

COURSE 3 :Processing BigData with Microsoft Azure Datalake Analytics
================================
all jobs in ADLA are USQL( SQL and C# for custom functions)
- A simple job could be to do some cleanup ,filtering and aggregation on files stored in ADLS or blobstore or a database
- all are C# datatypes and not SQL types, to specify nullable add?(name string,id int?)
- to skip the rows without throwing error set silent=true
- U-SQL include built in extractors(Extractors.Text,Extractors.Csv,Extractors.Tsv), some of the parameters they accept(delimiters,encoding,SkipFirstNRows,silent)
-Outputters.Csv()
-One can execute U-SQL job from visual studio on adla account, Azure SDK need to be installed
- U-SQL are case sensitive. WHILE,MERGE,UPDATE are not supported


U-SQL Catalog
   Similar to Hive, we can create managed and external tables
Once new data source is being added refering Azure SQL Datawwarehouse,SQLServer, we can use create external table command 
- Views functions and procedures can also be created
- Like HIve UDF, you can write C# code and compile it to get a .DLL file ,reference that in U-SQL(REFERENCE MyAssembly myasm, select myasn.myclass.myfunc())
to improve U-SQL jobs, increase the units for parallel processing
U-SQL job board gives a diagnostics option also

Model a job - to get the right resources(AU usage model)

Course 4: Ingesting realtime data with Azure
============================

Azure Event Hub : Millions of events/second
- max 10 Eventhuibs/namespace
-SharedAccessKey created at namespace level
-Sharedaccess sugnature token- used to authenticate client apps 
32 partitions/eventhub (partitions are nothing but commit logs equivalent)
Consumer groups are views on top of event hubs
namespace.servicebus.windows.net
event hub SDK available in java,C#,python ,node.js 
npm install azure-event-hubs( to install sdk for node.js)
Connect through connection string
node eventclient.js ( to produce the test events from a test clien application to the evetn hub) 

TimeSeries analysis in EventHUb
====================
TimeSeriesInsights ->create an event source to monitor the events from EventHUb 

IoTHub:   Similar to EventHub but designed to get messsages from devices connected through internet and the communication is bi-directional,meaning IoTHub can send messages back to the device( ex.Alexa,google voice etc)

Each device need to be registered with the hub using a secure endpoint. per device identity
IoTReader can read messages from an eventhub also

AzureStreamAnalytics : Similar to Aure DatalakeAnalytics, supports standard SQL querying. Input is from AzureEventHUb or IotHub, output can be to powerBi, or a sql database

all events are added with a field(EventtEnqueuedUTCTime)
Stream Topologies -  you can build multiple source,target in a farly complex flow 
a sample query: select col1,col2 into <table> from <queu>

Working with Temporarl Window
Tumbling window - simple,Constant time window, every 10 seconds a window example. An example would be to compute the average price of a stock over the last five minutes, computed every five minutes.Tumbling window is similar to hopping window in which the hop is equal to its  window size

Hopping window - moderate complex, HOPPINGWINDOW ( timeunit  , windowsize , hopsize, [offsetsize] )   . Every 5 seconds give me the count of tweets in the past 10 secs
Sliding window - highly complex ,An example would be to compute the moving average of a stock price across the last five minutes, triggered every second. Window starts when a new message comes in not based on hop time


Timestamps - by default windowing technique applies on top of messages in the order they recieved, with timestamp feature one can order the messages based on the timestamp field in the message itself before getting aggregated
Aggregating 

DAT223.3x: Orchestrating Big Data with Azure Data Factory. 
=======================================

Azure Data Factory : Encapsualtes workflow in a pipeline, mostly data integration workflows
Azure Logic apps :  orchestrator but best suited for integrating appliation rather than data

Pipelines services : can be created using wizard , under the covers its creating a json file with the workflow details, similar to oozie workflow.xml
While scheduling adf pipelines, availability frequency ='Day', interval='3' means once in 3 days. concurrency=2(2 threads)


Transformations services:  While Pipeline services are generally used to copy data or call multiple data integration activities, Transformation services on the other hand used to perform some transformation on the data while its movement

U-SQL pipelines contain: Tables,Dependancies,LinkedServices,REferences,pipelines,
Hive Transformations : timetolive: hh:mm:ss ( hours to live after completion)



MeasureUp PracticeTest:
================
Company B : Startup inv (adv real est,sec,prec metals). near real-time analysis of inv opportunities
datacenter with on-prem sqlserver db,MicrosoftCRM. Data infra is on Azure

Requirements:
==========
Customer data update:Mobile app collects customer data( category,housing,metaletc.. basically nested folders or partitions)
Data from blobstore to AzureSQLDatabase
new data blobstore -twice /day
Analyst - PowerBI

Quick Trac:
=======
investment API realtime
realtime querying
analyst-PowerBI
dev-on existing talent pool


Customer data update:
=============
parallel process(sqlserver,asqldb,crm webserivce->single dataset,publish back to sqlserver)
daily data update must use shared memory cluster

Quick alert app : visualstudio ->developer,C# Analysysts->PowerBI

Automation: Powershell - deployment automation  - Create a resource manager powershell script that supports incremental deployment, cmdlets(New-AzureRmResourceGroup,New-AzureRmStorageAccount)
azure storage account
related resources to be grouped
secured access
each deployment, updates previous
resource with no change shouldn't be provisioned, even if they are in the resource group

container.metadata.Add("key","value") or container.metadata["key"]="value"




answers: Spouts connects to API,Spouts perform first level partitioning for the raw data Bolt- to filter data
Storm should be used for Quick Alerts Data ingestion: due to its native use of visual studio

Case Study 3: 
=======
TESLA
processed car data is secured in the cloud
Customer data -on-prem and cloud
Car data is automatically organized into datasets, scheduled 3 times /day . each car at diff times 
alerts to be sent to departments for failures

Technical requirements
datasets ->parsed to multiple datasets , then processed by several pipelines
pipelines to be updated to process old car data
car data is used in across multiple activities and pipelines
car data from azure sqldb to organized datasets across multiple pipelines
Autonomous pipeleine->AD
Driver pipeline->requires AD->oututs DD

External to vehicle pipeline->ED
vehicle mechanics pipeline->ED->VD
Secondary pipelines->
Autonomous data is processed through HDI
ADF is monitored using aure portal and Datafactory monitoring app
Aged data is moved to blobstore
For automated batch deployment use Powershell and not azure portal

PSEG:
====
Utiity company Y
meter reading ssent over internet
maximize profile and reduce energy cost
On-Prem sqlserver database 2016
customer phone in plain text
existing web-app uses T-SQL to filter customers by phone number 
as part of pilot company wants to add one or more tables
propreitary accounting system stores utility bills and payments

Biz.REquirements:
========
solution azure resources only if its technically beneficial
protect customer PII, webapp should dissplay only the last 4 digits
minimize update in exsting apps
predict meter usage based on historical meter data

Technical requiremet:
=============
meter reading in real time
sql queries to process and access meter data in real time
processed meter usage data as json objects
Use R for predicting usage trends

DocuemtnDB should be used to store data in long term and it supports SQL /LINQ
Dynamic Data Masking feature in SQL2016

You cannot deploy ADF through TEam foundation or Visual Studio
Powershell is the only way to script out the deployment automation of ADF
PowerBI can directly access hive data ,PowerBI can also work with spark data

AzureMLBatch Execution(to train and score) in Azure Data Factory is used to invoke machine learning 
Block blobs are ideal for 200GB per message
Page blobs are tiny collection of pages 512 bytes with a page offset
append blobs are blocks for appending


Case Study : Insurance
===============
3  data center, data from legacy ,sql and cloud
1. Migration:
SQLServer from 3 DCs will need to be moved to cloud in 6 months
no disruption to the existing process
2. Daily Consolidation - nightly batch
cloud,on-prem,critical data 
processed data should go to single cloud based storage 
existing IT skills for tools 
data should be available at the sod 
LOB's couldnt be taken down
3. Claim Jumper: Real time customer driving behavior
data collected from sensors in the vehicle
customer data must be correlated with real time data
data analysts must be able to uery real time
Tech requirement
==========
DBAs and Devs are skilled in T-SQL
programmers are python
powerBI

security
====
credentials are to ne encypted
no firewall rules to be modified
legacy data with no certificate

App REquiremtns
processed data should go to ASQLDW
processers must be added as necessary to complete on time

Claim jumper: imput must be read from disk based solution

Implementing Predictive Analytics with Spark in Azure HDInsight
==========================================

Supervised Learning
=============
features- vector of numbers
label-result
Train the model
Test the model
classification algorithms: LogisticRegression for binary classification(ex. whether a flight will arrive delayed)
=============================
DecisionTree

ML models tend work best with numeric values than text
Prepare the column n a binary 

Training a model 
1. prepares a  vector  of features and label column
2. use logisticregression and fit the dataframe
REegularization parameter removes some bias(overfitting)
model = lr.fit(training_dataframe)

Testing a model
prediction=model.transform(testinf_data_frame)

Probability vector

Building a classification model usign python

Regression (how many minutes delayed a flight might arrive)
=======
Linear regression : Rather than predicting 0 or 1, predicts a numerical value

Pipeline Model
=========DecisionTreeclassfier
programming model to do multiple stages of transformation before training the model

Categorical feature(Ex, AirportID, dayofmonth,dayofweek,carrierid) , though they are numerical values, they are non changing or dynamic variables
continuous variable : (depDelay_in_mins,arrival_delay_in_minutes)
transformaers and estimators

tesing has to go through the same transformation that training data went thru

TextAnalytics
=========
Tokenizer,Stopwordsremover,HashingTF


EValuating a classification model
======================
Confusion Matrix : Evaluate pridicted vs actual label(TP,FN,TN,FP)  1  vs  0 vs 1 0 
threshold
accuracy is not the right measure to evaluate a classification model, becasue either 0 or 1 will always lean towards either side

Precision and recall are used to measure the effectiveness of the model, threshold is adjusted to achive 

Precision=TP/(TP+FP)
Recall=TP/(TP+FN)

RoC Curve
AUC: Area Under Curve

Evaluating a Regression model:
====================
Draw a scatter plot of predicted vs actual value and calcualte the average difference

RMSE : average diff of predicted values vs actual values
Parameter Tuning
==========
Compose a parameter grid with diff combination of maxIter and regParam, and train the model , evaluate the data and pick the best performaing combination of parameter for actual model training.

Cross validation:
==========
A process of making multiple folds of same data and train and validate the model with diff combination of parameter matrix , to get a more generalized model which is not overfit

Unsupervised learning ( you dont know the true label)
===============

Recommender
=========
Userbasesimilarity
Itembasedsimilarity
Latent features

ALS

Clustering - Cusotmer segmentation
======
K-Means clustering - plot the numerical values of various features in a 2 dimenstional space
keep moving the item to the center point of the cluster and find who all are close to the center item, forming a group


Course 220x : Delivering SQLDatawarehouse in the cloud
===================================
GEoBackup Policy
DWU: Datawarehouse Unit
Traditional on-prem dw:SMP (Symmetrical multi-processing) vs Azure SQL DW:MPP(massively parallel processing)Control nodes and compute nodes
DWU100(All 60 SQLDBs in 1 compute node)-DWU6000(each sqldb in each compute node)

distribution technique
-----------------------
Hash distribution : Adv: Scans faster,Disadv: Dataskew
Round robin: Adv: Even distribution, Query runs slow, cause all nodes need to be scanned
REplicated tables: Table rows are replicated to all 60 SQLdbs(avoid data movement). no broadcastMoveOperation in query plan. dimension tables that are less than 2GB in size are ideal for replicate distribution

dm_pdw_exec_requests
dm_pdw_request_steps

Partitioned tables(Partition switch and partition elimintation) : load ,delete and select runs faster
ClusteredColumnStore: Rowgroup of 100 mi rows

ClusteredColumnStoreIndex - Default column store index in azuresqldw - HIgh performant ,high compressed.Each rowgroup has segments, each segment maintains min and max of data value.
while creating clusteredcolumnstoreindex, no need to specify a column

ClusteredIndex or clusteredrowstoreindex outperforms when the need is to select single query
Nonclusteredndex or Nonclusteredrowstroreindex maintains index in physical storage

HEAP - 

statistics - Query Optimizers use them to understand cardinality
CostBasedOptimizer
AzureSQLDW doesnt update or create stats by default
To apply label to query(option(label="name")

ElasticQuery:ability to query sqldw from sqldb or powerBI,excel etc
:Create user,create master key,create external data source,create external table

LinkedService - connection
Datasets - Structure of the data
Activities
Pipelines

V1 vs V2 : json template vs drag and drop
ServicePrincipal vs OAuth(For User credentials auth)

Azure Stream Analytics: Transform data in motion, Data from stream analytics can be written to AzureSQLDW.
Cold path data from streams sent to asqldw and adls. Hot path data sent to PowerBI


Polybase
create masterKey
creatae database scoped credentials
create external datasources
create external file format
creaete external table
CTAS


You can query the external data from polybase, u can also use polybase to load data into sqldw from an external source
Once data is loaded, we need to manually rebuild index and run statistics on


BCP : command line utility to dunp and load,file to db or db to file, no in-built retry,large dw to be moved then use bcp, fastest
SSIS: UI,typical ETL tool, not the fastest , allows retry option

ADF vs Polybase vs Stream Analytics  : 
PowerBI: Hot Path for realtime data visualizatin
D
DatawarehouseMigration Utility:

SQLDW RPO: 8 Hours
Backup:copy of the db conaitning multiple data files stored in azure storage
Local snapshot: sys.pdw_loader_backup_runs to check the last snapshot
Geo Replication: On by default,backed up to a paired datacenter once a day
24 Hour RPO


https://docs.microsoft.com/en-us/azure/sql-data-warehouse/sql-data-warehouse-best-practices
