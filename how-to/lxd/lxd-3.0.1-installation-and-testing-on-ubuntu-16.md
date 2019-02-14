# lxd-3.0.1-on-ubuntu-16.md

## Installation

### Requirements

Ensure that python3-dev and python3-venv are installed:

```shell
sudo apt install -y python3-venv python3-dev
```

You will need to install zfsutils first as we will be setting up LXD with ZFS storage (zpool).

```shell
sudo apt-get install -y zfsutils-linux
```

### Installing LXD 3.x

To install LXD 3.x  you will need to use the backport or a snap install on Ubuntu 16.04.

Before starting you need to remove any older versiona of LXD.  See [lxd-reinitialization](lxd-reinitialization/lxd-reinitialization.md) or [lxd-reinitialization/lxd-reinitialization-lxd-3.0.1.md](lxd-reinitialization/lxd-reinitialization-lxd-3.0.1.md) if you have accidentally "upgraded" to 3.x before completely removing LXD 2.x.

### backports

For Ubuntu 16.04 I demo a [ Ubuntu Backports](https://help.ubuntu.com/community/UbuntuBackports#Enabling_Backports_on_Ubuntu_Desktop) installation [ UbuntuBackports](https://help.ubuntu.com/community/UbuntuBackports#Enabling_Backports_on_Ubuntu_Desktop) although in future LXD releases the "official" LXD 3.x install may be done using snap packages.

#### Enable Ubuntu backports

Edit /etc/apt/sources.list and uncomment the backports lines.

```shell
deb http://ca.archive.ubuntu.com/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://ca.archive.ubuntu.com/ubuntu/ xenial-backports main restricted universe multiverse
```

```shell
sudo apt update
sudo apt -y upgrade
```

Install txConfirm your version

```shell
lxc --version
```

Output example:

```shell
3.0.1
```

## lxd init

On a standard Desktop or Server. Here we go with the defaults, although we choose the call our (ZFS) storage pool **lxd** rather than **default** and for **IPv6** we choose **none** rather than auto.


```shell
csteel@ace-ws-101:~/projects/molecule/2.18.1-lxd$ lxd init
Would you like to use LXD clustering? (yes/no) [default=no]: 
Do you want to configure a new storage pool? (yes/no) [default=yes]: 
Name of the new storage pool [default=default]: lxd 
Name of the storage backend to use (dir, zfs) [default=zfs]: 
Create a new ZFS pool? (yes/no) [default=yes]: 
Would you like to use an existing block device? (yes/no) [default=no]: 
Size in GB of the new loop device (1GB minimum) [default=22GB]: 
Would you like to connect to a MAAS server? (yes/no) [default=no]: 
Would you like to create a new local network bridge? (yes/no) [default=yes]: 
What should the new bridge be called? [default=lxdbr0]:                
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: none
Would you like LXD to be available over the network? (yes/no) [default=no]: 
Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: yes
```

Prseed printout:

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
    size: 22GB
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

### lxd init container install

```shell
Would you like to use LXD clustering? (yes/no) [default=no]: 
Do you want to configure a new storage pool? (yes/no) [default=yes]: 
Name of the new storage pool [default=default]: lxd
Would you like to connect to a MAAS server? (yes/no) [default=no]: 
Would you like to create a new local network bridge? (yes/no) [default=yes]: 
What should the new bridge be called? [default=lxdbr0]: 
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: none

We detected that you are running inside an unprivileged container.
This means that unless you manually configured your host otherwise,
you will not have enough uids and gids to allocate to your containers.

LXD can re-use your container's own allocation to avoid the problem.
Doing so makes your nested containers slightly less safe as they could
in theory attack their parent container and gain more privileges than
they otherwise would.

Would you like to have your containers share their parent's allocation? (yes/no) [default=yes]: 
Would you like LXD to be available over the network? (yes/no) [default=no]: 
Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: 
```



## Confirm the installation and test it

### Activate your lxd group membership

`lxd init` will create a new group called **lxd** and add your user to it. You will need to reload your groups afterwards. You can do this by opening a new terminal session or using the folleoing method:

Start a new session:

```
su - $USER
```

after that check access level again:

```
id
```

In my output the last entry **131(lxd)** confirms that my user membership in the lxd group is now active:

```shell
uid=1000(cjs) gid=1000(cjs) groups=1000(cjs),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare),131(lxd)
```

### Ensure that the lxd service is started

```shell
sudo systemctl start lxd
sudo systemctl status lxd
```

### Ensure that the the ZFS pool was created

LXD provides a layer of abstraction for working with and controlling the ZFS storage backing your LXD installation.

```shell
lxc storage show lxd
```

Output example on a working system:

```shell
config:
  size: 22GB
  source: /var/lib/lxd/disks/lxd.img
  zfs.pool_name: lxd
description: ""
name: lxd
driver: zfs
used_by:
- /1.0/profiles/default
status: Created
locations:
- none
```

Everything looks good!

### Launching your first container

To make sure that everything is working correctly together we will first manually launch an LXD container running Ubuntu Xenial. Since this is the first time we are doing this the image will be downloaded from the Canonical (Ubuntu) image server. Here we go:

#### Initial lxc launch of an ubuntu xenial container

If a container image is not available locally then LXD will download it from the Canonical images server that is configured by default when running `sudo lxd init` for your convience. So on the first run our command below will:

1. Download the requested container image.
2. Use the image to create a new container image.
3. Launch a container called **c1**.

```shell
lxc launch images:ubuntu/xenial c1 --verbose
```

Your output will look a bit like this while is get your image from the Canonical image server:

```shell
Creating C1
Retrieving image: rootfs: 37% (1.88MB/s)
```

and once the image is retrieved, cloned and a new container has been launch it will look like this:

```shell
Creating c1
Starting c1
```

You can take a look at it by running the command `lxc list`. In the example output below you see that the container has been assigned an IP address and looking at the STATE column show us that it is running:

```shell
lxc list
```

Output:

```shell
+------+---------+-----------------------+------+------------+-----------+
| NAME |  STATE  |         IPV4          | IPV6 |    TYPE    | SNAPSHOTS |
+------+---------+-----------------------+------+------------+-----------+
| c1   | RUNNING | 10.240.125.218 (eth0) |      | PERSISTENT | 0         |
+------+---------+-----------------------+------+------------+-----------+
```

In addition to our running container, LXD has also retrieved the base image from the Canonical image server. Lets take a look at that now:

```shell
lxc image list
```

Here is our output:

```shell
+-------+--------------+--------+--------------------------------------+--------+----------+-----------------------------+
| ALIAS | FINGERPRINT  | PUBLIC |             DESCRIPTION              |  ARCH  |   SIZE   |         UPLOAD DATE         |
+-------+--------------+--------+--------------------------------------+--------+----------+-----------------------------+
|       | 950e32336b65 | no     | Ubuntu xenial amd64 (20181004_07:42) | x86_64 | 105.43MB | Oct 4, 2018 at 4:44pm (UTC) |
+-------+--------------+--------+--------------------------------------+--------+----------+-----------------------------+
```

### Copy On Write (COW) Magic

Now we are going to see some magic

Lets take a look at how much storage LXD is using now that our  `lxd launch` command has completed it's tasks:

```shell
lxc storage info lxd
```

Here is my output:

```shell
info:
  description: ""
  driver: zfs
  name: lxd
  space used: 173.65MB
  total space: 21.02GB
used by:
  containers:
  - c1
  images:
  - 950e32336b6551a7cc619bbc318d97bf64ebc26be4e8daa971539a1b0e616f0d
  profiles:
  - default
```

So in total I have used 173.65MB of 21.02GB in total.

#### Create a second container for some COW magic

```shell
lxc launch images:ubuntu/xenial c2 --verbose
```

This should have run much faster than our first container launch. 

Lets check out our space again

```shell
lxc storage info lxd
```

Output:

```shell
info:
  description: ""
  driver: zfs
  name: lxd
  space used: 176.33MB
  total space: 21.02GB
used by:
  containers:
  - c1
  - c2
  images:
  - 950e32336b6551a7cc619bbc318d97bf64ebc26be4e8daa971539a1b0e616f0d
  profiles:
  - default
```

Not only where we able to launch our second container almost instantly due to COW it also takes up almost no space at all!

Here is the math:

176.33 - 173.65  = 2.68 (MB) of space used for our second container. To see how that is possible check out the Wikipedia entries for [Copy On Write](https://en.wikipedia.org/wiki/Copy-on-write) and it's implementation in [ZFS](https://en.wikipedia.org/wiki/ZFS).

Wow! So some of the most striking advantages of the LXD on ZFS stack vs. VM's are:

* Significantly faster than VM's.
* Requires significantly less space.

In addition LXD containers are reported to run at near metal speeds. Roughly 2% slower than if you where working directly on the host operating system (When your container stack has been optimised, and ours is not).

### Managing images and containers

The lxc help is actually very helpfull. Appendng any command or command combination with `-h` or `--help` usually results in what you want to know.

#### Manipulating Containers

```shell
lxc start container_name
lxc stop container_name
and
lxc delete container_name

```

Lets give it a shot

```shell
lxc stop c2
lxc list
```

our results:

```shell
+------+---------+-----------------------+------+------------+-----------+
| NAME |  STATE  |         IPV4          | IPV6 |    TYPE    | SNAPSHOTS |
+------+---------+-----------------------+------+------------+-----------+
| c1   | RUNNING | 10.240.125.218 (eth0) |      | PERSISTENT | 0         |
+------+---------+-----------------------+------+------------+-----------+
| c2   | STOPPED |                       |      | PERSISTENT | 0         |
+------+---------+-----------------------+------+------------+-----------+
```

Lets delete this one:

```shell
lxc delete c2
lxc list
```

Output:

```shell
+------+---------+-----------------------+------+------------+-----------+
| NAME |  STATE  |         IPV4          | IPV6 |    TYPE    | SNAPSHOTS |
+------+---------+-----------------------+------+------------+-----------+
| c1   | RUNNING | 10.240.125.218 (eth0) |      | PERSISTENT | 0         |
+------+---------+-----------------------+------+------------+-----------+
```



## Testing



## Confirm your installation



```shell
for i in $(lxc-ls -1); do
  lxc-info -n $i
done
```

