# hadoop
* 安裝
  * 下載jdk1.8，並且移動到home目錄
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
