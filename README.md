# hadoop

* ## 目錄:
  * ## [1. HDFS安裝及基本指令](#001)  
  * ## [2. YARN](#002)
  * ## [3.python撰寫mapreduce範例](#003)
  * ## [4.Java Api存取HDFS](#004)
  * ## [3.資料擷取模組Sqoop,Flume](#005)
  * ## [4.資料分析模組Pig,Hive](#006)

<h2 id="001">1. HDFS安裝及基本指令</h2>  
  
* 由Name Node 與 Data Node組成:  
  * Name Node 由 fsimage 與 edits log 組成:  
    - fsimage: HDFS 當時的 snapshot, 檔案較為龐大，讀寫較慢  
    - edits log: HDFS 自 snapshot 後的變更，edit log 檔案比較小  
* 權限控管:
  * 類似Linux的rwxrwxrwx做User, Group, Others 的權限控管
    - 644(rw-r--r--)
    - 755(rwx-r-xr-x)
    - 600(rw--------)
    - x 代表可以瀏覽目錄
    - w 代表可以刪除檔案
    - 用戶名=linux命令中的`whoami`
    - 組名等於 `bash -c groups`
  * EX: 
  ```JS
  hadoop fs -chmod 755 /data/test2.txt  #修改權限
  haddop fs –chown  #修改文件所有者    
  hadoop fs –chgrp  #修改文件所屬組  
  ```
  * Kerberos驗證機制:
    - 票券(Ticketing)驗證
    - 最廣為測試跟使用的第三方認證
    - 可以由接受終端做驗證
    - 使用者先向KDC做請求，請求完拿到 Ticket 後向Service發出請求並驗證 Ticket，驗證成功之後便可發出回應。
  * 或將Hadoop  Clouster 放到獨立網段
  * 允許登入方法
    - 使用 ACL
    - 使用跳板(所有人都必須先連到跳板做驗證才能夠使用)
* Secondary Name Node:
  * 非及時備援(Non-Hot Failover)
  * 監控 Name Node 運行狀態
  * 排程檢核(每個小時):從 Name Node 處備份 fsimage 跟 edits log，將 fsimage 與 edits log 合併為較大的 fsimage
### 安裝(ubuntu):
1. 修改本機hostname:
  ```js
  vi /etc/hostname
  ```
  * centos: 
    ```js
    sudo groupadd hadoop #建立group
    sudo useradd -g hadoop hadoop  #建立user
    ```
2. 建立金鑰:
   ```js
   ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
   cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys   
   ```    
  * scp authorized_keys到各個slave內
  * 如果沒有.ssh資料夾，就先`ssh localhost` 再登出就會出現了
  
  
3. 安裝java
  * 下載jdk1.8
  ```js
  sudo apt-get install openjdk-8-jdk
  ```
  * 測試 
  ```js 
  java -version
  ```  
  
4. 安裝hadoop
  * 解壓縮
  ```js
  tar zxvf hadoop-2.10.1.tar.gz
  ```
  * 移動到 home目錄
  ```js
  mv hadoop-2.10.1 ~/
  ```
  * 修改 `core-site.xml` 檔
  ```js
  cd ~
  cd hadoop-2.10.1/etc/hadoop/
  vim core-site.xml
  ```
  * 找到下面:
  ```js
  <!-- Put site-specific property overrides in this file. -->

  <configuration>
  </configuration>           
  ```
  * 覆蓋  
      <name>fs.defaultFS</name>  (設定HDFS登入位置，port號為9000)  
      <name>hadoop.tmp.dir</name>  (設定保存臨時文件位置，預設是/tmp/hadoop-hadoop)  
      ex:  
     ```js
     <configuration>
          <property>
               <name>hadoop.tmp.dir</name>
               <value>/home/${user.name}/tmp</value>
               <description>Abase for other temporary directories.</description>
          </property>
          <property>
               <name>fs.defaultFS</name>
               <value>hdfs://master:9000</value>  #注意路徑(HDFS預設9000port)
          </property>
     </configuration>
     ```

   * 修改 `hdfs-site.xml` 檔
     ```js
     cd ~/hadoop-2.10.1/etc/hadoop/
     vim hdfs-site.xml
     ```
  * 一樣修改   
    <name>dfs.namenode.secondary.http-address</name>  主要是設定Name Node欲保存的Metadata之儲存目錄    
    <name>dfs.datanode.data.dir</name> Data Node欲保存的資料之儲存目錄  
    <name>dfs.replication</name>  儲存資料之副本數  
    ex:  
     ```js
     <configuration>
           <property>
                   <name>dfs.namenode.secondary.http-address</name>  
                   <value>master:50090</value>
           </property>
           <property>
                   <name>dfs.namenode.name.dir</name>
                   <value>/home/${user.name}/hdfs/namenode</value> 
           </property>
           <property>
                   <name>dfs.datanode.data.dir</name>
                   <value>/home/${user.name}/hdfs/datanode</value>
           </property>
           <property>
                   <name>dfs.replication</name>
                   <value>3</value>
           </property>
     </configuration>
     ```
  * 修改`slave` 檔，加上所有slave的名稱 :
     ```js
     master
     slave1
     slave2
     ```
     
  * 並於/etc/hosts 修改各機器ip及主機名稱 (每台都要)
    ```js
    127.0.0.1       localhost
    192.xxx.x.xxx   master
    192.xxx.x.xxx   slave1
    192.xxx.x.xxx   slave2
    ```
  * 修改 `hadoop-env.sh` 檔，在底下添加:
    ```js
    export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 
    export PATH=$JAVA_HOME/bin:$PATH

    export HADOOP_HOME=/home/${USER}/hadoop-2.10.1 #注意版本
    export PATH=$HADOOP_HOME/bin:$PATH

    export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop  
    ```
  * 修改`~/.bashrc`檔，底下添加:(記得source)
    ```js
    export HADOOP_HOME=/home/${USER}/hadoop-2.10.1 #注意版本
    export PATH=$HADOOP_HOME/bin:$PATH

    export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
    ```  
  * scp 整個hadoop資料夾、jave資料夾、`~/.bashrc` 檔到各個slave:(記得每台都要source)
  ```js
  scp -r hadoop-2.10.1xxx@xxx.xxx.x.xxx:~
  ```
  * 格式化hadoop:
  ```js
  hadoop namenode -format
  ```
  * 啟動hadoop:
  ```js
  cd ~
  cd hadoop-2.10.1sbin
  ./start-dfs.sh
  ```  
  
#參考資料:[https://medium.com/@sleo1104/hadoop-3-2-0-%E5%AE%89%E8%A3%9D%E6%95%99%E5%AD%B8%E8%88%87%E4%BB%8B%E7%B4%B9-22aa183be33a](https://medium.com/@sleo1104/hadoop-3-2-0-%E5%AE%89%E8%A3%9D%E6%95%99%E5%AD%B8%E8%88%87%E4%BB%8B%E7%B4%B9-22aa183be33a)

* 基本操作:[官方文件](https://hadoop.apache.org/docs/r1.0.4/cn/hdfs_shell.html#FS+Shell)
* 出現異常:
```js
hadoop@master:~/sparkk/spark101/wordcount/data$ hadoop fs -mkdir hdfs://master/user/
mkdir: Call From master/192.168.1.69 to master:8020 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused
```
  解決方法:
  * core-site.xml檔的9000port改成8020port
  
  
<h1 id="002">2. YARN(MRV2)</h1>  

1. 特色:
  * 傳統MRV1缺點:
    *  特色: 
        1. 由 JobTracker 與 TaskTracker 所組成(也就是Daemon 代理運行服務): 
        2. JobTracker(Master Daemon): 扮演 Master 角色，管理所有工作與資源，會盡可能將工作執行與欲處理的資料放安排同一台機器(老闆)
        3. TaskTracker(Slave Daemon): 扮演 Slave 角色，按照 JobTracker 指示執行Map 或 Reduce 工作(工人)
    * 缺點: 
       1. 延展性差: JobTracker 同時具備資源管理與作業控制功能
       2. Master 容錯性低: Master 壞掉就會導致單點故障讓所有工作失敗
       3. 資源利用率低: Map 跟 Reduce 之間資源無法共享
       4. 無法資源多個計算框架共存: EX: Spark
  * 傳統MRC1運作方式:
    * Client 送出工作給 JobTracker
    * JobTracker 詢問 Name Node 關於資料位置
    * JobTracker 根據資料(本地性)決定由哪幾個 TaskTracker 處理資料
    * 傭有資料處的 TaskTracker 會開始啟用 Mapper  
    * 執行完工作之後會將中介結果寫到本地端的磁碟
    * 空閒的 TaskTracker(沒有資料本地性) 會開始啟用 Reducer
    * 將Mapper 的中介結果複製到 Reducer 處進行 Reduce
    * 將處理結果寫回HDFS  
    * 當 Mapper 運作時，會將產生的中介資料寫在本地端，工作完成之後會被移除
    * (預測執行)當有工作的預測速度比其他節點要慢時，JobTracker 會再將工作分配給其他節點，兩個比賽誰先完成就吐出結果
    * 當 TaskTracker 通知 JobTracker 工作結束時，會在HDFS中寫入一個較_SUCCESS的檔案
    * JobTracker 會通知 TaskTracker 移除中介資料
    * 會保留 log  
  * 使用 YARN:  
    * 整合異質計算框架: MapReduce, Spark, MPI  
    * 資源利用率高: 資源共享
    * 維運成本低: 透過少數資源管理器就能管理許多框架
    * 想像 YARN 代表 Linux(擁有 Resource Manager, Node Manager)，ApplicationMaster 就是 Process，Task 就是 Thread。
    * YARN 所包含的原件: 
      - Resource Manager: 取代原本 JobTracker資源管理功能，由Scheduler 跟 Application Manager 所組成:  
          Resource Scheduler:將系統資源分配給運行的應用程序   
          Application Manager: 管理系統中的應用程序     
      - Application Master: 作業控制(檢查TaskTracker及工作執行的狀態)，與Resource Manager要資源，監控或分派工作給 map task 或 reduce task ，告知 Node Manager 啟動或停止  
      - Node Manager: 負責每個節點(slave)上的資源與任務管理，定時向 Resource Manager 回報目前 Node 跟 container 的運行狀況
      - Container: 根據需求動態配置資源(MRV1配置資源是固定的)，將YARN中的RAM,CPU 磁碟抽象化，當 Application Master 向 Resource Manager 申請資源，Resource Manager 會以 Container 的方式向 Resource Manager 提供資源(每個TASK會被分配一個Container)。
    * YARN 工作流程: 
      * 首先 Client 把工作送到 Resource Manager(包含 Scheduler 跟 Application Manager)  
      * 接下來 Application Manager 會跟 Node Manager 做通訊 由 Node Manager 開啟 Application Master 
      * 由 Application Master 跟 Resource Manager 溝通並建立連線，Resource Manager便可監控 Application Master 運行狀況  
      * Application Master 會依任務狀況跟 Resource Scheduler 要資源 
      * Application Master 要到資源後會與相對應的 Node Manager 溝通  
      * Node Manager 會依任務需求把 Container 等需求環境準備好並定期回報Node的狀況
      * 準備好環境之後產生 Task ，Task會隨時向 Application Master 溝通，即時掌握 Task 運行狀況
      * Client 可以隨時利用 Application Master 掌握狀況  
      * 所有工作結束之後 Application Master 會通知 Resource Manager ，並把工作從 Resource Manager 工作清單中移除。
      * 移除掉之後把環境關閉。
  * 改進 MRV1 的延展性跟異質性框架相容性
    * 延展性:
      1. 將 JobTracker 中的作業控制及資源管理分開，減少 Loading
      2. 使用Resource Manager 資源管理與調度
      3. 動態分配Map Slot 與 Reduce Slot: JobTracker中 Map Slot 與 Reduce Slot 分配是固定的，導致 map 時 Reduce Slot 呈現空閒。
    * 同時使用各種異質框架:
      - 可替換上面的計算框架: EX: 使用MPI, Spark
      
      
2. 安裝
  * 修改 `yarn-site.xml` 檔
  ```js
  <configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
  </configuration>
  ```
  * 修改 `mapred-site.xml.template`檔
  ```js
  <configuration>
    <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
    </property>
  </configuration>
  ```  
  *  開啟yarn:
  ```js
  ./start-yarn.sh
  
  # 查看yarn node
  yarn node -list
  ```
    * Resource Manage: 8088 port
    * Node Manager: 8042 port

<h1 id="003">3.python撰寫mapreduce範例</h1>  


* mapper.py:  
 ```js
 import sys
 for line in sys.stdin:
     line = line.strip()
     words=line.split()
     for word in words:
         print('%s\t%s' % (word,1))
 ```
* reducer.py:
```js
from operator import itemgetter
import sys

current_word = None
current_count = 0
word = None

# input comes from STDIN
for line in sys.stdin:
    line = line.strip()
    word, count = line.split('\t', 1)
    try:
        count = int(count)
    except ValueError:
        continue
    if current_word == word:
        current_count += count
    else:
        if current_word:
            print ('%s\t%s' % (current_word, current_count))
        current_count = count
        current_word = word
        
# do not forget to output the last word if needed!
if current_word == word:
    print ('%s\t%s' % (current_word, current_count))

```
* 執行:
```js

hadoop jar ~/hadoop-2.10.1/share/hadoop/tools/lib/hadoop-streaming-2.10.1.jar -file ~/mapper.py -mapper mapper.py -file ~/reducer.py  -reducer reducer.py -input /test.txt -output /data

#原理
echo 隨便一段英文句子 |python mapper.py|sort -k 1|python3 reducer.py

#觀看結果
hadoop fs -cat /data/part-00000

```

<h1 id="004">4.Java Api存取HDFS</h1>  

* 下載eclipse 解壓縮並安裝(jdk1.8與2019-06版本相容，太高不行)
* 先 new 一個 Project -> 右鍵 build path -> Add External Archives
* import以下:
```js
~/hadoop-2.10.1/share/hadoop/hdfs/hadoop-hdfs-2.10.1.jar
~/hadoop-2.10.1/share/hadoop/hdfs/hadoop-hdfs-client-2.10.1.jar
~/hadoop-2.10.1/share/hadoop/common/hadoop-common-2.10.1.jar
~/hadoop-2.10.1/share/hadoop/common/lib/commons-cli-1.2.jar
~/hadoop-2.10.1/share/hadoop/common/lib/commons-collections-3.2.2.jar
~/hadoop-2.10.1/share/hadoop/common/lib/commons-configuration-1.6.jar
~/hadoop-2.10.1/share/hadoop/common/lib/commons-lang-2.6.jar
~/hadoop-2.10.1/share/hadoop/common/lib/commons-logging-1.1.3.jar
~/hadoop-2.10.1/share/hadoop/common/lib/guava-11.0.2.jar
~/hadoop-2.10.1/share/hadoop/common/lib/hadoop-auth-2.10.1.jar
~/hadoop-2.10.1/share/hadoop/common/lib/log4j-1.2.17.jar
~/hadoop-2.10.1/share/hadoop/common/lib/protobuf-java-2.5.0.jar
~/hadoop-2.10.1/share/hadoop/common/lib/slf4j-api-1.7.25.jar
~/hadoop-2.10.1/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar
~/hadoop-2.10.1/share/hadoop/common/lib/woodstox-core-5.0.3.jar

或是可以iclude整個lib裡面的jar檔
```
* example: 
  * getdirlist.java(讀取目錄內容):  
   ```js
   package hdfsop;

   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.FileStatus;
   import org.apache.hadoop.fs.FileSystem;
   import org.apache.hadoop.fs.Path;

   public class GetDirList {
    public static void main(String[] args) throws Exception {

     Configuration configuration = new Configuration();  //new一個 Configuration 的物件
     configuration.addResource(new Path(                 //路徑(引入core-site.xml)
       "/usr/local/hadoop/etc/hadoop/core-site.xml")); 
     configuration.addResource(new Path(                 //路徑(引入hdfs-site.xml)
       "/usr/local/hadoop/etc/hadoop/hdfs-site.xml"));
     configuration.set("fs.hdfs.impl",                   //透過DistributedFileSystem做存取
     org.apache.hadoop.hdfs.DistributedFileSystem.class.getName());  
     FileSystem fs = FileSystem.get(configuration);      //將 Configuration 餵到 FileSystem.get
     FileStatus[] status = fs.listStatus(new Path("hdfs://master:9000/data/"));  //讀取路徑下所有檔案讀出來
     for (int i = 0; i < status.length; i++) {
      System.out.println(status[i].getPath());  //把檔案的名稱print出來

     }
    }
   }

   ```
  * writehdfsfile.java(寫入檔案):
  ```js
   package hdfsop;

   import java.io.BufferedWriter;
   import java.io.OutputStreamWriter;

   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.FileSystem;
   import org.apache.hadoop.fs.Path;

   public class WriteHDFSFile {
    public static void main(String[] args) throws Exception {

     Configuration configuration = new Configuration();
     configuration.addResource(new Path(
       "/usr/local/hadoop/etc/hadoop/core-site.xml"));
     configuration.addResource(new Path(
       "/usr/local/hadoop/etc/hadoop/hdfs-site.xml"));
     configuration.set("fs.hdfs.impl",
       org.apache.hadoop.hdfs.DistributedFileSystem.class.getName());
     FileSystem fs = FileSystem.get(configuration);
     Path pt = new Path("hdfs://master:9000/data/tibame.txt");  //打開一個tibame.txt的檔案
     BufferedWriter br = new BufferedWriter(new OutputStreamWriter(
       fs.create(pt, true)));
     String line;               //透過BufferedWriter 的OutputStreamWriter方法寫入檔案
     line = "Tibame Hadoop\n";  //寫入內容
     System.out.println(line);  //寫入一行就印出一行
     br.write(line);
     br.close();               //關檔
    }
  ```
  
  * getfilecontent(讀取目錄內容):
    ```js
    package hdfsop;

    import java.io.BufferedReader;
    import java.io.InputStreamReader;

    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.FileStatus;
    import org.apache.hadoop.fs.FileSystem;
    import org.apache.hadoop.fs.Path;

    public class GetFileContent {

     public static void main(String[] args) throws Exception {

      Configuration configuration = new Configuration();
      configuration.addResource(new Path(
        "/usr/local/hadoop/etc/hadoop/core-site.xml"));
      configuration.addResource(new Path(
        "/usr/local/hadoop/etc/hadoop/hdfs-site.xml"));
      configuration.set("fs.hdfs.impl",
        org.apache.hadoop.hdfs.DistributedFileSystem.class.getName());
      FileSystem fs = FileSystem.get(configuration);
      FileStatus[] status = fs.listStatus(new Path(
        "hdfs://master:9000/data/tibame.txt"));
      BufferedReader br = new BufferedReader(new InputStreamReader(
        fs.open(status[0].getPath())));  //讀取第一個檔案的路徑
      String line = br.readLine();  //逐行讀出
      while (line != null) {
       System.out.println(line);
       line = br.readLine();
      }

     }
    }

    ```
* MapReduce型態與格式:
  * [Interface Writable](https://hadoop.apache.org/docs/stable/api/org/apache/hadoop/io/Writable.html) 在套件org.apache.hadoop.io 底下
    * EX: IntWriteable,LongWriteable,Text
    * 屬於原生 Primitive Type(intWriteable,longWriteable 等等) Array, Map
    * 可對應到Java 原生型態 EX: intWriteable = int 
    * 使用 utf-8 編碼 
    * 所有 mapper 與 reducer 都必須使用 Interface Writable 的資料型態
  * [Input](https://hadoop.apache.org/docs/r2.8.4/api/org/apache/hadoop/mapreduce/lib/input/package-summary.html): 
    * 在套件 org.apache.hadoop.mapreduce.lib.input 底下
    * InputFormat: FileInputFormat, SequenceFileInputFormat, TextInputFormat 等等
    * RecordReader: SequenceFileRecordReader, CombineFileRecordReader
    * 主要提供如何將資料切成可供Map 階段執行的資料分割(splits), 確定Map Task 的個數，產生 RecordReader 類別以從資料分割產生一連串 <k,v>
    * InputFormat 跟 RecordReader 是 intput 與 key/value 間溝通的橋樑
  * [Splitter](https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/mapreduce/InputSplit.html) 在套件org.apache.hadoop.mapreduce.InputSplit 底下 
    * 紀錄資料的 Metadata
    * 可序列化，做為資料交換用
    * 屬於邏輯分割，實際上資料還是以資料塊 Chunk 為單位進行儲存
    * 為 Mapper 的輸入(spli 輸入 mapper)，一個 Split 對應一個 Mapper，透過 RecordReader 轉成 key/value
  * [Output](https://hadoop.apache.org/docs/stable/api/org/apache/hadoop/mapreduce/lib/output/package-summary.html) 在套件 org.apache.hadoop.mapreduce.lib.output 底下 
    * FileOutputFormat, NullOutputFormat ...etc 寫入
    * 在
* Mapper 類別:
   ```js
               //k1, v1:input key/value
               //k2, v2:output key/value
   class Mapper<k1, v1, k2, v2>
   {
   void map(k1 key, v1 value Mapper.Context context)  //吃下k1, v1，與 hadoop framwork 溝通產生 map 或reduce 的結果
   throws IOException, InterruptedException //定義例外
   {...}
   }
   ```
  * override方法
    * 當 key/value 還沒出現於 map, 執行 setup 動作
      ```js
      protected void setup( Mapper.Context context)
      throws IOException, InterruptedException //定義例外
      ```
    * 當 key/value 已出現於 map, 執行 cleanup 動作
      ```js
      protected void cleanup( Mapper.Context context)
      throws IOException, InterruptedException //定義例外
      ```
    * 流程控制: 預設事先執行 setup, 接著執行 map, 最後執行 cleanup
      ```js
      protected void run( Mapper.Context context)
      throws IOException, InterruptedException //定義例外
      ```
* Reducer 類別:
   ```js
               //k2, v2:input key/value
               //k3, v3:output key/value
   class Reducer<k2, v2, k3, v3>
   {
   void map(k2 key, Iterable<v2> value Mapper.Context context)  //吃下k1, v1，與 hadoop framwork 溝通產生 map 或reduce 的結果
   throws IOException, InterruptedException //定義例外
   {...}
   }
   ```
  * override方法
    * 當 key/value 還沒出現於 map, 執行 setup 動作
      ```js
      protected void setup( Reducer.Context context)
      throws IOException, InterruptedException //定義例外
      ```
    * 當 key/value 已出現於 map, 執行 cleanup 動作
      ```js
      protected void cleanup( Reducer.Context context)
      throws IOException, InterruptedException //定義例外
      ```
    * 流程控制: 預設事先執行 setup, 接著執行 map, 最後執行 cleanup
      ```js
      protected void run( Reducer.Context context)
      throws IOException, InterruptedException //定義例外
      ```
* Driver 類別:
```js
public class ExampleDriver {
...
public static void main(String[] args) throws Exception{
 Configuration conf = new Configuration();  //設定 Configuration
 Job job = Job.getInstance(conf, "Example"); //設定工作
 job.setJarByClass(ExampleDriver.class); //設定主類別名稱
 job.setMapperClass(ExampleMapper.class); //設定 Mapper
 job.setReducerClass(ExampleReducer.class); //設定 Reducer
 job.setOutputKeyClass(Text.class); //產出 key 資料類型
 job.setOutputValueClass(IntWritable.class); //設定產出value 資料類型
 //FileInputFormat.addInputPath(job, new Path(  //設定Input 路徑
 //		"hdfs://localhost:9000/data/pg1104.txt"));
 //FileOutputFormat.setOutputPath(job, new Path(  //設定Output 路徑
 //		"hdfs://localhost:9000/out8/"));
 FileInputFormat.addInputPath(job, new Path(args[0]));  //設定Input 路徑
 FileOutputFormat.setOutputPath(job, new Path(args[1]));  //設定Output 路徑
 System.exit(job.waitForCompletion(true) ? 0 : 1);  //執行工作並等待結束
	}
}
```
  
* sort 與 shuffle: 
  * MapReduce 產生資料的方式:
    * map -> shuffle -> reduce
  * 資料會在map 階段跟 reduce 階段都會進行資料排序
    * 以key 做 sorting
    * 如果要以value 做sorting 可以定義secondary sort
    * secondary sort 適合用來找出最大最小值
    * 使用自定義的WriteableComparator 或使用 Comoarator類別
    * job.setSortComparatorClass(Reducer.class)
* Combiner: 
  * 可以想像是pre-reducer
  * 會在Mapper 接換就預先Reduce 結果: 例如wordcount 問題中可預先針對同樣key 隊到的值進行加總
  * 可以大量降低中介(Intermediate File)檔案的尺寸:EX: 傳輸(cat,1),(cat,1)與(cat,2)之間的差別
  * job.setSortCombinerClass(Reducer.class)
  
* Partitioner: 
  * 控制哪個key 應該到哪個Reducer
  * 預設會分散key 到各個Reducer
  * 可以使用TotalOrderPartitioner 針對key 做排序
  * 可以設定Custom Partitioner: job.setPartitionerClass(Partitioner.class)
* 整體流程: 
  1. Mapper 產生key/value pair
  2. 會將資料寫入Partition 檔案(Intermediate File)，數量由Reducer 決定
  3. 每個Partition 檔案會經過排序(可用Comparator自訂排序)
  4. Comparator 會開始pre-reducer
  5. Reducer會到Mapper 取得Partition 檔案
  6. 當所有Mapper 執行完畢，且所有中介檔案被複製到Reducer 後會在執行一次排序
  7. 執行Reducer 
  
<h2 id="005">3.資料擷取模組Sqoop,Flume</h2>   

### Sqoop
Sqoop1 :  
* 透過JDBC 連線RDB 與 HDFS (Import Export)
* SQL to Hadoop
* 使用MR 的Map 去查詢或新增資料
* 預設使用4個Mapper 
* 可以限制占用頻寬  
* 不能進行過濾或去重複的動作
* 通常使用定期匯入與匯出的工作
* Sqoop 與資料庫連線蒐集Metadata 再由每個Mapper 分別去資料庫抓取資料，並寫入HDFS 或RDB
* 支援Incremental Updates(整數, Timestamp) 從記錄點開始做持續匯入、出的動作
* 使用SQL限制匯入的欄與列
* 可以匯入CSV或Arvo格式檔案，亦支援大型物件(Blob,Clob)
* 支援部分資料庫檢視語法(Show, List)
Sqoop1 VS Sqoop2:
* Sqoop1: 
  * Client Model(connector 跟Driver 都裝在client 上)
  * 只送出map工作
* Sqoop2: 
  * Service Model(connector跟Driver 都裝在Server 上)
  * Mapper 轉移資料，Reducer 轉換資料
  * 提供更佳的安全性(server權限控管)
* sqoop安裝:
  * 下載: https://downloads.apache.org/sqoop/1.4.7/
  * 解壓縮然後移動到home目錄  
  * 編輯`~/.bashrc`:
   ```js
   export SQOOP_HOME=/home/${USER}/sqoop-1.4.7.bin__hadoop-2.6.0
   export PATH=$PATH:$SQOOP_HOME/bin:$HADOOP_HOME/bin  #這行和HADOOP_HOME合併
   ```
  * 123找到sqoop/bin/configure-sqoop 把以下echo warning的地方都註解掉:  
    export HBASE_HOME  
    export HCAT_HOME  
    export HIVE_CONF_DIR  
    export ACCUMULO_HOME  
    export ZOOKEEPER_HOME(多行註解Ctrl+v然後按下，選取完之後按shift+i輸入要註解的符號(例如#)，然後按esc)  
  * 
* Sqoop2安裝:
  * 下載: https://downloads.apache.org/sqoop/1.99.7/
  * 解壓縮然後移動到home目錄
  * 編輯`~/.bashrc`:
   ```js
   export SQOOP_HOME=/home/${USER}/sqoop-1.99.7-bin-hadoop200
   export PATH=$PATH:$SQOOP_HOME/bin:$HADOOP_HOME/bin  #這行和HADOOP_HOME合併
   ```
  * source完之後執行`sqoop2-tool verify`:   
   如果出現failed，編輯conf 目錄底下的sqoop.properties:  
   ```js
   #找到這行，等於後面改成/home/test/hadoop-2.10.1/etc/hadoop/  #自己注意路徑(必須要是絕對路徑)
   org.apache.sqoop.submission.engine.mapreduce.configuration.directory=/home/test/hadoop-2.10.1/etc/hadoop/
   ```
 * 開啟sqoop2:
  ```js
  sqoop2-server start

  #關閉
  sqoop2-server stop
  
  #確認是否正確開啟
  wget -qO - localhost:12000/sqoop/version
  
  #開啟sqoop shell
  sqoop2-shell
  ```
* 使用sqoop 轉移結構化資料:   
  * [下載MySQL，底下有txt載入](https://github.com/a13140120a/SQL_note/blob/master/README.md)
  * Create Table
  * [載入txt 檔資料](https://github.com/a13140120a/hadoop/blob/main/ratings.txt)
  * [下載jdbc並放到sqoop底下的lib目錄中](https://github.com/a13140120a/SQL_note/blob/master/README.md#002)(注意版本!!)  
  * create table:
  ```js
  CREATE TABLE ratings(
  userid INTEGER,
  itemid INTEGER,
  ratings INTEGER
  );
  ```
  * 測試 :
  ```js
  sqoop list-databases --connect jdbc:mysql://mysql:3306/ --username user --password 000000
  ```
  * 若出現問題: ERROR manager.SqlManager: Error executing statement: java.sql.SQLException: Access denied for user 'test'@'localhost' (using password: NO)，是因為權限不足。
  ```js
  #賦予權限
  GRANT ALL ON *.* TO 'newuser'@'localhost';
  ```
  * Import:
  ```js
  sqoop import 
  --connect jdbc:mysql://localhost:3306/db_name 
  -username user 
  --query "select * from ratings where XXX AND \$CONDITIONS limit 200" 
  #如果query後使用的是雙引號，則$CONDITIONS前必須加反斜線，防止shell識別為自己的變數
  -target-dir /data/  #HDFS的路徑
  --password aaa123 
  --split-by "userid"  #以哪個欄位做區分然後(必要)
  -m 2  # 幾個mapper去執行任務
  ```
  * Export: 
  ```js
  sqoop export 
  --connect jdbc:mysql://localhost/test 
  --table ratings2 
  --export-dir /data/ 
  --username user 
  -m 2 
  --password aaa123
  --fields-terminated-by ',' # 表示HDFS檔案內容使用','去分隔欄位  
  --hive-import  #直接匯入到hive中
  ```
  * [連線hive](https://docs.cloudera.com/HDPDocuments/HDP2/HDP-2.6.0/bk_data-access/content/using_sqoop_to_move_data_into_hive.html)

### Flume #

* 只能單向匯入HDFS
* 從不同來源蒐集(Collect)、聚合(Aggregate) 資料串流(Streaming)
* 可水平或垂直擴充
* 可加值處理，去重複化，清理資料，過濾資料 ...等等
* 具交易性:使用ack 確定下游downsteam 有收到資料，否則上游(stream)會重送資料。
* HA 及平行擴充:
  - Hot/Standby Mode (及時備援)
  - 可以平行擴充成多台上游(Upstream)蒐集資料
* 每個Flume 的執行程式接命名為 Agent
* Agent 皆包含: 
  * Source(讀取Log File來源，將事件傳入Channel)
    - 可從特定檔案取得資料:  
      \- Event Driven 或Pollingbased  
      \- 從特定檔案取得資料(tail -f file)  
      \- 從特定連結取得資料(192.1.1.10:8888)  
  * Channel(Source 與Sink 之間的緩衝，負責緩存事件)
    * 負責Buffer 之間傳遞的訊息  
      \- Memory  
      \- Disk  
      \- JDBC  
  * Sink(取出事件，將資料放置於HDFS)
    - 負責儲存資料:    
      \- Polling Based  
      \- Localhost:9988(傳到另一個service)  
      \- hdfs:// (寫入HDFS)  
* 安裝:
  * [官網](http://www.apache.org/dyn/closer.lua/flume/1.9.0/apache-flume-1.9.0-bin.tar.gz)
  * 下載解壓縮安裝並mv 到~ 目錄
  * 修改.bashrc 並source: 
  ```js
  export FLUME_HOME=/home/${USER}/apache-flume-1.9.0-bin/
  export FLUME_CONF_DIR=/usr/local/flume/
  export PATH=$PATH:$FLUME_HOME/bin
  ```
  * 測試: 
  ```js
  flume-ng
  ```
* Source 設定:

  |類型|簡介|
  | --- | --- |
  |avro|1. 提供一個以avro 為協議的服務，並bind 到某個port 上，等待avro 協定用戶端發過來的訊息<br>2. avro 為一種序列化的資料交換格式<br>3. 一般在agent 之間傳輸資料時，可以配置為avro|
  |thrift|1. 使用Thrift作為傳輸協議<br>2. 跟avro 類似，交換過程較麻煩，已逐漸被avro取代|
  |exec|執行一個unix 指令，以stdout 做為資料來源，命令可以通`<agent>.<source>.command`來配置，如: tail -f /var/log/mesg 指令出現的stdout |
  |netcat|1. 監控特定port 每一行作為一個事件的傳輸<br>2. 假使輸入的資料為text，每一行資料最大長度(max-line-lengrh)預設為512|
  |http|支援http 的post 和get|
  |scribe|org.apache.flume.source.scribe.ScribeSource|
  |syslogtcp<br>syslogudp|監聽syslog，支援tcp 及udp|
  |sequence source(seq)|自動產生編號|
  |spooling directory source|1. 監控某個目錄下的所有檔案，如果有新檔案進來則將其新加入的檔案做為資料來源傳輸<br>2. 每傳輸一個檔案後，會被rename 成其他名字(表示已經傳輸過)或者刪掉<br>3. 預設監控目錄下的檔具有:immutable、uniquely-named屬性|
  |jims|從訊息佇列獲取資料，active message queue|
 
* Channel 設定: 
  |類型|簡介|
  | --- | --- |
  |memory|訊息放在記憶體中，提供高吞吐量，但容錯性低，可能因為裝載的機器毀損而導致資料消失|
  |file|1. 對資料容錯性高，但是配置較為複雜，需要配置資料目錄和checkpoint目錄<br>2. 不同的file channel 均需配置一個checkpoint 目錄|
  |jdbc|使用內建的java資料庫derby，對event 進行了高容錯性<br>2. 可來取代file channel<br>3. 提供一般資料庫交易t(ransation)的特性，所以也比file channel 更可靠|

* Sink 設定:

  |類型|簡介|
  | --- | --- |
  |hdfs|將數據寫道hdfs上|
  |logger|1. 採用logger，logger 可以配置輸出到控制台stdout，也可輸出到檔案<br>2. logger 對訊息長度有限制，超過限制會截斷訊息|
  |avro|發送給另一個avro 的source|
  |thrift|發送給另一個thrift 的source|
  |irc|Internet Relay Chat|
  |file_roll|1. 本地檔案，支援檔案輪替(rotate)<br>2. 可設定大小、時間、事件記數來進行輪替|
  |null|丟棄事件|
  |hbase asynchbase|寫到hbase中，需搭配hbase 的table、columnFaimly|
  |Solr|將資料傳遞至Solr|
  |Elasti Search|將資料傳遞至Elasti Search|
  |$FQCN|寫到特定FQCN|

* Flume Interceptor
  * Flume 的擴充元件
  * 可以在資料即時傳輸的過程中更改資料，或做資料加值(擴充新的Source, Sink, Channel)
  * 可以在Source, Sinks 之間增加資料處理功能
    - 例如可以增加正規表達式，做抽取或過濾的功能
    - 客製的擴充腳本必須是Java
  * 實作透過org.apache.flume.interceptor.Interceptor 介面
  * 現有Interceptor :
    - Timestamp Interceptor: 於是建標頭上寫入時間標籤
    - Host Interceptor: 於事件標頭上寫入Agent 來源
    - Static Interceptor: 使用者可以在標頭上增添任意值
    - UUID Inceptor: 可以為每個事件加註UUID

* 維護Flume 
  * 使用Deployment 或Config 管理工具
    - 透過其他軟體同時間散佈到大量機器上 EX: Puppet, Chef
  * 散佈出去之後Config 檔會隨著property 的變更即時更改，不用重啟Agent
* 實際範例:
  * 介紹:
    * [agent1](https://github.com/a13140120a/hadoop/blob/main/agent1.properties)
    * [agent2](https://github.com/a13140120a/hadoop/blob/main/agent2.properties)
    * [log-generator.py](https://github.com/a13140120a/hadoop/blob/main/log-generator.py)
    * 由`log-generator.py` 產生假的log資料寫入/tmp/log-generator.log 再由agent1 蒐集/tmp/log-generator.log 產生的資料並導入agent2 ，最後由agent2 寫入HDFS。
  * 流程:
    * 於檔案所在的當前目錄啟動log_generator.py, agent2 及agent1:(一隻一個bash)
      ```js
      python log_generator.py
      flume-ng agent --conf-file agent2.properties --name agent2
      flume-ng agent --conf-file agent1.properties --name agent1
      ```
    * 就可以於HDFS 中看到產生的log 資料了

<h2 id="006">4.資料分析模組Pig,Hive</h2>  

## Hive & Pig ##
* 讓非開發者也能輕鬆操作Hadoop
* Hive - SQL Like Language
* Pig - Data Flow Language
* 比原生Java 平均大約慢10~15%
* 讓開發者更有生產力，節省寫Code 的時間，把更多時間花在資料處理
* 使用情境:
  * hive
    * 當有明確的Schema
    * 當資料適合使用SQL
    * 高容錯姓
  * Pig
    * 可以執行複雜的ETE
    * Pig提供較Hive 更多控制
    * 高容錯姓
  * Impala
    * 有明確的Schema
    * 即時性互動查詢
    * 低容錯性的即時分析
----

### Hive #  
* Facebook 所開發
* SQL-Like 語言(偏向MySQL)
* Hive 是建立於Hadoop 之上的資料倉儲套件
* Hive 可以以Client 的形式安裝
* 或是可以使用網路服務存取，EX: 架設在server 上的 Hue(Web interface) 或者HiveServer2(ODBC, JDBC)
* 需要 metastore: 提供Hive 資料的映對(Mapping)，儲存table, 路徑, query 等等，轉換成mapreduce工作並送往cluster
* Hive 會跟 Pre-built 的Binaries 會轉換成Jar 檔去submmit 工作，並透過MapReduce 的工作執行 Query。
* 不會產生 Java 原始碼，所以不能做修改
* 優點:  
  * 簡潔方便、較易學習(相較Jave 而言)
  * Ad-Hoc 分析(比 Impala慢): 回答單一、確定的商業問題
  * 可以依時間分割資料(Partition)(加速查詢)
  * DBA(資料庫管理員) 可以重複使用部分Query(同MySQL 語法)
  * 使用共用的View 可以加入分析
    * 專注在資料表的建立與寫入
    * 共用的view 可以加速分析
* 缺點: 
  * 無法即時分析(需使用impala)
  * 非高效能(需使用impala)
  * 如需要使用資料ETL(請使用PIG)
  * 如需要精細控制流程，例如ifelse(請使用PIG)
  * 難以處理非結構化資料(請使用PIG)
* 與傳統資料庫比較:
  * 不提供:
    - row-level insert
    - updates
    - deletes
    - Transactions
  * 並非資料庫
  * Hive 並非用作線上交易
  * Hive 並不提供即時查詢 (可使用impala 替代)
* 安裝:
  * [下載](https://downloads.apache.org/hive/hive-2.3.7/)
  * 解壓縮然後移動到home目錄
  * 編輯`~/.bashrc`:
  ```js
  export HIVE_HOME=/home/${USER}/apache-hive-2.3.7-bin
  export PATH=$PATH:$HIVE_HOME/bin:$HIVE_HOME/conf
  ```
  * 設定檔hive-default.xml介紹:(連線MySQL)
  ```js
  cp hive-default.xml.template hive-site.xml
  vim hive-default.xml
  
  <property>
    <!--  this should eventually be deprecated since the metastore should supply this -->
    <name>hive.metastore.warehouse.dir</name>
    <value>/user/hive/warehouse</value>
    <description>指定Hive 資料儲存目錄，為HDFS路徑，可改可不改</description>
  </property>
  <property>
    <name>hive.exec.scratchdir</name>
    <value>/tmp/hive</value>
    <description>指定Hive 資料暫存檔案目錄，為HDFS路徑</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://localhost:3306/hive_metadata?createDatabaseIfNotExist=true</value>
    <description>連線mySQL資料庫，預設是使用內建derby資料庫</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
    <description>jdbc名稱，這邊要改</description>
  </property>
    <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>user</value>
    <description>指定連線mysql的用戶名稱</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>aaa123</value>
    <description>連線資料庫的密碼</description>
  </property>
  ```
  
  * 編輯hive-env.sh: 
  ```js
  cp hive-env.sh.template hive-env.sh
  vim hive-env.sh
  
  #加上(注意版本)
  HADOOP_HOME=/home/${USER}/hadoop-2.10.1
  export HIVE_CONF_DIR=/home/${USER}/apache-hive-2.3.7-bin/conf

  ```
  * 新增剛剛設定的目錄: 
  ```js
  hadoop fs -mkdir -p  /user/hive/warehouse
  hadoop fs -mkdir -p /tmp/hive
  
  #修改權限
  hadoop fs -chmod 777 /user/hive/warehouse
  hadoop fs -chmod 777 /tmp/hive
  ```
  * 把[MySQL的JDBC檔](https://github.com/a13140120a/SQL_note/edit/master/README.md#002)放進hive底下的lib目錄(8.0.22版可)
  * MySQL 建立一個名為hive_metadata 的資料庫
  * [創建剛剛設定的USER](https://github.com/a13140120a/SQL_note/blob/master/README.md)
  ```js
  CREATE USER 'user'@'%' IDENTIFIED BY 'newpassword';
  #賦予權限
  GRANT ALL PRIVILEGES ON *.* TO 'user'@'%'
  FLUSH PRIVILEGES;
  
  #退出MySQL 並重啟
  sudo /etc/init.d/mysql restart
  ``` 
  * 測試(輸入hive)出現以下代表成功:
  ```js
  test@master:~$ hive
  SLF4J: Class path contains multiple SLF4J bindings.
  SLF4J: Found binding in [jar:file:/home/test/apache-hive-2.3.7-bin/lib/log4j-slf4j-impl-2.6.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
  SLF4J: Found binding in [jar:file:/home/test/hadoop-2.10.1/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
  SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
  SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]

  Logging initialized using configuration in jar:file:/home/test/apache-hive-2.3.7-bin/lib/hive-common-2.3.7.jar!/hive-log4j2.properties Async: true
  Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
  hive> 
  hive> create table test(id string);
  Loading class `com.mysql.jdbc.Driver'. This is deprecated. The new driver class is `com.mysql.cj.jdbc.Driver'. The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.
  OK
  Time taken: 4.575 seconds

  ```
  * 出現問題:
  ```js
  Exception in thread "main" java.lang.IllegalArgumentException: java.net.URISyntaxException: Relative path in absolute URI: ${system:java.io.tmpdir%7D/$%7Bsystem:user.name%7D
  ```
    * 解決: 在hive-site.xml 的\<configuration\> 下加上:
	```js
	<property>
         <name>system:java.io.tmpdir</name>
         <value>/tmp/hive/java</value>
        </property>
        <property>
          <name>system:user.name</name>
          <value>${user.name}</value>
        </property>
	```

* 操作Hive 資料表:
  * 建立內部資料表:ratings
  ```js
  create table ratings(userid INT,itemid INT, rating INT) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
  ```
  * 將本地端資料ratings.csv讀入ratings表格
  ```js
  
  ```



