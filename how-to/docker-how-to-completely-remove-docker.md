# docker-how-to-completely-remove-docker.md

## List installed packages

```shell
dpkg -l | grep -i docker
```

Output example:

```shell
rc  docker                                        1.5-1                                        amd64        System tray for KDE3/GNOME2 docklet applications
ii  docker-ce                                     5:18.09.0~3-0~ubuntu-xenial                  amd64        Docker: the open-source application container engine
ii  docker-ce-cli                                 5:18.09.0~3-0~ubuntu-xenial                  amd64        Docker CLI: the open-source application container engine
```

## Purge installed packages


```shell
sudo apt-get purge -y docker docker.ce docker-ce-cli
sudo apt-get autoremove -y --purge docker docker.ce docker-ce-cli
```

## Remove artifacts

(untested)

* images
* containers
* volumes
* user created configuration files

```shell
sudo rm -rf /var/lib/docker
ls -al /var/lib/ | grep docker
sudo rm /etc/apparmor.d/docker
ls -al /etc/apparmor.d/ | grep docker
sudo groupdel docker
sudo rm -rf /var/run/docker.sock
```

## Reboot the system

Ensure that docker network interface(s) are removed.

## Confirm

```sh
sudo ifconfig
```

That is it

## Ubuntu 16.04 example

### docker io

```shell
dpkg -l | grep -i docker
sudo apt-get purge -y docker docker.io
sudo apt-get autoremove -y --purge docker docker-io
```

### docker ce

```shell
dpkg -l | grep -i docker
sudo apt-get purge -y docker docker-ce
sudo apt-get autoremove -y --purge docker docker-ce
```


