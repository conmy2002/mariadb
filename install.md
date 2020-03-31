# mariadb server 安裝

各系統安裝方式可由[此處](https://downloads.mariadb.org/mariadb/repositories)查閱，亦可由[此處](https://downloads.mariadb.org/mariadb/10.4.12)查閱。

## 實際演練

使用 ubuntu 18.04，mariadb 10.4.12

### 安裝

```shell script
sudo apt-get install software-properties-common
sudo apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'
```

添加檔案 /etc/apt/sources.list.d/mariadb.list，附上以下內容

```.list
# MariaDB 10.4 repository list - created 2020-03-31 08:33 UTC
# http://downloads.mariadb.org/mariadb/repositories/
deb [arch=amd64,arm64,ppc64el] http://mirror.netcologne.de/mariadb/repo/10.4/ubuntu bionic main
deb-src http://mirror.netcologne.de/mariadb/repo/10.4/ubuntu bionic main
```

執行安裝指令

```shell script
sudo apt update
sudo apt install mariadb-server
```

啟動並確認狀態

```shell script
sudo systemctl start mariadb
sudo systemctl status mariadb
```
