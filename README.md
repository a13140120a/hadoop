# hadoop

* 修改本機hostname:
  ```js
  vi /etc/hostname
  ```
* 建立hadoop使用者以及群組(每台都要)  
  * 建立群組  
   ```js
   sudo addgroup hadoop_group  
   ```
   * 建立 Hadoop 專用帳戶
   ```js
   sudo adduser --ingroup hadoop_group [帳號]
   ```
   * 將 hadoop_admin 帳戶加入 sudo 權限
     ```js
     sudo vi /etc/sudoers
     ```
     下方新增:
     ```js
     [帳號] ALL=(ALL:ALL) ALL
     ```
  * 切換至剛剛建立好的 hadoop 管理帳號
    ```js
    su [帳號]
    ```
    
* 建立金鑰:
   ```js
   ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
   cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys   
   ```    
  * scp authorized_keys到各個slave內
  * 如果沒有.ssh資料夾，就先`ssh localhost` 再登出就會出現了
  
  
* 安裝java
  * 下載jdk1.8
  * 解壓縮
  ```js
  tar zxvf jdk-8u251-linux-x64.tar.gz
  ```
  * 把資料夾移動到home目錄
  ```js
  mv jdk1.8.0_251 ~/
  ```
  * 設定 `~/.bashrc` 檔
    ```js
    #找到這幾行
    
    # enable programmable completion features (you don't need to enable
    # this, if it's already enabled in /etc/bash.bashrc and /etc/profile
    # sources /etc/bash.bashrc).
    if ! shopt -oq posix; then
      if [ -f /usr/share/bash-completion/bash_completion ]; then
        . /usr/share/bash-completion/bash_completion
      elif [ -f /etc/bash_completion ]; then
        . /etc/bash_completion
      fi
    fi
    

    export JAVA_HOME=/home/{username}/jdk1.8.0_251   #加上這兩行 (注意路徑)
    export PATH=$JAVA_HOME/bin:$PATH            #加上這兩行
    ```
  * 執行~/.bashrc檔 `source ~/.bashrc`
  * 測試 
  ```js 
  java -version
  ```  
  
* 安裝hadoop
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
               <value>/home/[帳號]/tmp</value>
               <description>Abase for other temporary directories.</description>
          </property>
          <property>
               <name>fs.defaultFS</name>
               <value>hdfs://{主機}:9000</value>  #注意路徑
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
                   <name>dfs.namenode.secondary.http-address</name>  #可有可無
                   <value>master:50090</value>
           </property>
           <property>
                   <name>dfs.namenode.name.dir</name>
                   <value>/home/[帳號]/hdfs/namenode</value>   #注意路徑
           </property>
           <property>
                   <name>dfs.datanode.data.dir</name>
                   <value>/home/[帳號]/hdfs/datanode</value>
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
    export JAVA_HOME=/home/[帳號]/jdk1.8.0_251  #注意路徑
    export PATH=$JAVA_HOME/bin:$PATH

    export HADOOP_HOME=/home/[帳號]/hadoop-2.10.0
    export PATH=$HADOOP_HOME/bin:$PATH

    export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop  
    ```
  * 修改`~/.bashrc`檔，底下添加:(記得source)
    ```js
    export HADOOP_HOME=/home/[帳號]/hadoop-2.10.0   #注意路徑
    export PATH=$HADOOP_HOME/bin:$PATH

    export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
    ```
  * 建立需要的資料夾(hadoop/tmp 還有hdfs/namenode、datanode)
  ```js
  cd ~
  mkdir tmp
  mkdir hdfs

  cd hdfs
  mkdir namenode
  mkdir datanode
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
* 安裝yarn:(可有可無)
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
#參考資料:[https://medium.com/@sleo1104/hadoop-3-2-0-%E5%AE%89%E8%A3%9D%E6%95%99%E5%AD%B8%E8%88%87%E4%BB%8B%E7%B4%B9-22aa183be33a](https://medium.com/@sleo1104/hadoop-3-2-0-%E5%AE%89%E8%A3%9D%E6%95%99%E5%AD%B8%E8%88%87%E4%BB%8B%E7%B4%B9-22aa183be33a)

* 出現異常:
```js
hadoop@master:~/sparkk/spark101/wordcount/data$ hadoop fs -mkdir hdfs://master/user/
mkdir: Call From master/192.168.1.69 to master:8020 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused
```
  解決方法:
  core-site.xml檔的9000port改成8020port















