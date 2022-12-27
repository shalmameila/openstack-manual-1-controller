# placement
all command run in controller

### access database with **root** user
```bash
mysql -u root -p

# and then input database password = db-p4ss
```

### create **placement** database
```bash
CREATE DATABASE placement;
```

### grant proper access to the database
```bash
GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' IDENTIFIED BY 'pl4cem3nt-db';
GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' IDENTIFIED BY 'pl4cem3nt-db';
exit
```

### source the admin credentials to gain access to admin-only CLI commands
```bash
. admin-openrc
```

### create **placement** user
```bash
openstack user create --domain default --password-prompt placement
# input password pl4cem3nt
```

### add the **placement** user to the **service** project with the **admin** role
```bash
openstack role add --project service --user placement admin
```

### create the Placement API entry in the service catalog
```bash
openstack service create --name placement --description "Placement API" placement
```

### create the Placement API service endpoints
```bash
openstack endpoint create --region RegionOne placement public http://controller:8778
openstack endpoint create --region RegionOne placement internal http://controller:8778
openstack endpoint create --region RegionOne placement admin http://controller:8778
```

### install **placement** package
```bash
apt install placement-api
```

### configure
```bash
nano /etc/placement/placement.conf

...
[placement_database]
connection = mysql+pymysql://placement:pl4ceme3nt-db@controller/placement

[api]
auth_strategy = keystone

[keystone_authtoken]
auth_url = http://controller:5000/v3
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = placement
password = pl4cem3nt
...
```

### populate the placement database
```bash
su -s /bin/sh -c "placement-manage db sync" placement
```

### restart apache2 server
```bash
service apache2 restart
```

## verify
### perform status checks to make sure everything is in order
```bash
placement-status upgrade check
```

### install the osc-placement plugin
```bash
pip install osc-placement
```

### list available resource classes and traits
```bash
openstack --os-placement-api-version 1.2 resource class list --sort-column name
openstack --os-placement-api-version 1.6 trait list --sort-column name
```
