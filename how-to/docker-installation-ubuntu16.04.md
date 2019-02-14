# docker-installation-ubuntu16.04.md

## References

* https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04

## Installation

### Ubuntu 16.04

add the GPG key for the official Docker repository:

```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Add the Docker repository to APT sources:

```shell
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

update

```shell
sudo apt-get update
```

Make sure you are about to install from the Docker repo instead of the default Ubuntu 16.04 repo:

```
sudo apt-cache policy docker-ce
```

You should see output similar to the follow:

Output of apt-cache policy docker-ce

```
docker-ce:
  Installed: (none)
  Candidate: 5:18.09.0~3-0~ubuntu-xenial
  Version table:
     5:18.09.0~3-0~ubuntu-xenial 500
        500 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
     18.06.1~ce~3-0~ubuntu 500
        500 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
     18.06.0~ce~3-0~ubuntu 500
...
```

Notice that `docker-ce` is not installed, but the candidate for installation is from the Docker repository for Ubuntu 16.04 (`xenial`).

Finally, install Docker:

```shell
sudo apt-get install -y docker-ce
```

Docker should now be installed, the daemon started, and the process enabled to start on boot. Check that it's running:

```shell
sudo systemctl status docker
```

The output should be similar to the following, showing that the service is active and running:

```shell
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2018-12-06 12:50:54 EST; 4s ago
     Docs: https://docs.docker.com
 Main PID: 18372 (dockerd)
   CGroup: /system.slice/docker.service
           └─18372 /usr/bin/dockerd -H unix://

Dec 06 12:50:54 ace-ws-101 dockerd[18372]: time="2018-12-06T12:50:54.573874749-05:00" level=warning msg="Your kernel does not support swap memory limit"
Dec 06 12:50:54 ace-ws-101 dockerd[18372]: time="2018-12-06T12:50:54.573925651-05:00" level=warning msg="Your kernel does not support cgroup rt period"
Dec 06 12:50:54 ace-ws-101 dockerd[18372]: time="2018-12-06T12:50:54.573939631-05:00" level=warning msg="Your kernel does not support cgroup rt runtime"
Dec 06 12:50:54 ace-ws-101 dockerd[18372]: time="2018-12-06T12:50:54.574293750-05:00" level=info msg="Loading containers: start."
Dec 06 12:50:54 ace-ws-101 dockerd[18372]: time="2018-12-06T12:50:54.674632926-05:00" level=info msg="Default bridge (docker0) is assigned with an IP address 
Dec 06 12:50:54 ace-ws-101 dockerd[18372]: time="2018-12-06T12:50:54.737679846-05:00" level=info msg="Loading containers: done."
Dec 06 12:50:54 ace-ws-101 dockerd[18372]: time="2018-12-06T12:50:54.826287859-05:00" level=info msg="Docker daemon" commit=4d60db4 graphdriver(s)=overlay2 ve
Dec 06 12:50:54 ace-ws-101 dockerd[18372]: time="2018-12-06T12:50:54.826358016-05:00" level=info msg="Daemon has completed initialization"
Dec 06 12:50:54 ace-ws-101 systemd[1]: Started Docker Application Container Engine.
Dec 06 12:50:54 ace-ws-101 dockerd[18372]: time="2018-12-06T12:50:54.869624332-05:00" level=info msg="API listen on /var/run/docker.sock"
```

### Installation Confirmation

#### /var/run/docker.sock permissions

Newer Docker installs should correctly set the permissions on the docker socket file

```shell
ls -al /var/run/ | grep docker
```

Output when permissions are correct:

```shell
drwx------  6 root                root                 140 Feb 13 14:58 docker
-rw-r--r--  1 root                root                   5 Feb 13 14:58 docker.pid
srw-rw----  1 root                docker                 0 Feb 12 13:37 docker.sock
```

#### Group membership

##### (Regular) Local user

Does the group exist and are you a member?:

```shell
cat /etc/group | grep docker
```

##### LDAP users

LDAP users can make a GLPI ticket to request to be added to the docker group.

## Executing the Docker Command Without Sudo

Sometime Docker configures this for you, other times it does not!

If you are getting a message like the following Docker is probably not configured correctly:

```shell
Cannot connect to the Docker daemon. Is the docker daemon running on this host?.
See 'docker run --help'.
```

If you have root access via sudo you can correct this by adding your user to the docker group:

```shell
sudo usermod -aG docker ${USER}
```

To apply the new group membership, you can log out of the server and back in, or start a new session with:

```shell
su - ${USER}
```

You will be prompted to enter your user's password to continue.  Afterwards, you can confirm that your user is now added to the `docker` group by typing:

```shell
groups
```

OR

```shell
id -nG
```

## Confirmation and testing

```shell
docker --version
```

Output example:

```shell
Docker version 18.06.1-ce, build e68fc7a
```

Run the Hello world test

```shell
docker run hello-world
```

run an ubuntu container

```shell
docker run -it ubuntu bash
```

check the release:

```shell
cat /etc/lsb-release 
```

output example:

```shell
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=18.04
DISTRIB_CODENAME=bionic
DISTRIB_DESCRIPTION="Ubuntu 18.04.1 LTS"
```

exit

```shell
exit
```



list all images

```shell
docker ps -a
```

Output example:

```shell
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
a1b244c5f390        ubuntu              "bash"              3 minutes ago       Exited (0) 2 minutes ago                       clever_northcutt
7877c7d60c94        hello-world         "/hello"            4 minutes ago       Exited (0) 4 minutes ago                       lucid_ellis
```

Thing are working correctly. If you have problems after restarting your system recheck your the group setting on the `/var/run/docker.sock` file and the docker status using `systemctl status docker`.

