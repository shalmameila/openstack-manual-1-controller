# controller
### install package
```bash
apt install memcached python3-memcache
```

### configure
```bash
nano /etc/memcached.conf

...
-l 10.9.1.11
...
```
> note : Change the existing line that had `-l 127.0.0.1`.



### restart service
```bash
service memcached restart
```


