# Shell Script

## 函式與引用

* 函式

  * 編輯 function.sh

        #!/usr/bin/bash
  
        test_fun(){
            echo "hello world"
            echo $1 $2
        }

        test_fun a b

  * 執行結果

        $ bash function.sh
        hello world
        a b

* 引用

  * 編輯 a.sh

        source /home/user/function.sh # 用 source 或 . 來引用

        test_fun a b

  * 執行結果

        $ bash function
        hello world
        a b

## 應用

* 透過 /etc/rc.d/init.d/rsyncd 開關及查詢 rsync daemon

  * 編輯 /etc/rc.d/init.d/rsyncd

        . /etc/init.d/functions

        exec="/bin/rsync" # 程式路徑
        prog="rsync" # 程式名稱
        config="/etc/rsyncd.conf" 程式配置檔
        lockfile="/var/lock/subsys/$prog" # 程式 lockfile
        OPTS="--daemon" # 程式參數

        if [[ $1 == "start" ]];then # 第1個參數 == start
          [ -x $exec ] || exit 5  # 程式可不可行
          [ -f $config ] || exit 6 # 有沒有配置檔
          echo -n "Starting $prog: " # 輸出開始執行程式，不換行
          daemon $exec $OPTS # 執行程式
          retval=$? # 取得回傳代碼
          echo # 輸出換行
          [ $retval -eq 0 ] && touch $lockfile # 執行成功就創建一個 lockfile

        elif [[ $1 == "stop" ]];then
          echo -n $"Stopping $prog:"
          killall $prog
          retval=$?
          [ $retval -eq 0 ] && rm -f $lockfile # 關閉成功就刪除 lockfile

        elif [[ $1 == "status" ]];then
          status $prog

        else
          echo $"Usage: $0 (start|stop|status)"
          exit 2
        fi
  * 執行

        # /etc/rc.d/init.d/rsyncd start

* 用 inotify 監聽並自動備份

  * 編輯 b.sh

        #!/usr/bin/bash
        prog="inotifywait"
        events="create,delete,modify,attrib"
        iopt="-mrq"

        lpath="/home/user/a/"

        rhost="192.168.56.115"
        vuser="vuser1"
        passwdfile="/etc/rsync.passwd"
        ropt="-az"
        modName="mod1"

        $prog $iopt --format "%w%f" -e $events $lpath | while read line
        do
          #echo $line
          #sleep 1
          rsync $ropt $line $vuser@$rhost::$modName --password-file=$passwdfile
        done

  * 執行 b.sh

        # ./b.sh
