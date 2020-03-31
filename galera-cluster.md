# galera cluster

galera cluster 為 mariadb 的群集 server，功能於 10.1 版後已內坎在 mariadb 中，只需進行配置即可啟動。

## 實際演練

以下說明共使用 3 台主機，作業系統 ubuntu 18.04，mariadb 10.4.12，IP 192.168.0.1 至 192.168.0.3

### 安裝所需套件

每台主機都需要安裝

```shell script
sudo apt-get install socat mariadb-backup
```

### 建立 SST 所需的帳號

每台主機都需要建立此帳號  
本範例的 [SST](https://mariadb.com/kb/en/introduction-to-state-snapshot-transfers-ssts/) 採用官方所推薦的 [mariabackup](https://mariadb.com/kb/en/mariabackup-sst-method/)，詳細內容可參閱連結說明。  
透過 [Unix Socket](https://mariadb.com/kb/en/authentication-plugin-unix-socket/) 建立不需密碼的帳號， SQL 語法如下：

確認帳號 mysql 是否已建立

```sql
select Host, User from mysql.user where User = 'mysql' and Host = 'localhost';
```

如果沒有，建立使用者 mysql

```sql
CREATE USER 'mysql'@'localhost' IDENTIFIED VIA unix_socket;
```

配置權限

```sql
GRANT RELOAD, PROCESS, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'mysql'@'localhost';
```

### my.cnf 配置

於 my.cnf 中找到段落 `[galera]`，添加以下配置

```.cnf
wsrep_on=ON # 啟動抄寫
wsrep_provider=/usr/lib/libgalera_smm.so  # wsrep library 的路徑
wsrep_cluster_address=gcomm://192.168.0.1,192.168.0.2,192.168.0.3  # cluster 中各 node 的 ip 位置
wsrep_sst_method=mariabackup  # SST 的運行模式
wsrep_sst_auth=mysql  # SST 的運行身份驗證，使用上面建立的
default_storage_engine=InnoDB # 預設的 storage engine
binlog_format=row # binary log format
innodb_autoinc_lock_mode=2  # auto_increment 欄位的運行模式
```

### 啟動 cluster

當要啟動 cluster 時，需先啟動擁有最新資料的 node，之後啟動的 node 將會抄寫此 node 的資料。執行指令啟動第一個 node：

```shell script
galera_new_cluster # 或 mysqld --wsrep-new-cluster
```

### 加入新 node

```shell script
systemctl start mariadb # 或 mysqld
```

### wsrep 狀態查詢

wsrep_cluster_size 為已連結的 node 數，其餘可查閱[此處](https://mariadb.com/kb/en/galera-cluster-status-variables/)

```sql
SHOW GLOBAL STATUS LIKE 'wsrep_%';
```

### 關閉單一 node

```shell script
systemctl stop mariadb
```

### 關閉全部 node

關閉時需記住那一個 node 是最後一個關閉的，因為 mariadb 會要求 cluster 中最後關閉的 node 需最先啟動。

### 再次啟動 cluster

再次啟動 cluster 的方式與啟動 cluster 相同，但需由上次關閉的最後一個 node 開始啟動，其它 node 的啟動也與加入新 node 相同。
