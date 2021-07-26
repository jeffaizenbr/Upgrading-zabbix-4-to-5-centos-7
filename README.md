# Minimum zabbix 5 databases versions 

•	MySQL 5.5.62
•	MariaDB 10.0.37
•	PostgreSQL 9.2.24


# 1.0 – Stop zabbix server 

```bash
systemctl stop zabbix-server 
```

## Backup steps

# 1.1 - Create directories for backup files 

```bash
mkdir -p /opt/zabbix_backup/bin_files /opt/zabbix_backup/conf_files ; mkdir -p /opt/zabbix_backup/doc_files ; mkdir -p /opt/zabbix_backup/web_files /opt/zabbix_backup/db_files
```

# 1.2 - Backup zabbix binary, doc and conf files

```bash
cp -rp /etc/zabbix/zabbix_server.conf /opt/zabbix_backup/conf_files ; cp -rp /usr/sbin/zabbix_server /opt/zabbix_backup/bin_files ; cp -rp /usr/share/doc/zabbix-* /opt/zabbix_backup/doc_files ; cp -rp /etc/httpd/conf.d/zabbix.conf /opt/zabbix_backup/conf_files 2>/dev/null ; cp -rp /etc/apache2/conf-enabled/zabbix.conf /opt/zabbix_backup/conf_files 2>/dev/null ;  cp -rp /etc/zabbix/php-fpm.conf /opt/zabbix_backup/conf_files 2>/dev/null
```

# 1.3 - Backup zabbix web files (Frontend)

```bash
cp -rp /usr/share/zabbix/ /opt/zabbix_backup/web_files
```

# 1.4 - Backup Zabbix database

```bash
mysqldump -h localhost -u'root' -p'123' --single-transaction 'zabbix' | gzip > /opt/zabbix_backup/db_files/zabbix_backup.sql.gz
```

## Update Steps

# 2.0 - Adding zabbix release repos 

```bash
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/$(rpm -E %{rhel})/x86_64/zabbix-release-5.0-1.el$(rpm -E %{rhel}).noarch.rpm ; yum clean all ; yum repolist ; yum install zabbix-release -y
```
```bash
vim /etc/yum.repos.d/zabbix.repo
```
```bash
[zabbix-frontend]
name=Zabbix Official Repository frontend - $basearch
baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/frontend
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591
```
```bash
yum clean all ; yum repolist
```
# 2.1 - Install required packages

```bash
yum install scl-utils -y
```
```bash
yum install centos-release-scl -y
```
obs: to install no RHEL 7 enable centos-release-scp 
```bash
sudo yum-config-manager --enable rhel-server-rhscl-7-rpms
```
```bash
sudo yum-config-manager --add-repo=https://copr.fedoraproject.org/coprs/rhscl/centos-release-scl/repo/epel-7/rhscl-centos-release-scl-epel-7.repo
```
```bash
sudo yum install centos-release-scl
```
===========================
```bash
yum list rh-php7\*
```
```bash
yum install rh-php72* -y
```

# 2.2 - Update zabbix packages


```bash
yum update zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf -y
```
```bash
yum remove zabbix-web-4.* -y ; yum install zabbix-web-mysql-scl zabbix-apache-conf-scl -y
```
## Database issue steps

# 2.3 - Resolving doubleSQL issue on database 

```bash
cd /usr/share/doc/zabbix-server-mysql-XXXX
```
```bash
mysql -u zabbix -p zabbixdb < double.sql
```
OR secondary method
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

OR using PosgreSQL

```bash
/usr/pgsql-9.3/bin/psql -U db_zabbix -p 5432 -f /home/double.sql zabbix2 
```



## Configuration steps

# 2.4 – Add line in zabbix.conf.php

```bash
vim /etc/zabbix/web/zabbix.conf.php
```
```bash
$DB['DOUBLE_IEEE754'] = 'true';
```

# 2.5 - Add the line on PHP zabbix configuration file

```bash
vim /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf
```
```bash
php_value[date.timezone] = America/Sao_Paulo
```
## Start and test de zabbix uptade

# 2.6 - start zabbix server

```bash
systemctl enable rh-php72-php-fpm ; systemctl start rh-php72-php-fpm ; systemctl restart httpd
```
```bash
systemctl start zabbix-server ; tail -f /var/log/zabbix/zabbix_server.log
```
```bash
zabbix_server -V
```
```bash
cat /var/log/zabbix/zabbix_server.log | grep database
```







