# DNS

## DNS 正反解

* 利用 Domain Name 查詢 ip 稱為正解

* 利用 ip 查詢 Domain Name 稱為反解

## DNS 資源紀錄

* A：查 IPv4的位址（address）
* AAAA：查 IPv6的位址
* MX：查郵件伺服器（mail exchanger）
* NS：查名稱伺服器（name server）
* CNAME：查別名
* PTR：用 ip 反查 Domain Name
* HINFO：DNS Server 的系統資訊

## DNS 查詢流程

### Client to Server - Recursive

#### 1. 與本地伺服器進行查詢

* 若網域為自己管轄的即直接回覆（正式授權）

#### 2. 與本地伺服器進行查詢

* 若快取有資料則直接回覆（未經授權）
* 若無有資料則繼續執行

### Server to Server - Iterative

#### 3. 與根伺服器進行查詢

* 根伺服器回傳下一層伺服器的 ip
* 在美國查 xxx.com.tw，由於 .tw 屬於 TWNIC 管理，因此美國伺服器會先從根伺服器查詢，根伺服器再回應 TWNIC Server 的 ip

#### 4. 與下一層伺服器進行查詢

* 若有資料則直接回覆（未經授權）
* 若無資料則繼續詢問下層伺服器

## 用 bind9 架設 DNS Server

### bind9 主要配置文件 /etc/named.conf

* 格式

      options {
          //全域選項
          listen-on port 53 { any; };
          directory         "/var/named";
          ...
          allow-query     { any; };
          ...
      }
      zone　"zone name" {
          //區域選項
      }
      logging {
          //日誌文件
      }
      include：加載别的文件

* 編輯重點

      listen-on port 53 { any; }; // 可設定監聽的 ip
      allow-query     { any; }; // 可設定回應的網段

### include 檔 /etc/named.rfc1912.zones

* 格式

      zone　"a.com" IN {
          type master;
          file "a.com.zone";
          allow-update { none; };
      }

      zone　"56.168.192.in-addr.arpa" IN {
          type master;
          file "56.168.192.in-addr.arpa.zone";
          allow-update { none; };
      }

* 編輯重點

      file "a.com.zone"; // 相對路徑，位置為 named.conf -> options -> directory 設定的位置
      allow-update { none; }; // 是否可在開啟狀態下動態更新

#### DNS 正解資源紀錄 /var/named/a.com.zone

* 格式

      $TTL 600 ;10 minutes

      @ IN SOA   @ kevin823lin.gmail.com (
              2021031001 ;serial
              10800      ;refresh
              900        ;retry
              604800     ;expire
              86400      ;minimum
              )
      @           NS  dns1.a.com.
      @           NS  dns2.a.com.
      dns.com.    A   192.168.56.111
      dns1        A   192.168.56.111
      dns2        A   192.168.56.113
      www         A   192.168.56.111
      eshop       CNAME  www

* 編輯重點

      TTL // 緩存時間
      ; // 分號後面是註解
      @ // 管理的網域
      serial // 資源序列號
      refresh // Master/Slave 多久更新一次
      retry // Master/Slave 連線失敗後多久重試一次
      expire // slave 過了多久都無法和 master 取得連絡，就會刪除自己的這份 copy。
      minimum // 資料可以保存多久

#### DNS 反解資源紀錄 /var/named/56.168.192.in-addr.arpa.zone

* 格式

      $TTL 600 ;10 minutes

      @ IN SOA   @ kevin823lin.gmail.com (
              2021031001 ;serial
              10800      ;refresh
              900        ;retry
              604800     ;expire
              86400      ;minimum
              )
      56.168.192.in-addr.arpa.        IN NS dns1.a.com.
      56.168.192.in-addr.arpa.        IN NS dns2.a.com.
      
      111.56.168.192.in-addr.arpa.    IN PTR www.a.com.

#### named-checkzone

* 功能

  * 檢查區域配置檔格式有沒有問題

* 用法

      # named-checkzone a.com /var/named/a.com.zone

#### named-checkconf

* 功能

  * 檢查 bind9 配置檔格式有沒有問題

* 用法

      # named-checkconf // 沒問題就不會有輸出

#### nsupdate

* 功能

  * 動態更新 dns 資源紀錄

* 限制

  * DNS Server -> /etc/named.rfc1912.zones -> zone "XXX" -> allow-update 要填入下指令端的 ip

* 用法

  1. 新增

         # nsupdate
         > server 192.168.56.111
         > update add ftp.a.com. 60 A 192.168.56.115
         > send
         > quit

  2. 修改

         # nsupdate
         > server 192.168.56.111
         > update del ftp.a.com. A
         > update add ftp.a.com. 60 A 192.168.56.115
         > send
         > quit

  3. 刪除

         # nsupdate
         > server 192.168.56.111
         > update del ftp.a.com. A
         > send
         > quit

#### DNS Server Master/Slave 架構

* Master/Slave 時間要同步

* 編輯 Master /etc/named.conf

      options {
          ...
          allow-query     { any; };
          allow-transfer { 192.168.56.115; };
          also-notify { 192.168.56.115; };
          ...
      }

* 編輯 Slave /etc/named.conf

      options {
          ...
          allow-query     { any; };
          masterfile-format text;
          ...
          // dnssec-validation yes;
          dnssec-validation no;
          ...
      }

* 編輯 Slave /etc/sysconfig/named

      ...
      ENABLE_ZONE_WRITE=yes
      OPTIONS="-4"

* 編輯 Slave /etc/named.rfc1912.zones

      zone　"a.com" IN {
          type slave;
          master { 192.168.56.111; };
          file "slave/a.com.zone";
      }
* 編輯 Slave /etc/sysconfig/named

      ...
      OPTIONS="-4"
