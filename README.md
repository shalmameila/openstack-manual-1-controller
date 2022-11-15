## Bikin VM di GCP
```bash
  gcloud compute instances create backyard-lab --project=the-byway-366616 --zone=us-central1-a --machine-type=n2-standard-8 --network-interface=address=34.72.70.87,network-tier=PREMIUM,subnet=default --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=568476732011-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=http-server,https-server --create-disk=auto-delete=yes,boot=yes,device-name=backyard-lab,image=projects/ubuntu-os-cloud/global/images/ubuntu-2004-focal-v20221018,mode=rw,size=550,type=projects/the-byway-366616/zones/us-central1-a/diskTypes/pd-standard --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any --enable-nested-virtualization
```
> Note, jangan lupa flavor-nya pakai n series biar bisa nested virtualization

## Bikin VM di dalam VM GCP pakai KVM
### Create network (Create file xml)
1. Public network, Admin network, Internal network, Ceph public network, Ceph cluster network, Self-service network
```bash
for i in {1..6}; do echo "<network>
  <name>net-10.9.$i</name>
  <forward mode='route'/>
  <ip address='10.9.$i.1' netmask='255.255.255.0'>
    <dhcp>
    </dhcp>
  </ip>
</network>" >> net-10.9.$i.xml; done
```

2. Provider network
```bash
for i in {9..10}; do echo "<network>
  <name>net-172.9.$i</name>
  <forward mode='route'/>
  <ip address='172.9.$i.1' netmask='255.255.255.0'>
    <dhcp>
    </dhcp>
  </ip>
</network>" >> net-172.9.$i.xml
```

### Define, start dan autostart network 
```bash
for i in {1..6}; do virsh net-define net-10.9.$i.xml; virsh net-start net-10.9.$i; virsh net-autostart net-10.9.$i; done
for i in {9..10}; do virsh net-define net-172.9.$i.xml; virsh net-start net-172.9.$i; virsh net-autostart net-172.9.$i; done
```

### Install terraform
**[REFERENSI INSTALASI](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)**

```bash
wget https://releases.hashicorp.com/terraform/0.13.7/terraform_0.13.7_linux_amd64.zip
unzip terraform_0.13.7_linux_amd64.zip
```

### Create KVM storage pool
[REF-1](https://book.btech.id/books/setup-server/page/setup-lab) 

[REF-2](https://book.btech.id/books/btech/page/setup-kvm-host-lab-btech)

```bash 
mkdir /data/isos
mkdir /data/vms
virsh pool-define-as vms dir - - - - "/data/vms"
virsh pool-define-as isos dir - - - - "/data/isos"
virsh pool-autostart vms
virsh pool-autostart isos
virsh pool-start vms
virsh pool-start isos

# Lalu copy .img ke dir `/data/isos`
# Update list isos
virsh pool-refresh isos
virsh vol-list isos
```

### Create VM on KVM
[REF TFGEN MAS ARYA](https://go.btech.id/arya/tfgen)

Create environment file
```bash
[LAB]
PUBKEY1: {diambil dari .ssh/id_rsa.pub}

[VM1]
NAME: jennie-controller
OS: ubuntu-focal.img
NESTED: n
VCPUS: 4
MEMORY: 8G
DISK1: 100G
IFACE_NETWORK1: 10.9.1
IFACE_IP1: 10.9.1.11
IFACE_NETWORK2: 10.9.2
IFACE_IP2: 10.9.2.11
IFACE_NETWORK3: 172.9.9.0
IFACE_IP3: none
IFACE_NETWORK4:  172.9.10.0
IFACE_IP4: none
CONSOLE: vnc

[VM2]
NAME: jennie-compute-1
OS: ubuntu-focal.img
NESTED: n
VCPUS: 4
MEMORY: 8G
DISK1: 50G
DISK2: 50G
DISK3: 50G
IFACE_NETWORK1: 10.9.1.0
IFACE_IP1: 10.9.1.12
IFACE_NETWORK2: 10.9.2
IFACE_IP2: 10.9.2.12
IFACE_NETWORK3: 172.9.9.0
IFACE_IP3: none
IFACE_NETWORK4:  172.9.10.0
IFACE_IP4: none
CONSOLE: vnc

[VM3]
NAME: jennie-compute-2
OS: ubuntu-focal.img
NESTED: n
VCPUS: 4
MEMORY: 8G
DISK1: 50G
DISK2: 50G
DISK3: 50G
IFACE_NETWORK1: 10.1.1.0
IFACE_IP1: 10.1.1.13
IFACE_NETWORK2: 10.9.2
IFACE_IP2: 10.9.2.13
IFACE_NETWORK3: 172.9.9.0
IFACE_IP3: none
IFACE_NETWORK4:  172.9.10.0
IFACE_IP4: none
CONSOLE: vnc

[VM4]
NAME: jennie-compute-3
OS: ubuntu-focal.img
NESTED: n
VCPUS: 4
MEMORY: 8G
DISK1: 50G
DISK2: 50G
DISK3: 50G
IFACE_NETWORK1: 10.1.1.0
IFACE_IP1: 10.1.1.14
IFACE_NETWORK2: 10.9.2
IFACE_IP2: 10.9.2.14
IFACE_NETWORK3: 172.9.9.0
IFACE_IP3: none
IFACE_NETWORK4: 172.9.10.0
IFACE_IP4: none
CONSOLE: vnc
```
```bash
# Initialize a Terraform working directory
terraform init

# Builds or changes infrastructure
terraform apply

# autostart all VM
for i in `virsh list --all --name`; do virsh autostart $i; done
```
