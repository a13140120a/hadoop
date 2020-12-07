# hadoop

* ## 目錄:
  * ## [1. HDFS安裝及基本指令](#001)  
  * ## [2. YARN](#002)
  * ## [3.python撰寫mapreduce範例](#003)
  * ## [3. Hue](#003)
  * ## [4. Oozie](#004)
  * ## [3.資料擷取模組Sqoop,Flume](#005)
  * ## [4.資料分析模組Pig,Hive](#006)

<h2 id="001">1. HDFS安裝及基本指令</h2>  
  
* 由Name Node 與 Data Node組成:  
  * Name Node 由 fsimage 與 edits log 組成:  
    - fsimage: HDFS 當時的 snapshot, 檔案較為龐大，讀寫較慢  
    - edits log: HDFS 自 snapshot 後的變更，edit log 檔案比較小  
    
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
  tar zxvf hadoop-2.10.0.tar.gz
  ```
  * 移動到 home目錄
  ```js
  mv hadoop-2.10.0 ~/
  ```
  * 修改 `core-site.xml` 檔
  ```js
  cd ~
  cd hadoop-2.10.0/etc/hadoop/
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
               <value>hdfs://master:9000</value>  #注意路徑
          </property>
     </configuration>
     ```

   * 修改 `hdfs-site.xml` 檔
     ```js
     cd ~/hadoop-2.10.0/etc/hadoop/
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

    export HADOOP_HOME=/home/${USER}/hadoop-2.10.0  #注意版本
    export PATH=$HADOOP_HOME/bin:$PATH

    export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop  
    ```
  * 修改`~/.bashrc`檔，底下添加:(記得source)
    ```js
    export HADOOP_HOME=/home/${USER}/hadoop-2.10.0  #注意版本
    export PATH=$HADOOP_HOME/bin:$PATH

    export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
    ```  
  * scp 整個hadoop資料夾、jave資料夾、`~/.bashrc` 檔到各個slave:(記得每台都要source)
  ```js
  scp -r hadoop-2.10.0 xxx@xxx.xxx.x.xxx:~
  ```
  * 格式化hadoop:
  ```js
  hadoop namenode -format
  ```
  * 啟動hadoop:
  ```js
  cd ~
  cd hadoop-2.10.0/sbin
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
      * Node Manager 會依任務需求把 Container 等需求環境準備好  
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
echo 隨便一段英文句子 |python mapper.py|sort -k 1|python3 reducer.py
```











