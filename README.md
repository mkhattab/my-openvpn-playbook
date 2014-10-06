My OpenVPN Server Playbook
==========================

This repository contains my playbook for creating and managing my
OpenVPN server. It's currently a basic setup if the playbook is run
with minimal modifications. Out of the box, the playbook sets up an
OpenVPN server with a subnet of **172.16.1.0/24**, **NAT** and a
*minimal* firewall.

Prerequisites
-------------

[Ansible](http://www.ansible.com) and a
[Digital Ocean](https://www.digitalocean.com) droplet/server.

Installing Ansible and DoPy (Digital Ocean API library):

```bash
pip install ansible dopy
```

Also, set the DO API keys:

```bash
export DO_CLIENT_ID='client_id'
export DO_API_KEY='long_api_key'
```

Directory Structure
-------------------

This repo is organized according to the following:

* client-config/ --- per-client server configuration rsync'd to the
  server
* client-keys/ --- client keys and configuration zipped & fetched from
  the server
* files/ --- configuration files templates
* vars/ --- server, client and CA variable files
* digital_ocean.py --- inventory script
* ansible.cfg --- local Ansible config
* handlers.yml --- restart/reload service handlers
* server.yml --- server playbook
* client.yml --- client playbook


Server Playbook
---------------

The server playbook will install the required packages for OpenVPN and
create the root CA and server certificates. Run the playbook using
the following commands, but before you do, please modify `files/ca.yml` with
values you want.

```bash
ansible-playbook --extra-vars 'target=[target host] common_name=[CA Name]' server.yml
```

Where `target` is the host(s) which you would like the playbook to be
executed. To see a list of hosts, run this: `./digital_ocean.py --list`

And where `common_name` is the certificate common name you'd like for the root
certificate authority (CA).

Modifying or changing server variables (i.e. `vars/server.yml`) or the
`files/server.conf.j2` template will require re-running the playbook.
However, not all the tasks need to be run, so it's much faster to use
the following command:

```bash
ansible-playbook --extra-vars 'target=[host]' server.yml --tags conf
```

Which will run the tasks tagged `conf` and skip the rest.

Modifying the iptables rules template (`files/rules.v4.j2`) also
requires the re-running playbook using the `firewall` tag.

```bash
ansible-playbook --extra-vars 'target=[host]' server.yml --tags firewall
```


Client Playbook
---------------

The client playbook can be used to generate client certificates and
fetch zipped client keys and configurations. Which can then be given
to clients without modification. This playbook can also sync
per-client server configuration to the server from the directory
`client-config/`. See the
[OpenVPN documentation](http://openvpn.net/index.php/open-source/documentation/howto.html#policy)
or the
[manual](https://community.openvpn.net/openvpn/wiki/Openvpn23ManPage)
on **--client-config-dir** option for more info.

To run the client playbook:

```bash
ansible-playbook --extra-vars 'target=[host] common_name=[client]' client.yml --tags new-client
```

Where `common_name` is the `CN` given to the client certificate and
the zip file that is fetched from the server.

To synchronize the `client-config/` directory, run the following:

```bash
ansible-playbook --extra-vars 'target=[host]' client.yml --tags sync-ccd
```
