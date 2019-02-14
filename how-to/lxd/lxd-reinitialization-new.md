https://discuss.linuxcontainers.org/t/how-to-remove-lxd-from-my-system/2336/4

## General Idea / Tricky

You may need to manually edit the default profile so that it refers to nothing. In other words delete everything but "config: {}". LXC may recreate the file with other entries with "empty" values but you should be able to delete underlying zfs storage using the lxc storage delete command.


    lxc list
    lxc delete <whatever came from list>
    lxc image list
    lxc image delete <whatever came from list>
    lxc network list
    lxc network delete <whatever came from list>
    # alternativly run "lxc profile edit default" and make content "config: {}"
    echo ‘{“config”: {}}’ | lxc profile edit default
    lxc storage volume list default
    lxc storage volume list lxd
    lxc storage volume delete default <whatever came from list>
    lxc storage delete default
```shell
sudo systemctl stop lxd
sudo systemctl start lxd
```

Unintsall 

```shell
sudo apt remove -t xenial-backports lxd lxd-client
sudo zpool list
no pools available
reboot
```

Run lxd init (without sudo)

```shell
lxd init
```

lxd init preseed command and example file:

```shell
lxd init --preseed custom_preseed_file 
```

content of custom_preseed_file:

```shell
config: {}
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
    size: 79GB
  description: ""
  name: default
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
      pool: default
      type: disk
  name: default
cluster: null
```

## Installation Confirmation

```shell
lxc profile show default
```

### Launch a container

```shell
lxc launch images:ubuntu/xenial c1
```

In addition to container management the LXC/LXD CLI also allows you to interact storage, networking and devices, so for example, you can work with the underlying ZFS storage we use and do not need to mess with the ZFS CLI at all.

------

### Launching a Test Container

Here we launch a container called c1 from a Canonical (Container) Image Server. *images* is an alias for this particular server and *ubuntu/xenial* is the name of the container image we want.

```shell
lxc launch images:ubuntu/xenial c2
```

------

## Lets see how much space is used by our new container image:

```shell
lxc storage info default
```

output example:

```shell
info:
  description: ""
  driver: zfs
  name: lxd
  space used: ?.??GB
  total space: 17.89GB
used by:
  images:
  - 34851ebf08f1c375504b9d83cd037f9cb09ce7ca2d9d0c45b28c98c87b6242e7
  profiles:
  - default
```

------

## Creating a second container

- `lxc launch` first checks to see if this image has been updated on the *images* server.
- It has not so `lxc launch` clones a second container from the existing local image.
- It's up in ready to go almost instantly.

```shell
lxc launch images:ubuntu/xenial c2
```

------

## Lets see how much space is used by our second container:

```shell
lxc storage info lxd
```

------

## Additional Space Used By Our Second Container?

output example:

```shell
info:
  description: ""
  driver: zfs
  name: lxd
  space used: ?.??GB
  total space: 17.89GB
used by:
  containers:
  - c1
  - c2
  images:
  - 34851ebf08f1c375504b9d83cd037f9cb09ce7ca2d9d0c45b28c98c87b6242e7
  profiles:
  - default
```

------

## List our running Containers

```shell
lxc list
```

------

## Notes before moving on

- COW Power + Linux Containers = Something Bordering on Magic.
- The `lxd init` and `lxc` CLI  allows us to use LXD with ZFS without the needing to know much about ZFS or the ZFS CLI.
- LXD on ZFS is very fast./*
- LXD on ZFS is very space efficient.