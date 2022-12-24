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
connection = mysql+pymysql://glance:gl4nce-db@controller/glance

[keystone_authtoken]
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = gl4nce

[paste_deploy]
flavor = keystone

[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
...
```

### populate the glance database
```bash
su -s /bin/sh -c "glance-manage db_sync" glance
```

### restart glance
```bash
service glance-api restart
```
