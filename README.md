# hadoop
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
    

    export JAVA_HOME=/home/spark/jdk1.8.0_251   #加上這兩行
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
  * 進到 `hadoop-2.10.0/etc/hadoop/` 編輯 `core-site.xml`
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
  * <value>hdfs://主機名稱(@右邊):9000</value>  (設定HDFS登入位置，port號為9000)
  * <value>file:/usr/local/hadoop/tmp</value>  (設定保存臨時文件位置，預設是/tmp/hadoop-hadoop)
  * 進到 `hadoop-2.10.0/etc/hadoop/` 編輯 `hdfs-site.xml`
  ```js
  cd ~/hadoop-2.10.0/etc/hadoop/
  vim hdfs-site.xml
  ```
  * 一樣修改
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




























