# controller node
### install package
```bash
apt install chrony
```

### configure
```bash
nano /etc/chrony/chrony.conf

...
server 10.9.1.11 iburst
allow 10.9.1.0/24
...

```

### restart NTP server
```bash
service chrony restart
```

# compute node
### install package
```bash
apt install chrony
```

### configure
```bash 
nano /etc/chrony/chrony.conf 

...
server jennie-controller iburst
...
```
Comment out the `pool 2.debian.pool.ntp.org offline iburst` line.

### restart NTP server
```bash
service chrony restart
```


