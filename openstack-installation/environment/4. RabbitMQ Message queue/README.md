# controller node
### install package
```bash
apt install rabbitmq-server
```

### add openstack user
```bash
rabbitmqctl add_user openstack r4bb1t
```

### permit configuration for openstack user
```bash
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```
