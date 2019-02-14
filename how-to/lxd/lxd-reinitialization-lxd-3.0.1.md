# lxd-reinitialization.md

## Resources

### lxd module

* https://github.com/lxc/lxd/blob/master/doc/rest-api.md#post-1
* https://docs.ansible.com/ansible/latest/modules/lxd_container_module.html

### lxd

* https://blog.simos.info/how-to-initialize-lxd-again/
* [Setup a ZFS pool for your LXC containers with LXD](https://angristan.xyz/lxc-zfs-pool-lxd/)
* https://blog.ubuntu.com/2016/02/16/zfs-is-the-fs-for-containers-in-ubuntu-16-04
* https://github.com/lxc/lxd/blob/master/doc/storage.md
* [Ubuntu 16.04 - Using Files To Test ZFS](https://blog.programster.org/ubuntu-16-04-using-files-to-test-zfs)
* https://github.com/lxc/lxd/blob/master/doc/storage.md

## Recreate botched zfs storage as a file

https://serverfault.com/questions/583733/how-to-use-a-file-as-a-zpool

```shell
sudo dd if=/dev/zero of=/var/lib/lxd/default.img bs=1M count=128
sudo zpool create test /var/lib/lxd/default.img
sudo zpool list
```

Error message to solve:

```shell
csteel@a3d-laptop:~/projects/exo/ansible/roles/myrole$ systemctl status lxd
● lxd.service - LXD - main daemon
   Loaded: loaded (/lib/systemd/system/lxd.service; indirect; vendor preset: enabled)
   Active: activating (start-post) (Result: exit-code) since Thu 2018-12-13 18:28:12 EST; 8min ago
     Docs: man:lxd(1)
  Process: 5818 ExecStart=/usr/bin/lxd --group lxd --logfile=/var/log/lxd/lxd.log (code=exited, status=1/FAILURE)
  Process: 5799 ExecStartPre=/usr/lib/x86_64-linux-gnu/lxc/lxc-apparmor-load (code=exited, status=0/SUCCESS)
 Main PID: 5818 (code=exited, status=1/FAILURE);         : 5819 (lxd)
    Tasks: 8
   Memory: 5.8M
      CPU: 133ms
   CGroup: /system.slice/lxd.service
           └─control
             └─5819 /usr/lib/lxd/lxd waitready --timeout=600

Dec 13 18:28:12 a3d-laptop systemd[1]: Starting LXD - main daemon...
Dec 13 18:28:12 a3d-laptop lxd[5818]: lvl=warn msg="CGroup memory swap accounting is disabled, swap limits will be ignored." t=2018
Dec 13 18:28:12 a3d-laptop lxd[5818]: lvl=eror msg="Failed to start the daemon: ZFS storage pool \"default\" could not be imported:
Dec 13 18:28:12 a3d-laptop lxd[5818]: Error: ZFS storage pool "default" could not be imported: cannot import 'default': no such poo
Dec 13 18:28:12 a3d-laptop systemd[1]: lxd.service: Main process exited, code=exited, status=1/FAILURE
```

Restart / reload lxd service

```shell
sudo systemctl stop lxd
sudo systemctl start lxd
sudo systemctl status lxd
```

Status output when working:

```shell

● lxd.service - LXD - main daemon
   Loaded: loaded (/lib/systemd/system/lxd.service; indirect; vendor preset: enabled)
   Active: active (running) since Thu 2018-12-13 18:37:11 EST; 16s ago
     Docs: man:lxd(1)
  Process: 6421 ExecStartPost=/usr/bin/lxd waitready --timeout=600 (code=exited, status=0/SUCCESS)
  Process: 6399 ExecStartPre=/usr/lib/x86_64-linux-gnu/lxc/lxc-apparmor-load (code=exited, status=0/SUCCESS)
 Main PID: 6420 (lxd)
    Tasks: 25
   Memory: 18.2M
      CPU: 615ms
   CGroup: /system.slice/lxd.service
           ├─6420 /usr/lib/lxd/lxd --group lxd --logfile=/var/log/lxd/lxd.log
           └─6629 dnsmasq --strict-order --bind-interfaces --pid-file=/var/lib/lxd/networks/lxdbr0/dnsmasq.pid --except-interface=l

Dec 13 18:37:10 a3d-laptop systemd[1]: Starting LXD - main daemon...
Dec 13 18:37:10 a3d-laptop lxd[6420]: lvl=warn msg="CGroup memory swap accounting is disabled, swap limits will be ignored." t=2018
Dec 13 18:37:10 a3d-laptop lxd[6420]: 2018/12/13 18:37:10 http: multiple response.WriteHeader calls
Dec 13 18:37:10 a3d-laptop lxd[6420]: 2018/12/13 18:37:10 http: multiple response.WriteHeader calls
Dec 13 18:37:10 a3d-laptop lxd[6420]: 2018/12/13 18:37:10 http: multiple response.WriteHeader calls
Dec 13 18:37:11 a3d-laptop systemd[1]: Started LXD - main daemon.
Dec 13 18:37:20 a3d-laptop systemd[1]: Started LXD - main daemon.
9
```



##  Fast reinitialization

```shell
lxc list
lxc delete mytest

lxc image list
lxc image delete e03fa828cdb0
lxc image list

lxc storage list
```

output

```shell
+------+-------------+--------+----------------------------+---------+
| NAME | DESCRIPTION | DRIVER |           SOURCE           | USED BY |
+------+-------------+--------+----------------------------+---------+
| lxd  |             | zfs    | /var/lib/lxd/disks/lxd.img | 1       |
+------+-------------+--------+----------------------------+---------+
```

```shell
sudo zpool destroy -f lxd
lxc profile delete default
lxc storage delete default
lxd init
```

create storage pool

```shell
lxc storage create default zfs
```



## /var/lib/lxd/containers/

Created when you configure LXD to use the **dir storage** backend.

```shell
sudo ls -al /var/lib/lxd/containers
```

## Configured as loop file

when you configure LXD to use the **zfs storage** backend (on a loop file and **not** a block device) you get either:

```shell
/lxd/containers/zfs.img
```

or

```shell
/var/lib/lxd/containers/disks/
```

in newser versions of LXD.

## Configured on a block device

You get a block device (partition) that is formatted with ZFS (or btrfs) when you configure LXD to use the **zfs storage** backend on a block device.

## Storage backend: zfs

```shell
lxc config show
config:
  storage.zfs_pool_name: lxd
```

### loopfile

Here we see that we are using a loop file rather than a block device. The loop file is located here:

```shell
/var/lib/lxd/zfs.img
```

Here is our test and output:

```shell
sudo zpool status
  pool: lxd
 state: ONLINE
  scan: scrub repaired 0 in 0h0m with 0 errors on Sun Aug 12 00:24:03 2018
config:

	NAME                    STATE     READ WRITE CKSUM
	lxd                     ONLINE       0     0     0
	  /var/lib/lxd/zfs.img  ONLINE       0     0     0

errors: No known data errors
```

## Cleaning up containers

```shell
lxc list
```

then stop and delete any containers

```shell
$ lxc stop some_container
$ lxc delete some_container
$ lxc list
```

Alternate output formating:

```shell
lxc list --format csv -c n
```

output:

```shell
instance
xenial
```

To delete them all

```shell
for i in $(lxc list --format csv -c n); do
    lxc delete $i --force
done
```

Confirm:

```shell
lxc list
```

## Cleaning up the images

We are going to list the cached images, then delete them. Until the list is empty!

```
lxc image list
```

Output example:



Not delete any container images listed:

```shell
lxc image delete
```

Using a loop

```shell
for i in $(lxc image list --format csv -c f); do
    lxc image delete $i
done
```



## Clearing up the ZFS storage

show our config:

```SHELL
lxc config show
```

output example:

```shell
config:
  storage.zfs_pool_name: lxd
```

unset our zfs storage:

```shell
lxc config unset storage.zfs_pool_name
```

Confirm

```shell
lxc config show
```

output example:

```shell
config: {}
```

## LXC Profiles

```shell
lxc profile list
```

Show

```shell
lxc profile show default
```

Delete

```shell
lxc profile delete default
```

Confirm

```shell
lxc profile list
```

## LXC Network

```shell
lxc network list
```

output example:

```shell
+-----------+----------+---------+-------------+---------+
|   NAME    |   TYPE   | MANAGED | DESCRIPTION | USED BY |
+-----------+----------+---------+-------------+---------+
| enp0s31f6 | physical | NO      |             | 0       |
+-----------+----------+---------+-------------+---------+
| lxdbr0    | bridge   | YES     |             | 0       |
+-----------+----------+---------+-------------+---------+
```

Delete

```shell
lxc network delete lxdbr0
```

Confirm

```shell
lxc network list
```

Output example:

```shell
+-----------+----------+---------+-------------+---------+
|   NAME    |   TYPE   | MANAGED | DESCRIPTION | USED BY |
+-----------+----------+---------+-------------+---------+
| enp0s31f6 | physical | NO      |             | 0       |
+-----------+----------+---------+-------------+---------+
```

## configure the LXC storage

```shell
lxc storage list
```

Output example:

```shell
+---------+-------------+--------+--------------------------------+---------+
|  NAME   | DESCRIPTION | DRIVER |             SOURCE             | USED BY |
+---------+-------------+--------+--------------------------------+---------+
| default |             | zfs    | /var/lib/lxd/disks/default.img | 0       |
+---------+-------------+--------+--------------------------------+---------+
| lxd     |             | zfs    | /var/lib/lxd/disks/lxd.img     | 1       |
+---------+-------------+--------+--------------------------------+---------+
```

Delete your unused strorage pool, in my case **default**

```shell
lxc storage delete default
```

Confirm

```shell
lxc storage list
sudo zfs list
sudo zpool list
```

## For lxc 2.x destroy the ZFS pool this way

List our pool(s)

```shell
sudo zpool list
```

output example:

```shell
NAME      SIZE  ALLOC   FREE  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
default  87.5G  4.31M  87.5G         -     1%     0%  1.00x  ONLINE  -
```

Destroy any pools using something like :

```shell
sudo zpool destroy default
```

Confirm that our pool(s) is/are all gone:

```shell
sudo zpool list
```

Output example:

```shell
no pools available
```

## Ensure all datasets are destroyed as well

```shell
sudo zfs list
```

Desired output:

```shell
no datasets available
```

## configure the LXC storage

```shell
lxc storage list
```

Output example:

```shell
+---------+-------------+--------+--------------------------------+---------+
|  NAME   | DESCRIPTION | DRIVER |             SOURCE             | USED BY |
+---------+-------------+--------+--------------------------------+---------+
| default |             | zfs    | /var/lib/lxd/disks/default.img | 0       |
+---------+-------------+--------+--------------------------------+---------+
| lxd     |             | zfs    | /var/lib/lxd/disks/lxd.img     | 1       |
+---------+-------------+--------+--------------------------------+---------+
```

Delete your unused strorage pool, in my case **default**

```shell
lxc storage delete default
```

Confirm

```shell
lxc storage list
sudo zfs list
sudo zpool list
```



## LXD INIT

```shell
sudo lxd init
```

output example:

```shell
Would you like to use LXD clustering? (yes/no) [default=no]: 
Do you want to configure a new storage pool? (yes/no) [default=yes]: 
Name of the new storage pool [default=default]: lxd
Name of the storage backend to use (dir, zfs) [default=zfs]: 
Create a new ZFS pool? (yes/no) [default=yes]: 
Would you like to use an existing block device? (yes/no) [default=no]: 
Size in GB of the new loop device (1GB minimum) [default=88GB]: 
Would you like to connect to a MAAS server? (yes/no) [default=no]: 
Would you like to create a new local network bridge? (yes/no) [default=yes]: 
What should the new bridge be called? [default=lxdbr0]: 
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: none
Would you like LXD to be available over the network? (yes/no) [default=no]: 
Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: yes
```

Preseed of above:

```shell
config: {}
cluster: null
networks:
- config:
    ipv4.address: auto
    ipv6.address: none
  description: ""
  managed: false
  name: lxdbr0
  type: ""
storage_pools:
- config:
    size: 88GB
  description: ""
  name: lxd
  driver: zfs
profiles:
- config: {}
  description: ""
  devices:
    eth0:
      name: eth0
      nictype: bridged
      parent: lxdbr0
      type: nic
    root:
      path: /
      pool: lxd
      type: disk
  name: default
```

## Testing

```shell
lxc launch images:ubuntu/xenial/amd64 instance-001
```



## Troubleshooting

### lxd will not restart

```shell
Sep 28 13:51:48 ace-ws-101 systemd[1]: Starting LXD - main daemon...
Sep 28 13:51:48 ace-ws-101 lxd[32172]: lvl=warn msg="CGroup memory swap accounting is disabled, swap limits will be ignored." t=2018-09-28T13:51:48-0400
Sep 28 13:51:48 ace-ws-101 lxd[32172]: lvl=eror msg="Failed to start the daemon: ZFS storage pool \"default\" could not be imported: cannot import 'default'
Sep 28 13:51:48 ace-ws-101 lxd[32172]: Error: ZFS storage pool "default" could not be imported: cannot import 'default': no such pool available
Sep 28 13:51:48 ace-ws-101 systemd[1]: lxd.service: Main process exited, code=exited, status=1/FAILURE
```

The fix when the zfs pool has been deleted:

```shell
truncate --size 10G /tmp/default.img
```

Create the temporary pool from your temporary file image

```shell
sudo zpool create default /tmp/default.img
```

Confirm:

```shell
sudo zpool list
```

Try and start LXD now

```shell
sudo systemctl stop lxd
sudo systemctl start lxd
```

Confirm

```shell
sudo systemctl status lxd
```

##  Set the new storage pool name

```shell
lxc config set storage.zfs_pool_name lxd
```



## Testing

### profiles

```shell
lxc profile list
```

output:

```shell
default
docker
devprofile
```

```shell
lxc profile show default
```
Output:

```shell
config:
  environment.http_proxy: ""
  user.network_mode: ""
description: Default LXD profile
devices:
  eth0:
    name: eth0
    nictype: bridged
    parent: lxdbr0
    type: nic
name: default
used_by: []
```

```shell
lxc profile show docker
```
Output:

```shell
config:
  linux.kernel_modules: overlay, nf_nat
  security.nesting: "true"
description: Profile supporting docker in containers
devices:
  aadisable:
    path: /sys/module/apparmor/parameters/enabled
    source: /dev/null
    type: disk
name: docker
used_by: []
```

```shell
lxc profile show devprofile
```
Output:

```shell
config:
  environment.http_proxy: ""
  user.network_mode: ""
description: Default LXD profile
devices:
  eth0:
    name: eth0
    nictype: bridged
    parent: lxdbr0
    type: nic
name: devprofile
used_by: []
```

## lxd-bridge

```shell
sudo cat /etc/default/lxd-bridge 
```

Output:

```shell
# WARNING: This file is generated by a debconf template!
# It is recommended to update it by using "dpkg-reconfigure -p medium lxd"

# Whether to setup a new bridge or use an existing one
USE_LXD_BRIDGE="true"

# Bridge name
# This is still used even if USE_LXD_BRIDGE is set to false
# set to an empty value to fully disable
LXD_BRIDGE="lxdbr0"

# Update the "default" LXD profile
UPDATE_PROFILE="true"

# Path to an extra dnsmasq configuration file
LXD_CONFILE=""

# DNS domain for the bridge
LXD_DOMAIN="lxd"

# IPv4
## IPv4 address (e.g. 10.0.8.1)
LXD_IPV4_ADDR="10.194.29.1"

## IPv4 netmask (e.g. 255.255.255.0)
LXD_IPV4_NETMASK="255.255.255.0"

## IPv4 network (e.g. 10.0.8.0/24)
LXD_IPV4_NETWORK="10.194.29.1/24"

## IPv4 DHCP range (e.g. 10.0.8.2,10.0.8.254)
LXD_IPV4_DHCP_RANGE="10.194.29.2,10.194.29.254"

## IPv4 DHCP number of hosts (e.g. 250)
LXD_IPV4_DHCP_MAX="252"

## NAT IPv4 traffic
LXD_IPV4_NAT="true"

# IPv6
## IPv6 address (e.g. 2001:470:b368:4242::1)
LXD_IPV6_ADDR=""

## IPv6 CIDR mask (e.g. 64)
LXD_IPV6_MASK=""

## IPv6 network (e.g. 2001:470:b368:4242::/64)
LXD_IPV6_NETWORK=""

## NAT IPv6 traffic
LXD_IPV6_NAT="false"

# Run a minimal HTTP PROXY server
LXD_IPV6_PROXY="false"
```

```shell
sudo cat /etc/default/zfs 
# ZoL userland configuration.

# Wait this many seconds during system start for pool member devices to appear
# before attempting import and starting mountall.
ZFS_AUTOIMPORT_TIMEOUT='30'

# Run `zfs share -a` during system start?
# nb: The shareiscsi, sharenfs, and sharesmb dataset properties.
ZFS_SHARE='no'

# Run `zfs unshare -a` during system stop?
ZFS_UNSHARE='no'

# Build kernel modules with the --enable-debug switch?
ZFS_DKMS_ENABLE_DEBUG='no'

# Build kernel modules with the --enable-debug-dmu-tx switch?
ZFS_DKMS_ENABLE_DEBUG_DMU_TX='no'

# Keep debugging symbols in kernel modules?
ZFS_DKMS_DISABLE_STRIP='no'
```

## LXD molecule.yml example

```shell
---
dependency:
  name: galaxy
driver:
  name: lxd
lint:
  name: yamllint
platforms:
  - name: xenial
    source:
      type: image
      mode: pull
      server: https://images.linuxcontainers.org
      protocol: lxd
      aliase: ubuntu/xenial/amd64
    profiles: ["default"]
  - name: bionic
    source:
      type: image
      mode: pull
      server: https://images.linuxcontainers.org
      protocol: lxd
      aliase: ubuntu/bionic/amd64
    profiles: ["default"]
  - name: centos6
    source:
      type: image
      mode: pull
      server: https://images.linuxcontainers.org
      protocol: lxd
      aliase: centos/6/amd64
    profiles: ["default"]
  - name: centos7
    source:
      type: image
      mode: pull
      server: https://images.linuxcontainers.org
      protocol: lxd
      aliase: centos/7/amd64
    profiles: ["default"]
provisioner:
  name: ansible
  log: true
  lint:
    name: ansible-lint
scenario:
  name: default
verifier:
  name: testinfra
  lint:
    name: flake8
```

## molecule/provisioner/ansible/playbooks/lxd

### create.yml

```shell
---
- name: Create
  hosts: localhost
  connection: local
  gather_facts: false
  no_log: "{{ not (lookup('env', 'MOLECULE_DEBUG') | bool or molecule_yml.provisioner.log|default(false) | bool) }}"
  tasks:
    - name: Create default source variable
      set_fact:
        default_source:
          type: image
          server: https://images.linuxcontainers.org
          alias: ubuntu/xenial/amd64

    - name: Create molecule instance(s)
      lxd_container:
        url: "{{ item.url | default(omit) }}"
        cert_file: "{{ item.cert_file | default(omit) }}"
        key_file: "{{ item.key_file | default(omit) }}"
        trust_password: "{{ item.trust_password | default(omit) }}"
        name: "{{ item.name }}"
        state: started
        source: "{{ default_source | combine(item.source | default({})) }}"
        config: "{{ item.config | default(omit) }}"
        architecture: "{{ item.architecture | default(omit) }}"
        devices: "{{ item.devices | default(omit) }}"
        profiles: "{{ item.profiles | default(omit) }}"
        wait_for_ipv4_addresses: true
        timeout: 600
      with_items: "{{ molecule_yml.platforms }}"
```

## Ansible module defaults

```shell
source:
  type: image  # Can be: "image", "migration", "copy" or "none"
  mode: pull  # One of "local" (default) or "pull"
  server: https://10.0.2.3:8443 # Remote server (pull mode only)
  protocol: lxd   # Protocol (one of lxd or simplestreams, defaults to lxd)
  certificate: PEM certificate # Optional PEM certificate. If not mentioned, system CA is used.
  alias: ubuntu/bionic/amd64   # Name of the alias
```



## molecule/test/resources/playbooks/lxd

### create.yml

```shell
---
- name: Create
  hosts: localhost
  connection: local
  gather_facts: false
  no_log: "{{ not (lookup('env', 'MOLECULE_DEBUG') | bool or molecule_yml.provisioner.log|default(false) | bool) }}"
  tasks:
    - name: Create default source variable
      set_fact:
        default_source:
          type: image
          server: https://images.linuxcontainers.org
          alias: ubuntu/xenial/amd64

    - name: Create molecule instance(s)
      lxd_container:
        url: "{{ item.url | default(omit) }}"
        cert_file: "{{ item.cert_file | default(omit) }}"
        key_file: "{{ item.key_file | default(omit) }}"
        trust_password: "{{ item.trust_password | default(omit) }}"
        name: "{{ item.name }}"
        state: started
        source: "{{ default_source | combine(item.source | default({})) }}"
        config: "{{ item.config | default(omit) }}"
        architecture: "{{ item.architecture | default(omit) }}"
        devices: "{{ item.devices | default(omit) }}"
        profiles: "{{ item.profiles | default(omit) }}"
        wait_for_ipv4_addresses: true
        timeout: 600
      with_items: "{{ molecule_yml.platforms }}"
```

##  lxd_container.py

ansible-2.6.4/local/lib/python2.7/site-packages/ansible/modules/cloud/lxd/lxd_container.py

## Testing lxd

```shell
lxc exec instance -- su --login ubuntu

and

lxc exec instance -- sudo --login --user ubuntu
```

## LXC

```shell
nano /etc/default/lxc-net
```

change:

```shell
USE_LXC_BRIDGE="true"

to

USE_LXC_BRIDGE="false"
```

then:

```shell
sudo apt-get purge lxc
```

restart system