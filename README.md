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
     hadoop_admin ALL=(ALL:ALL) ALL
     ```
  * 切換至剛剛建立好的 hadoop 管理帳號
    ```js
    su hadoop_admin
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
  * 下載 openjdk-8-jre-headless
  ```js
  sudo apt install openjdk-8-jre-headless
  ```
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
               <value>file:/usr/local/hadoop/tmp</value>
               <description>Abase for other temporary directories.</description>
          </property>
          <property>
               <name>fs.defaultFS</name>
               <value>hdfs://{主機}:9000</value>
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
     <value>file:/home/master/hadoop/tmp/dfs/name</value>
           </property>
           <property>
                   <name>dfs.datanode.data.dir</name>
     <value>file:/home/master/hadoop/tmp/dfs/data</value>
           </property>
           <property>
                   <name>dfs.replication</name>
                   <value>3</value>
           </property>
     </configuration>
     ```
  * 修改 `yarn-site.xml` 檔(可有可無)
  ```js
  <configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
  </configuration>
  ```
  * 修改 `yarn-site.xml`
  ```js
  <configuration>
    <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
    </property>
  </configuration>
  ```
  * 修改`slave` 
    加上所有slave的名稱(不包含master)  
  * 並於/etc/hosts 修改各機器ip及主機名稱  
  ```js
  127.0.0.1       localhost
  192.xxx.x.xxx   slave1
  192.xxx.x.xxx   slave2
  ```
  
























