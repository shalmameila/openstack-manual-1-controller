# neutron (controller)
### access database with root user
```bash
mysql -u root -p

# and then input database password = db-p4ss
```

### create the neutron database
```bash
CREATE DATABASE neutron;
```

### grant proper access to the neutron database
```bash
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'n3utr0n-db';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'n3utr0n-db';
```

### source the admin credentials to gain access to admin-only CLI commands
```bash
. admin-openrc
```

### create the neutron user
```bash
openstack user create --domain default --password-prompt neutron

# input password n3utr0n
```

### add the admin role to the neutron user
```bash
openstack role add --project service --user neutron admin
```

### create the neutron service entity
```bash
openstack service create --name neutron --description "OpenStack Networking" network
```

### create the Networking service API endpoints
```bash
openstack endpoint create --region RegionOne network public http://controller:9696
openstack endpoint create --region RegionOne network internal http://controller:9696
openstack endpoint create --region RegionOne network admin http://controller:9696
```

# provider network configuration
### install components
```bash
apt install neutron-server neutron-plugin-ml2 neutron-linuxbridge-agent neutron-dhcp-agent neutron-metadata-agent
```

### configure the server component
```bash
nano /etc/neutron/neutron.conf

...
[DEFAULT]
core_plugin = ml2
service_plugins =
transport_url = rabbit://openstack:r4bb1t@controller
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true

[database]
connection = mysql+pymysql://neutron:n3utr0n-db@controller/neutron

[keystone_authtoken]
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = n3utr0n

[nova]
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = n0va

[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
...
```

### configure the Modular Layer 2 (ML2) plug-in
```bash
nano /etc/neutron/plugins/ml2/ml2_conf.ini

...
[ml2]
type_drivers = flat,vlan
tenant_network_types =
mechanism_drivers = linuxbridge
extension_drivers = port_security

[ml2_type_flat]
flat_networks = provider

[securitygroup]
enable_ipset = true
...
```

### configure the Linux bridge agent
```bash
nano /etc/neutron/plugins/ml2/linuxbridge_agent.ini

...
[linux_bridge]
physical_interface_mappings = provider1:ens5,provider2:ens6

[vxlan]
enable_vxlan = false

[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
...
```

### ensure your Linux operating system kernel supports network bridge filters by verifying all the following sysctl values are set to 1
```bash
sysctl net.bridge.bridge-nf-call-iptables
sysctl net.bridge.bridge-nf-call-ip6tables
```

### configure DHCP agent
```bash
nano /etc/neutron/dhcp_agent.ini

...
[DEFAULT]
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
...
```

# configure self-service network
### install components
```bash
apt install neutron-server neutron-plugin-ml2 neutron-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent neutron-metadata-agent
```

### configure the server component
```bash
nano /etc/neutron/neutron.conf

...
[database]
connection = mysql+pymysql://neutron:n3utr0n-db@controller/neutron

[DEFAULT]
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = true
transport_url = rabbit://openstack:r4bb1t@controller
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true

[keystone_authtoken]
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = n3utr0n

[nova]
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = n0va

[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
...
```

### configure the Modular Layer 2 (ML2) plug-in
```bash
nano /etc/neutron/plugins/ml2/ml2_conf.ini

...
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security

[ml2_type_flat]
flat_networks = provider

[ml2_type_vxlan]
vni_ranges = 1:1000

[securitygroup]
enable_ipset = true
...
```

### configure the Linux bridge agent
```bash
nano /etc/neutron/plugins/ml2/linuxbridge_agent.ini

...
[linux_bridge]
physical_interface_mappings = provider1:ens5,provider2:ens6

[vxlan]
enable_vxlan = true
local_ip = 10.9.1.11
l2_population = true

[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
...
```

### ensure your Linux operating system kernel supports network bridge filters by verifying all the following sysctl values are set to 1
```bash
sysctl net.bridge.bridge-nf-call-iptables
sysctl net.bridge.bridge-nf-call-ip6tables
```

### configure the layer-3 agent
```bash
nano /etc/neutron/l3_agent.ini

...
[DEFAULT]
interface_driver = linuxbridge
```

### configure the DHCP agent
```bash
nano /etc/neutron/dhcp_agent.ini

...
[DEFAULT]
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
```

### configure the metadata agent
```bash
nano /etc/neutron/metadata_agent.ini

...
[DEFAULT]
# ...
nova_metadata_host = controller
metadata_proxy_shared_secret = m3tadt4ta-rahasia
...
```

### populate the database
```bash
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

### restart the Compute API service
```bash
service nova-api restart
```

### restart the Networking services
- For both networking options
```bash
service neutron-server restart
service neutron-linuxbridge-agent restart
service neutron-dhcp-agent restart
service neutron-metadata-agent restart
```

- for networking option 2, also restart the layer-3 service
```bash
service neutron-l3-agent restart
```








































