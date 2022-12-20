# Install openstack (only 1 controller and 3 computes)
actually i configure following all of documentation, with a few reference from server-world.

- Environment configuration (https://docs.openstack.org/install-guide/environment.html)
- Openstack services install and configuration for USSURI (https://docs.openstack.org/install-guide/openstack-services.html#minimal-deployment-for-ussuri)

while installing nova on controller, there's a command to register cell0 database. it display like this :
![image](https://user-images.githubusercontent.com/62281604/208558413-f2b0daee-80a4-4c35-a7db-007a07e5bb5a.png)
but it doesn't work, i modified to :
```bash
su -s /bin/sh -c "nova-manage --debug cell_v2 map_cell0 --database_connection mysql+pymysql://nova:obiwgans@controller/nova_cell0" nova
```
and it works!!




