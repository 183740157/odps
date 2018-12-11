# Example for multi-thread downloading {#concept_lq5_k4g_vdb .concept}

This article shows you how to use the TableTunnel interface to achieve multithreaded download through code examples.

```
import java.io.IOException;
 import java.util.ArrayList;
 import java.util.Date;
 import java.util.List;
 import java.util.concurrent.Callable;
 import java.util.concurrent.ExecutionException;
 import java.util.concurrent.ExecutorService;
 import java.util.concurrent.Executors;
 import java.util.concurrent.Future;
 import com.aliyun.odps.Column;
 import com.aliyun.odps.Odps;
 import com.aliyun.odps.PartitionSpec;
 import com.aliyun.odps.TableSchema;
 import com.aliyun.odps.account.Account;
 import com.aliyun.odps.account.AliyunAccount;
 import com.aliyun.odps.data.Record;
 import com.aliyun.odps.data.RecordReader;
 import com.aliyun.odps.tunnel.TableTunnel;
 import com.aliyun.odps.tunnel.TableTunnel.DownloadSession;
 import com.aliyun.odps.tunnel.TunnelException;
 class DownloadThread implements Callable<Long> {
         private long id;
         private RecordReader recordReader;
         private TableSchema tableSchema;
         public DownloadThread(int id,
                         RecordReader recordReader, TableSchema tableSchema) {
                 this.id = id;
                 this.recordReader = recordReader;
                 this.tableSchema = tableSchema;
         }
         @Override
         public Long call() {
                 Long recordNum = 0L;
                 try {
                         Record record;
                         while ((record = recordReader.read()) ! = null) {
                                 recordNum++;
                                 System.out.print("Thread " + id + "\t");
                                 consumeRecord(record, tableSchema);
                         }
                         recordReader.close();
                 } catch (IOException e) {
                         e.printStackTrace();
                 }
                 return recordNum;
         }
         private static void consumeRecord(Record record, TableSchema schema) {
                 for (int i = 0; i < schema.getColumns().size(); i++) {
                         Column column = schema.getColumn(i);
                         String colValue = null;
                         switch (column.getType()) {
                         case BIGINT: {
                                 Long v = record.getBigint(i);
                                 colValue = v == null ? null : v.toString();
                                 Break;
                         }
                         case BOOLEAN: {
                                 Boolean v = record.getBoolean(i);
                                 colValue = v == null ? null : v.toString();
                                 break;
                         }
                         case DATETIME: {
                                 Date v = record.getDatetime(i);
                                 colValue = v == null ? null : v.toString();
                                 break;
                         }
                         case DOUBLE: {
                                 Double v = record.getDouble(i);
                                 colValue = v == null ? null : v.toString();
                                 break;
                         }
                         case STRING: {
                                 String v = record.getString(i);
                                 colValue = v == null ? null : v.toString();
                                 break;
                         }
                         Default:
                                 throw new RuntimeException("Unknown column type: "
                                                 + column.getType());
                         }
                         System.out.print(colValue == null ? "null" : colValue);
                         If (I! = schema.getColumns().size())
                                 System.out.print("\t");
                 }
                 System.out.println();
         }
 }
 public class DownloadThreadSample {
         private static String accessId = "<your access id>";
         private static String accessKey = "<your access Key>";
         private static String odpsUrl = "http://service.odps.aliyun.com/api";
         private static String tunnelUrl = "http://dt.cn-shanghai.maxcompute.aliyun-inc.com";
                         //The tunnelURL must be set if you need to connect internal network, otherwise, the system uses public network as default. The example shows the Tunnel Endpoint of classical network in HuaDong 2, for other regions, see Access domain and data centers.
         private static String project = "<your project>";
         private static String table = "<your table name>";
         private static String partition = "<your partition spec>";
         private static int threadNum = 10;
         public static void main(String args[]) {
                 Account account = new AliyunAccount(accessId, accessKey);
                 Odps odps = new Odps(account);
                 odps.setEndpoint(odpsUrl);
                 odps.setDefaultProject(project);
                 TableTunnel tunnel = new TableTunnel(odps);
                 tunnel.setEndpoint(tunnelUrl);  //set tunnelUrl
                 PartitionSpec partitionSpec = new PartitionSpec(partition);
                 DownloadSession downloadSession;
                 try {
                         downloadSession = tunnel.createDownloadSession(project, table,
                                         partitionSpec);
                         System.out.println("Session Status is : "
                                         + downloadSession.getStatus().toString());
                         long count = downloadSession.getRecordCount();
                         System.out.println("RecordCount is: " + count);
                         ExecutorService pool = Executors.newFixedThreadPool(threadNum);
                         ArrayList<Callable<Long>> callers = new ArrayList<Callable<Long>>();
                         long start = 0;
                         long step = count / threadNum;
                         for (int i = 0; i < threadNum - 1; i++) {
                                 RecordReader recordReader = downloadSession.openRecordReader(
                                                 step * i, step);
                                 callers.add(new DownloadThread( i, recordReader, downloadSession.getSchema()));
                         }
                         RecordReader recordReader = downloadSession.openRecordReader(step * (threadNum - 1), count
                                         - ((threadNum - 1) * step));
                         callers.add(new DownloadThread( threadNum - 1, recordReader, downloadSession.getSchema()));
                         Long downloadNum = 0L;
                         List<Future<Long>> recordNum = pool.invokeAll(callers);
                         for (Future<Long> num : recordNum)
                                 downloadNum += num.get();
                         System.out.println("Record Count is: " + downloadNum);
                         pool.shutdown();
                 } catch (TunnelException e) {
                         e.printStackTrace();
                 } catch (IOException e) {
                         e.printStackTrace();
                 } catch (InterruptedException e) {
                         e.printStackTrace();
                 } catch (ExecutionException e) {
                         e.printStackTrace();
                 }
         }
 }
```

**Note:** 

The Tunnel Endpoint can be specified or left blank.

-   If specified, the downloading data goes through the specified Endpoint.
-   If not specified, the downloading data goes through public network Endpoint.
-   This paper gives the Tunnel Endpoint of East China 2 classical network. The Tunnel Endpoint settings of other regions can be referred to [Configure Endpoint](../../../../reseller.en-US/Prepare/Configure Endpoint.md#).

