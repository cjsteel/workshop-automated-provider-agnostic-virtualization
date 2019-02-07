# NADM Workshop - Automated Provider Agnostic Virtualization

NW142 - Thursday, February 14th, 11:00 - 12:00, 12:30 Max

## Demo's and Discussions

### [Molecule](https://molecule.readthedocs.io/en/latest/) Demo and Discussion - John Le

Time estimate 15-30 min depending on discussion / questions
Providers demoed : Docker, Vagrant/VirtualBox

* Creating a python virtualenv (sandbox)
* Installing Molecule into the virtualenv
* Cloning of an Ansible role to install minc-tools
* Initialising the mic-tools role for usage with Molecule 
* Automated the creation and configuration of one or more provider (Docker and Vagrant/VirtualBox) using Molecule scenarios and the minc-tools Ansible role.

### Using [LXC/LXD](https://linuxcontainers.org/lxd/introduction/) as a Provider for Development - Christopher Steel

Time estimate 15-30 min depending on discussion / questions
Providers: LXC/LXD on ZFS as a file

* Leveraging Copy on Write (COW) via ZFS as a file.
* Sharing images via a single or or multiple clustered LXD image server(s).
* Molecule Demo

### Hands On Workshop

Time estimate 30min - 1hour depending on previous discussions

* Running the demo's on your notebook or workstation
* Discussions about  potential uses in development and research

#### Requirements

(In order to run the demos you will want a notebook or workstation running one or more of the virtualization providers)

##### Python

* pip
* virtualenv

##### Virtualization providers

* Docker
* LXC/LXD
* Vagrant
* VirtualBox