# Minimum zabbix 5 databases versions 

•	MySQL 5.5.62
•	MariaDB 10.0.37
•	PostgreSQL 9.2.24


# 1 – Stop zabbix server 

```bash
Systemctl stop zabbix server 
```

# 3 - Create directories for backup files 

```bash
mkdir -p /opt/zabbix_backup/bin_files /opt/zabbix_backup/conf_files ; mkdir -p /opt/zabbix_backup/doc_files ; mkdir -p /opt/zabbix_backup/web_files /opt/zabbix_backup/db_files
```

# 4 - Backup zabbix binary, doc and conf files

```bash
cp -rp /etc/zabbix/zabbix_server.conf /opt/zabbix_backup/conf_files ; cp -rp /usr/sbin/zabbix_server /opt/zabbix_backup/bin_files ; cp -rp /usr/share/doc/zabbix-* /opt/zabbix_backup/doc_files ; cp -rp /etc/httpd/conf.d/zabbix.conf /opt/zabbix_backup/conf_files 2>/dev/null ; cp -rp /etc/apache2/conf-enabled/zabbix.conf /opt/zabbix_backup/conf_files 2>/dev/null ;  cp -rp /etc/zabbix/php-fpm.conf /opt/zabbix_backup/conf_files 2>/dev/null
```

# 5 - Backup zabbix web files (Frontend)

```bash
cp -rp /usr/share/zabbix/ /opt/zabbix_backup/web_files
```

# 6 - Backup Zabbix database

```bash
mysqldump -h localhost -u'root' -p'123' --single-transaction 'zabbix' | gzip > /opt/zabbix_backup/db_files/zabbix_backup.sql.gz
```

# 7 - Upgrade Zabbix Server and Frontend

```bash
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/$(rpm -E %{rhel})/x86_64/zabbix-release-5.0-1.el$(rpm -E %{rhel}).noarch.rpm ; yum clean all ; yum repolist ; yum install zabbix-release -y
```
```bash
yum clean all
```
```bash
yum update zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf
```
```bash
yum install scl-utils -y
```
```bash
yum install centos-release-scl -y
```
```bash
yum list rh-php7\*
```
```bash
yum install php-XXXX
```
```bash
yum clean all ; yum repolist all
```
```bash
yum remove zabbix-web-4.* -y ; yum install zabbix-web-mysql-scl zabbix-apache-conf-scl
```

# 8 - start zabbix server

```bash
systemctl start zabbix-server
```
```bash
zabbix_server -V
```
```bash
cat /var/log/zabbix/zabbix_server.log | grep database
```


# 9 – patch dbfix sql

```bash
wget https://git.zabbix.com/projects/ZBX/repos/zabbix/raw/database/mysql/double.sql
```
```bash
mysql -u'zabbix' -p'123' zabbix < double.sql
```

check the new table description
```bash
mysql -u'zabbix' -p'123' zabbix -e "show create table history;"
```
# 10 – add line in zabbix.conf.php
Now we need to add the line bellow to “zabbix.conf.php” with the text editor (“nano /etc/zabbix/web/zabbix.conf.php“) to remove warning message from the frontend:
```bash
$DB['DOUBLE_IEEE754'] = 'true';
```
