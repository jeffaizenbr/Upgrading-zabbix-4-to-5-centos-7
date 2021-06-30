Minimum zabbix 5 databases versions 

•	MySQL 5.5.62
•	MariaDB 10.0.37
•	PostgreSQL 9.2.24


1 – Stop zabbix server 

Systemctl stop zabbix server 

2 - Upgrade Zabbix Server and Frontend

rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/$(rpm -E %{rhel})/x86_64/zabbix-release-5.0-1.el$(rpm -E %{rhel}).noarch.rpm
yum clean all
yum zabbix-server-mysql zabbix-web-mysql


3- start zabbix server

systemctl start zabbix-server
zabbix_server -V
cat /var/log/zabbix/zabbix_server.log | grep database



4 – patch dbfix sql

wget https://git.zabbix.com/projects/ZBX/repos/zabbix/raw/database/mysql/double.sql
mysql -u'zabbix' -p'123' zabbix < double.sql


check the new table description

mysql -u'zabbix' -p'zabbixDBpass' zabbix -e "show create table history;"

5 – add line in zabbix.conf.php
Now we need to add the line bellow to “zabbix.conf.php” with the text editor (“nano /etc/zabbix/web/zabbix.conf.php“) to remove warning message from the frontend:

$DB['DOUBLE_IEEE754'] = 'true';
