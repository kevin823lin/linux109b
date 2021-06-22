# rsync

## 簡介

rsync 是一款可以輕鬆同步本地與異地電腦資料的軟體

## 前置步驟

ssh 免密登入

    # ssh-keygen
    # ssh-copy-id

## 基本用法

* `-v`：verbose 模式，輸出比較詳細的訊息
* `-r`：遞迴（recursive）備份所有子目錄下的目錄與檔案
* `-a`：封裝備份模式，相當於 -rlptgoD，遞迴備份所有子目錄下的目錄與檔案，保留連結檔、檔案的擁有者、群組、權限以及時間戳記
* `-z`：啟用壓縮
* `-h`：將數字以比較容易閱讀的格式輸出
* `--delete`：會同步刪除的檔案
* `--bwlimit=`；限制網路限制網路頻寬
* `-e 'ssh -p 2222'`：自訂 ssh 連接埠
* `progress`：顯示傳輸進度
* `--include '*.c'`：包含所有 .c 的檔案
* `--exclude '*.txt'`：排除所有 .txt 的檔案
* `--min-size=`：備份的檔案大小下限
* `--max-size=`：備份的檔案大小上限
* `--dry-run`：模擬執行指令，不會真的修改

## 基本指令

    # rsync 參數 來源檔案 目的檔案

### 本地備份範例

* push

  * 備份資料夾裡的檔案

        # rsync -avh /test/ /backup

  * 備份資料夾及裡的檔案

        # rsync -avh /test /backup

* pull

  * 還原資料夾裡的檔案

        # rsync -avh /backup/ /test

### 異地備份範例

* push

  * 備份資料夾裡的檔案

        # rsync -avzh /test/ root@192.168.56.115:/backup

  * 備份資料夾及裡的檔案

        # rsync -avh /test root@192.168.56.115:/backup

* pull

  * 還原資料夾裡的檔案

        # rsync -avh root@192.168.56.115:/backup/ /test

## 備份伺服器

* rsync client - 備份伺服器 /etc/rsyncd.conf

      uid = root # 執行身分
      gid = root # 執行組別
      pid file = /var/run/rsync.pid # 存放 pid 編號的路徑
      lock file = /var/run/rsync.lock # 互斥鎖
      log file = /var/run/rsync.log # log 檔

      [mod1] # 模組名稱
      path = /backup/centos
      user chroot = no
      max connection = 100
      read only = false
      hosts allow = 192.168.56.0/24
      auth users = vuser1 # 允許的虛擬使用者
      secrets file = /backup/rsync.passwd # 密碼檔

* rsync client - 備份伺服器 /backup/rsync.passwd

      vuser1:passwd

* rsync client - 備份伺服器執行，預設873埠

      # rsync --daemon

* rsync server - 備份客戶端

      # rsync -avzh /test/ vuser1@192.168.56.115::mod1

---

## 參考資料

1. G. T. Wang：<https://blog.gtwang.org/linux/rsync-local-remote-file-synchronization-commands/>

2. 51CTO 博客 - wzlinux：<https://blog.51cto.com/wzlinux/2045659/>
