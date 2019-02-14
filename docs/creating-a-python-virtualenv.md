# creating-a-python-virtualenv.md

## Requirements

* Pythony futile  2.7.x
* pip
* virtualenv

### Installing the requirements

* [installing-python-and-requirements-on-ubuntu-18.04](installing-python-and-requirements-on-ubuntu-18.04.md)

## Creating your virtual environment

We are dong this in a very specific way in order to avoid potential issues and to allow for the creation of new environments for different versions of molecule. It's a little more complicated and certainly not required but helpful for troubleshooting and avoiding "Python Hell" frequently associated with older OS's that shipped with funky Python installations...

In addition we are using a speciic directory name for our python environments so that if you decide to create your python environment in your automation directory molecule will not scan your python environment in order to "delint" it when running the `molecule test` command a time consuming and futile process ;-).

### Determine what the latest version of Molecule is

This a trick to get the latest version of molecule. We **do not** want to install Molecule now

```shell
pip install molecule==
```

Output example:

```shell
  Could not find a version that satisfies the requirement molecule== (from versions: 1.20.1, 1.20.3, 1.21.1, 1.22.0, 1.23.0, 1.25.0, 1.25.1, 2.10.0, 2.10.1, 2.11.0, 2.12.0, 2.12.1, 2.13.0, 2.13.1, 2.14.0, 2.15.0, 2.16.0, 2.17.0, 2.18.0, 2.18.1, 2.19.0)
No matching distribution found for molecule==
```

So here our latest version is 2.19.0

### Create your python environment

```shell
virtualenv ~/.venv/molecule/2.19.0 python==python2.7 --no-site-packages
```

### Activate it

```shell
source ~/.venv/molecule/2.19.0/bin/activate
```

### Install molecule

```shell
pip install molecule
```

### Confirm your installation

```shell
which molecule
```
Output example:

```shell
/home/csteel/.venv/molecule/2.19.0/bin/molecule
```

That's it.