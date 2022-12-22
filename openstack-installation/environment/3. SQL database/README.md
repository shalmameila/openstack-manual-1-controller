# controller node
### install package
```bash
apt install mariadb-server python3-pymysql
```

### create and edit the `/etc/mysql/mariadb.conf.d/99-openstack.cnf` file
```bash
nano /etc/mysql/mariadb.conf.d/99-openstack.cnf

...
[mysqld]
bind-address = 10.9.1.11

default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
...
```

### restart service
```bash
service mysql restart
```

### secure database
```bash
mysql_secure_installation
```
