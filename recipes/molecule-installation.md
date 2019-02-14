# molecule-installation-and-setup.md

## Requirements

### Python 2.7

#### On OS X



#### On Ubuntu

Confirm Python 2.7 is installed:

```shell
sudo apt install python-minimal
which python
python --version
```

### pip

To install `pip`:

```shell
sudo apt install python-pip
```

### Python Environment Creation

We can get a list of all pip installable versions of molecule using the following `pip install` command:

```shell
pip install molecule==
```

Output example:

```shell
  Could not find a version that satisfies the requirement molecule== (from versions: 1.20.1, 1.20.3, 1.21.1, 1.22.0, 1.23.0, 1.25.0, 1.25.1, 2.10.0, 2.10.1, 2.11.0, 2.12.0, 2.12.1, 2.13.0, 2.13.1, 2.14.0, 2.15.0, 2.16.0, 2.17.0, 2.18.0, 2.18.1, 2.19.0)
No matching distribution found for molecule==

```

In this case we will create an environment to hold the latest available version of Molecule, **2.19.0**, by adding the version to our environments path. *In addition, here we will use the **--no-site-packages** flag to ensure that our python environment is created without copying any local python libraries, ensuring that we have a fresh installation*. Note:  The **--no-site-packages** flag has been depreciated as not having access to global site-packages is now the default behavior. It will not hurt anything and if you are using an older OS/Python/virtualenv installed it can prevent mysterious problems.

Create your environment

```shell
virtualenv ~/.venv/molecule/2.19.0 --no-site-packages --python=python2.7
```

Activate the environment

```shell
source ~/.venv/molecule/2.19.0/bin/activate
```

Install Molecule (all dependencies, including Ansible, will be installed to your new virtual environment as well: 

```shell
pip install 'molecule==2.19.0'
```

Confirming your installation:

```shell
which python
which molecule
which ansible
```



## Initialize a new role using molecule

Ensure for your projects directory

```shell
mkdir ~/projects
cd ~/projects
```

Initialize a new role using Molecule

```shell
molecule init role -r my_role
```

By default Molecule will also create a default scenario that uses Docker as the provider. Any time a scenario is created it will also create an `INSTALL.rst` file in the scenario's directory. In this case the default scenario is in a sub directory of the molecule directory called `molecule/default` 

To determine any additional requirements needed to use Docker as your virtualzation provider you can take a look at the scenarios `INSTALL.rst` file:

```shell
cat molecule/default/INSTALL.rst
```

Output example:

```shell
*******
Docker driver installation guide
*******

Requirements
============

* General molecule dependencies (see https://molecule.readthedocs.io/en/latest/installation.html)
* Docker Engine
* docker-py
* docker

Install
=======

    $ sudo pip install docker-py
```

Since we are using a virtual environment in our users directory we **do not need sudo** in order to install `docker-py` to ur virtual environment. As a bonus, since we are doing this as a regualr user, none of the changes we make to our virtual environment will affect the rest of our system nor any other Python environments.

```shell
pip install docker-py
```

