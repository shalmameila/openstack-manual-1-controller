# keystone (identity service)
all command run in **controller**

### access database with **root** user
```bash
mysql -u root -p

# and then input database password = db-p4ss
```

### create **keystone** database
```bash
CREATE DATABASE keystone;
```

### grant propper access to the database
```bash
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'keyst0ne-db';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'keyst0ne-db';
exit
```
> `keyst0ne-db` is my KEYSTONE_DBPASS

### install keystone package
```bash
apt install keystone
```

### configure
```bash
nano /etc/keystone/keystone.conf

...
[database]
connection = mysql+pymysql://keystone:keyst0ne-db@controller/keystone

[token]
provider = fernet
...
```

### populate keystone database
```bash
su -s /bin/sh -c "keystone-manage db_sync" keystone
```

### initialize Fernet key repositories
```bash
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

### bootstrap the Identity service
```bash
keystone-manage bootstrap --bootstrap-password 4dm1n-pw \
  --bootstrap-admin-url http://controller:5000/v3/ \
  --bootstrap-internal-url http://controller:5000/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne
```
> `4dmin-pw` is my ADMIN_PASS

### configure apache HTTP server
```bash
nano /etc/apache2/apache2.conf

...
ServerName controller
...
```

### restart service apache 
```bash
service apache2 restart
```

### configure admin-openrc
```bash
nano ~/admin-openrc

...
export OS_USERNAME=admin
export OS_PASSWORD=4dm1n-pw
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
...
```

### create a domain, projects, users, and roles
```bash
openstack domain create --description "An Example Domain" example
openstack project create --domain default --description "Service Project" service
openstack project create --domain default --description "Demo Project" myproject
openstack user create --domain default --password-prompt myuser
# password myus3r
openstack role create myrole
openstack role add --project myproject --user myuser myrole
```

### verify
```bash
unset OS_AUTH_URL OS_PASSWORD
openstack --os-auth-url http://controller:5000/v3 --os-project-domain-name Default --os-user-domain-name Default --os-project-name admin --os-username admin token issue
```
> input password **ADMIN_PASS** = 4dm1n-pw

```bash
openstack --os-auth-url http://controller:5000/v3 --os-project-domain-name Default --os-user-domain-name Default --os-project-name myproject --os-username myuser token issue
```
> input password myus3r

### request authentication token
```bash
openstack token issue
```
