# Java SDK {#concept_dhw_rhl_vdb .concept}

This article introduces common MapReduce interfaces.

If you are using Maven, you can search "odps-sdk-mapred" from [Maven Library](http://search.maven.org/) to get the required Java SDK \(available in different versions\).  The configuration is as follows:

```
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-sdk-mapred</artifactId>
    <version>0.20.7-public</version>
</dependency>
```

|Interface|Description|
|:--------|:----------|
|MapperBase|The user-defined Map function is required to inherit from this class. It processes the record object of the input table, processes the object into key value and outputs the value to the Reduce stage, or outputs result record to the result table without passing through the Reduce stage.  Jobs that do not pass through the Reduce stage, but directly outputs computation results are called Map-Only job.|
|ReducerBase|Your customized Reduce function must inherit from this class. The set of Values associated with a Key is reduced.|
|TaskContext|It is one of the input parameters of multiple member functions in MapperBase and ReducerBase. Contains contextual information about tasks.|
|JobClient|It is used for submitting and managing jobs. The submission mode includes blocking \(synchronous\) mode or non-blocking \(asynchronous\) mode.|
|RunningJob|Indicates object in job running and used for tracing MapReduce job instance during the job running process.|
|JobConf|Describes configuration of a MapReduce task. The JobConf object is generally defined in the main program \(main function\), then jobs are submitted by JobClient to MaxCompute.|

## MapperBase {#section_dgt_z3l_vdb .section}

Main function interfaces are as follows.

|Interface|Description|
|:--------|:----------|
|void cleanup\(TaskContext context\)|The Map method is called after the map stage ends.|
|void map\(long key, Record record, TaskContext context\)|The Map method processes records of the input table.|
|void setup\(TaskContext context\)|The Map method is called before the map stage begins.|

## ReducerBase {#section_jng_hjl_vdb .section}

Main function interfaces are as follows.

|Interface|Description|
|:--------|:----------|
|void cleanup\( TaskContext context\)|The Reduce method is called after the reduce stage ends.|
|void reduce\(Record key, Iterator<Record \> values,  TaskContext context\)|The Reduce method processes input table records.|
|void setup\( TaskContext context\)|The Reduce method is called before the reduce stage begins.|

## TaskContext {#section_vch_kjl_vdb .section}

Main function interfaces are as follows.

|Interface|Description|
|:--------|:----------|
|TableInfo\[\] getOutputTableInfo\(\)|Gets output table information.|
|Record createOutputRecord\(\)|Creates the record object of the default output table.|
|Record createOutputRecord\(String label\)|Creates the record object of the output table with a specified label.|
|Record createMapOutputKeyRecord\(\)|Creates the record object of Key output by Map.|
|Record createMapOutputValueRecord\(\)|Creates the record object of Value output by Map.|
|void write\(Record record\)|Writes record to default output and is used for writing output data by Reduce client, and can be called on the Reduce client multiple times.|
|void write\(Record record, String label\)|Writes record to the given label output and is used for writing output data by Reduce client, and can be called on the Reduce client multiple times.|
|void write\(Record key, Record value\)|Map writes record for an intermediate result. It can be called in Map function  and called on the Map client multiple times.|
|BufferedInputStream readResourceFileAsStream\(String resourceName\)|Reads file type resource.|
|Iterator<Record \> readResourceTable\(String resourceName\)|Reads table type resource.|
|Counter getCounter\(Enum<? \> \> name\)|Gets the Counter object with the specified name.|
|Counter getCounter\(String group, String name\)|Gets the Counter object with specified name and the group name.|
|void progress\(\)|Reports heartbeat information to the MapReduce framework. If a user’s method takes a long time to process, and no framework is called in the process, this method can be called to avoid task timeout.  Timeout of the framework is 600s by default.|

**Note:** 

-   MaxCompute TaskContext interface provides the progress function, however, this function is to prevent the Worker from being terminated as it runs for long time and the framework considers it as a timeout Worker. This interface is similar to sending heartbeat information to the framework, but does not report the progress of the Worker. 
-   The default timeout schedule of MaxCompute MapReduce Worker is 10 minutes \(system default, cannot be controlled by the user\). If the schedule exceeds 10 minutes and Worker is unable to send heartbeat information to the framework \(not to call progress interface\), the framework is forced to stop this Worker and MapReduce task fails and exits. We recommend calling the progress interface regularly in Mapper/Reducer functions  to prevent the worker from being terminated by the framework.

## JobConf {#section_nht_pjl_vdb .section}

Main function interfaces are as follows:

|Interface|Description|
|:--------|:----------|
|void setResources\(String resourceNames\)|Declares resources used in this job. Only the declared resource can be read by TaskContext object during Mapper/Reducer running process.|
|void setMapOutputKeySchema\(Column\[\] schema\)|Sets the Key attribute output from Mapper to Reducer.|
|void setMapOutputValueSchema\(Column\[\] schema\)|Sets the Value attribute output from Mapper to Reducer.|
|void setOutputKeySortColumns\(String\[\] cols\)|Sets key sort columns output from Mapper to Reducer.|
|void setOutputGroupingColumns\(String\[\] cols\)|Sets Key grouping columns.|
|void setMapperClass\(Class<?  extends Mapper \>  theClass\)|Sets Mapper function of the job.|
|void setPartitionColumns\(String\[\] cols\)|Sets the partition column specified in the job. The default is all columns of Key output by Mapper.|
|void setReducerClass\(Class<?  extends Reducer  theClass\)|Sets Reducer of the job.|
|void setCombinerClass\(Class<?  extends Reducer  theClass\)|Sets combiner of the job, running on Map client. Its function is similar to performing Reduce operation on the identical local Key values by a single Map.|
|void setSplitSize\(long size\)|Sets the size of input slice. Unit: MB. The default value is 640.|
|void setNumReduceTasks\(int n\)|Sets the number of Reducer tasks. The default is 1/4 of Mapper tasks.|
|void setMemoryForMapTask\(int mem\)|Sets the memory size of single Worker in the Mapper task. Unit: MB. The default value is 2048.|
|void setMemoryForReduceTask\(int mem\)|Sets the memory size of single Worker for Reducer task. Unit: MB. The default value is 2048.|

**Note:** 

-   Usually, GroupingColumns are included in KeySortColumns, while KeySortColumns and PartitionColumns are included in the Key.
-   In the Map side, mappers’ output records are distributed to reducers according to the hash values computed using PartitionColumns, and then sorted by KeySortColumns.
-   In the Reduce side, after being sorted by KeySortColumns, input records are grouped as input groups of the reduce function sequentially. In other words , records with the same GroupingColumns values are treated as the same input group.

## JobClient {#section_jz2_5jl_vdb .section}

Main function interfaces are as follows:

|Interface|Description|
|:--------|:----------|
|static RunningJob runJob\(JobConf job\)|Returns immediately after submitting a MapReduce job in a synchronous \(blocking\) mode.|
|static RunningJob submitJob\(JobConf job\)|Returns immediately after submitting a MapReduce job in an asynchronous \(non-blocking\) mode.|

## RunningJob {#section_wnp_vjl_vdb .section}

Main function interfaces are as follows.

|Interface|Description|
|:--------|:----------|
|String getInstanceID\(\)|Gets an instance ID for checking run log and job management.|
|boolean isComplete\(\)|Checks whether job is complete.|
|boolean isSuccessful\(\)|Checks whether job instance is successful.|
|void waitForCompletion\(\)|Waits until job instance is complete. It is typically iused for jobs submitted is asynchronous mode.|
|JobStatus getJobStatus\(\)|Checks job instance status.|
|void killJob\(\)|Ends the job.|
|Counters getCounters\(\)|Gets Counter information.|

## InputUtils {#section_udl_xjl_vdb .section}

Main function interfaces are as follows:

|Interface|Description|
|:--------|:----------|
|static void addTable\(TableInfo table, JobConf conf\)|Adds table to the task input. It can be called multiple times. The new added table is added to input queue in an append mode.|
|static void setTables\(TableInfo \[\] tables, JobConf conf\)|Adds tables to the task input.|

## OutputUtils {#section_uzf_zjl_vdb .section}

Main function interfaces are as follows:

|Interface|Description|
|:--------|:----------|
|static void addTable\(TableInfo table, JobConf conf\)|Adds table to the task output. It can be called multiple times. Also, adds the new added table to output queue in an append mode.|
|static void setTables\(TableInfo \[\] tables, JobConf conf\)|Adds multiple tables to the task output.|

## Pipeline {#section_zld_bkl_vdb .section}

Pipeline is the subject of [MR2](reseller.en-US/User Guide/MapReduce/Summary/Extended MapReduce.md). It can be constructed by Pipeline.builder. Pipelines are as follows:

```
    public Builder addMapper(Class<? extends Mapper> mapper)
    public Builder addMapper(Class<? extends Mapper> mapper,
           column [] keyschema, column [] valueschema, string [] sortcols,
           SortOrder [] order, string [] partcols,
            Class<? extends Partitioner> theClass, String[] groupCols)
    public Builder addReducer(Class<? extends Reducer> reducer)
    public Builder addReducer(Class<? extends Reducer> reducer,
           column [] keyschema, column [] valueschema, string [] sortcols,
           SortOrder [] order, string [] partcols,
            Class<? extends Partitioner> theClass, String[] groupCols)
    public setoutputkeyschema builder (Column [] keyschema)
    public setoutputvalueschema builder (Column [] valueschema)
    public setoutputkeysortcolumns builder (String [] sortcols)
    public setoutputkeysortorder builder (Sortorder [] order)
    public setpartitioncolumns builder (String [] partcols)
    public Builder setPartitionerClass(Class<? extends Partitioner> theClass)
    void setOutputGroupingColumns(String[] cols)
```

Example:

```
    job job = new job ();
    pipeline pipeline = pipeline. builder ()
     . addmapper (Tokenizermapper. class)
     . setoutputkeyschema (
         new column [] {new column ("word", OdpsType. string)})
     . setoutputvalueschema (
         new column [] {new column ("count", OdpsType. bigint)})
     . addreducer (Sumreducer. class)
     . setoutputkeyschema (
         new column [] {new column ("count", OdpsType. bigint)})
     . setoutputvalueschema (
         new column [] {new column ("word", OdpsType. string),
         new column ("count", OdpsType. bigint)})
     . addreducer (Identityreducer. class). createPipeline ();
    job. setpipeline (pipeline);  
    job. addinput (...)
    job. addoutput (...)
    job. submit ();
```

As shown in the preceding example, a user can construct a Map in the main class, and then consecutively get MapReduce tasks of two Reduces.  If you are familiar with the basic functions of MapReduce,  then you can use MR2 as well, as the functions are similar.

**Note:** 

-   Specifically, we recommend that users must complete the configuration of MapReduce task by JobConf, 
-   as JobConf can get MapReduce task of single Reduce only after configuring Map.

## Data Type {#section_bqj_3kl_vdb .section}

The data types supported in MapReduce include: BIGINT, STRING, DOUBLE, BOOLEAN, and DATETIME.  MaxCompute between MaxCompute data types and Java types are as follows:

|MaxCompute SQL Type|Java Type|
|-------------------|---------|
|Bigint|Long|
|String|String|
|Double|Double|
|Boolean|Boolean|
|Datetime|Date|
|Decimal|BigDecimal|

