### LXD

#### Add remote

##### Firewall on remote image server

Shorewall entry example:

```shell
ACCEPT  net:132.216.122.0/24    $FW             tcp     8443    #McGill AceLabs NW LXD
ACCEPT  net:142.157.139.0/24    $FW             tcp     8443    #McGill AceLabs NW LXD
```

##### Adding a remote to new client

```shell
lxc remote add ace-ws-101 ace-ws-101.cbrain.mcgill.ca:8443 --password=password_here
```

Output:

```shell
Certificate fingerprint: f154c98ab05932474a3bb483a4226f18f0316396cff7d818b494be9500cb8e62
ok (y/n)? y
```

Output:

```shell
Client certificate stored at server:  ace-ws-101
```

##### Testing  on new client

List available images

```shell
lxc image list ace-ws-101:
```

Output example:

```shell
+-----------------+--------------+--------+---------------------------------------------+--------+----------+-----------------------------+
|      ALIAS      | FINGERPRINT  | PUBLIC |                 DESCRIPTION                 |  ARCH  |   SIZE   |         UPLOAD DATE         |
+-----------------+--------------+--------+---------------------------------------------+--------+----------+-----------------------------+
| 18.04           | b7c4dbea897f | no     | ubuntu 18.04 LTS amd64 (release) (20190131) | x86_64 | 175.03MB | Feb 4, 2019 at 7:56pm (UTC) |
+-----------------+--------------+--------+---------------------------------------------+--------+----------+-----------------------------+
| ubuntu (1 more) | 8ef652f60666 | no     |                                             | x86_64 | 293.14MB | Feb 4, 2019 at 8:15pm (UTC) |
+-----------------+--------------+--------+---------------------------------------------+--------+----------+-----------------------------+
|                 | ee33f784ee1f | no     | Centos 7 amd64 (20190205_07:09)             | x86_64 | 124.86MB | Feb 5, 2019 at 1:44pm (UTC) |
+-----------------+--------------+--------+---------------------------------------------+--------+----------+-----------------------------+
```

Launch remote image locally as a container

```shell
lxc launch ace-ws-101:18.04 c1 --alias bionic
```

* https://help.ubuntu.com/lts/serverguide/lxd.html.en
* https://help.ubuntu.com/lts/serverguide/lxc.html.en
* [LXC: Linux container tools](https://developer.ibm.com/tutorials/l-lxc-containers/?mhq=%22LXC%3A%20Linux%20container%20tools%22%20%28developerWorks%2C%20February%202009%29.&mhsrc=ibmsearch_a#ssh)
