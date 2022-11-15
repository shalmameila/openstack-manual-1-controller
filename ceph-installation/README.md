# Install Ceph 
- [server-world references](https://www.server-world.info/en/note?os=Ubuntu_20.04&p=ceph15&f=1)
- [official documentation](https://docs.ceph.com/en/latest/install/manual-deployment/)

1. install ceph on all nodes
```bash
sudo apt update; sudo apt install ceph -y
```

2. configurasi file `/etc/ceph/ceph.conf`
```bash
## generate unique ID (fsid) untuk cluster
uuidgen

## edit config file
nano /etc/ceph/ceph.conf

...
fsid = 883d49c7-5d63-481d-b5d5-991e9b576b25
mon initials member = compute-1
mon host = 10.9.1.11
cluster network = 10.9.2.0/24
public network = 10.9.2.0/24
...

```

- abis itu lanjut ngikutin tutor ini sampe ceph-mon [yang ini](https://docs.ceph.com/en/latest/install/manual-deployment/)
- setelah ceph-mon installed kita nyalain ceph-mgr [yang ini](https://docs.ceph.com/en/latest/mgr/administrator/#mgr-administrator-guide). Untuk part berikut :
> Place that key as file named keyring into mgr data path, which for a cluster “ceph” and mgr $name “foo” would be /var/lib/ceph/mgr/ceph-foo respective /var/lib/ceph/mgr/ceph-foo/keyring.

bisa pakai command berikut :

```bash
# bikin dircetory keyring
sudo -u ceph mkdir /var/lib/ceph/mgr/ceph-jennie-controller

# bikin file keyring
ceph auth get mgr.jennie-controller -o /var/lib/ceph/mgr/ceph-jennie-controller/keyring
```

- kalau dapet HEALTH_WARN kaya gini: 
```bash
root@jennie-controller:~# ceph -s
  cluster:
    id:     730a4b51-787f-46ec-9626-e1b644bcba5e
    health: HEALTH_WARN
            mon is allowing insecure global_id reclaim
            1 monitors have not enabled msgr2
            OSD count 0 < osd_pool_default_size 3

  services:
    mon: 1 daemons, quorum jennie-controller (age 33m)
    mgr: jennie-controller(active, since 7m)
    osd: 0 osds: 0 up, 0 in

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:
```
buat yang `insecure global_id reclaim` solve pakai [referensi ini](https://access.redhat.com/articles/6136242)
- ini untuk bikin OSD pakai bluestore [disini ya](https://docs.ceph.com/en/latest/install/manual-deployment/#bluestore). tapi kalau masih error we just need to create OSDs
- biar ceph-mon dan ceph-mgr nya auto start meski node nya di-reboot bisa bisa jalanin command :
```bash
systemctl enable ceph-mon.target
systemctl enable ceph-mgr@{node-name}
```
