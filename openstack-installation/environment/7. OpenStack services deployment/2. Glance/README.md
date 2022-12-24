# glance (image service)
all command run in controller

### ### access database with **root** user
```bash
mysql -u root -p

# and then input database password = db-p4ss
```

### create **glance** database
```bash
CREATE DATABASE glance;
```

### grant propper access to the database
```bash
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'gl4nce-db';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'gl4nce-db';
exit
```

### source the admin credentials to gain access to admin-only CLI commands
```bash
. admin-openrc
```

### create **glance** user
```bash
openstack user create --domain default --password-prompt glance
# input password gl4nce
```

### add **admin** role to the **glance** user and **service** project
```bash
openstack role add --project service --user glance admin
```

### create the glance service entity
```bash
openstack service create --name glance --description "OpenStack Image" image
```

### create the Image service API endpoints
```bash
openstack endpoint create --region RegionOne image public http://controller:9292
openstack endpoint create --region RegionOne image internal http://controller:9292
openstack endpoint create --region RegionOne image admin http://controller:9292
```

### install **glance** package
```bash
apt install glance
```

### configure
```bash
nano /etc/glance/glance-api.conf

...
[database]
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance





