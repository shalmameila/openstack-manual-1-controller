# nova (controller node)

### access database with root user
```bash
mysql -u root -p
```

### create the `nova_api`, `nova`, and `nova_cell0` databases
```bash
CREATE DATABASE nova_api;
CREATE DATABASE nova;
CREATE DATABASE nova_cell0;
```

### grant proper access to the databases
```bash
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'n0va-db';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'n0va-db';

GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'n0va-db';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'n0va-db';

GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY 'n0va-db';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'n0va-db';
```

### source the admin credentials to gain access to admin-only CLI commands
```bash
. admin-openrc
```

### create the nova user
```bash
openstack user create --domain default --password-prompt nova
```

### add the admin role to the nova user
```bash
openstack role add --project service --user nova admin
# password n0va
```

### create the nova service entity
```bash
openstack service create --name nova --description "OpenStack Compute" compute
```

### create the Compute API service endpoints
```bash
openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
```

### install package
```bash
apt install nova-api nova-conductor nova-novncproxy nova-scheduler
```

### configure
> Due to a packaging bug, remove the log_dir option from the [DEFAULT] section.


```bash
nano /etc/nova/nova.conf

...
[DEFAULT]
transport_url = rabbit://openstack:RABBIT_PASS@controller:5672/
my_ip = 10.9.1.11

[api_database]
connection = mysql+pymysql://nova:n0va-db@controller/nova_api

[database]
connection = mysql+pymysql://nova:n0va-db@controller/nova

[api]
auth_strategy = keystone

[keystone_authtoken]
www_authenticate_uri = http://controller:5000/
auth_url = http://controller:5000/
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = n0va

[neutron]
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = NEUTRON_PASS
service_metadata_proxy = tru
metadata_proxy_shared_secret = m3tadt4ta-rahasia

[vnc]
enabled = true
server_listen = $my_ip
server_proxyclient_address = $my_ip

[glance]
api_servers = http://controller:9292

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[placement]
region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = pl4cem3nt
```

### populate the nova-api database
```bash
su -s /bin/sh -c "nova-manage api_db sync" nova
```

### register the cell0 database
```bash
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
```

### create the cell1 cell
```bash
su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
```

### populate the nova database
```bash
su -s /bin/sh -c "nova-manage db sync" nova
```

### verify nova cell0 and cell1 are registered correctly
```bash
su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova
```
the result should be like this :
```bash
+-------+--------------------------------------+----------------------------------------------------+--------------------------------------------------------------+----------+
|  Name |                 UUID                 |                   Transport URL                    |                     Database Connection                      | Disabled |
+-------+--------------------------------------+----------------------------------------------------+--------------------------------------------------------------+----------+
| cell0 | 00000000-0000-0000-0000-000000000000 |                       none:/                       | mysql+pymysql://nova:****@controller/nova_cell0?charset=utf8 |  False   |
| cell1 | f690f4fd-2bc5-4f15-8145-db561a7b9d3d | rabbit://openstack:****@controller:5672/nova_cell1 | mysql+pymysql://nova:****@controller/nova_cell1?charset=utf8 |  False   |
+-------+--------------------------------------+----------------------------------------------------+--------------------------------------------------------------+----------+
```

### restart the Compute services
```bash
service nova-api restart
service nova-scheduler restart
service nova-conductor restart
service nova-novncproxy restart
```








