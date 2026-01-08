# piholes

[![CI](https://github.com/steffkelsey/piholes/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/steffkelsey/piholes/actions/workflows/ci.yml)

An Ansible playbook to install the software needed to run pihole DNS filtering off
of two raspberry pis.

Currently, my DNS hosts are a Rasperry Pi Zero 2 W and a secondary Raspberry Pi Zero.
Both are powered by Waveshare POE hats and housed in the Waveshare case.

This repo gives instructions on how to setup the hardware and stops after the 
basic software is installed.

- [x] pihole

additional packages  
 - [x] jq  
 - [x] vim  

## Setup Raspberry Pis

I decided to provision each pi the light Raspberry Pi OS because there is not
much memory on either of these boards (esp the Pi Zero).

I flashed Rasberry Pi OS (port of Debian Trixie) to the SD Card using there
Raspberry Pi Imager. The Zero still uses the 32bit version, 2 W on 64bit.

To make network discovery and integration easier, I edit the advanced
configuration in Imager, and set the following options:

  - Set hostname: `dns1.local` (set to `2` for secondary 2)
  - Enable SSH: 'Allow public-key', and paste in my public SSH key(s)

- [x] set static ip on each Pi

Raspberry Pi OS starts avahi by default, so when on the same network, you can
ssh into each using the username and hostname. Eg:  

```bash
ssh steff@dns1.local
```

## Setup Ansible

First, install Molecule, Ansible and the linters in a virtual env (not on a Pi,
on the computer you will use to SSH into the each Pi in the cluster).

```bash
# Setup using a virtual env
python3 -m venv .venv
# activate the env
source .venv/bin/activate
# (optional) update pip to the latest
pip3 install --upgrade pip
# Install everything from the requirements file
pip3 install -r requirements.txt

```

Second, install the dependencies for the main playbook and the roles.
```bash
ansible-galaxy install -r requirements.yml
```
in this directory


## SSH Connection Test

To be sure that Ansible can connect to each node, test with:
```bash
ansible all -m ping
```

## Running

To provision all the roles in this repo for the entire pi-cluster in hosts.ini, navigate
to the root directory of this repo and run:  
```bash
ansible-playbook main.yml --ask-become-pass
```

To upgrade all the software installed with apt, run   
```bash
ansible-playbook upgrade.yml --ask-become-pass
```

To set the pihole passwords on the primary and secondary DNS, run   
```bash
DNS1_PWD="<password_1>" DNS2_PWD="<password_2>" ansible-playbook setpassword.yml --ask-become-pass
```

## Tests

To test the main playbook, go to the top level directory and run:  
```bash
molecule test
```

## FYI 
I had trouble testing the playbook with molecule from a host running Rocky
Linux 9. The issue is with older docker containers needing the host to have
ip_tables present. Info
[here](https://ryandaniels.ca/blog/docker-and-trouble-with-red-hat-enterprise-linux-9-iptables/).

To see the error, login to a running container and try `dockerd`. You should
see an error about ip_tables not being present. In that case, use the fix from
the link above.

## Debugging

To debug the playbook, from the project root:  

1. start the dev environment `$ molecule converge`  
2. ssh into the container `$ molecule login`  
3. rerun the playbook `$ molecule converge`  
4. run the linter `$ molecule lint`  
5. run the verify steps `$ molecule verify`  
6. clean up `$ molecule destroy`  
7. testing - see the Tests section above for running the entire cycle which
includes creating, verifying, and destroying the environment

