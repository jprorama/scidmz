Ansible node config and management for a ScienceDMZ

The ScienceDMZ cluster typically contains a perfSONAR node, bro node, and one or
more Data Transfer Nodes (DTNs) for Globus Online file transfer (ie. GridFTP).

There are various playbooks to help simplify setup.  Create a site specific
site_vars.yaml and hosts file and then most playbooks can be run with a simple

```sh
ansible_playbook -i hosts playbook_name.yaml
```

Copy the example-site_vars.yaml and example-hosts and customize for your
ScienceDMZ cluster

```sh
cp example-site_vars.yaml site_vars.yaml
cp example-hosts hosts
```

# Common Configuration

Base configurations that apply across all hosts in the site.

## Ansible helper

Ansible is easiest when you can sudo without password prompts. Copy the example
config and an appropriate user account.  The "zz_" prefix ensures that the rules
are very likely the last ones and therefore highest priority.

```sh
cd config/sudoers/
cp example-zz-ansible-admin zz-ansible-admin
```

Then run the playbook
```sh
ansible_playbook -i hosts set_admins.yaml
```

You'll only need to run this once and maybe never, but it's here for completeness.

## Timezone

Your host should agree on time and timezone.  Set the timezone in the site_vars.yaml
and get time setup and in place with:

```sh
ansible-playbook -i hosts config_ntp.yaml
```

Note perfSONAR sets up ntp install so this host is not changed.

# DTN Configuration

## Update firewall

The GridFTP requires ports open for access to control channels to initiate transfers
and data channels to move data between sites.  The control channel is managed
by globusonline.org.  The rule sets add permissions for globusonline to initiate
third-party transfers and allow data to flow to any site.

The configuration uses [firewalld zones](https://www.hogarthuk.com/?q=node/9) to
control source IP restrictions from globusonline.org.  The on-disk firewall is
updated and then reloaded into the current state.

##  Apply the Globus On-line configuration

The [Globus configuration](https://docs.globus.org/resource-provider-guide/#install_section)
installs globus-connect-server metapackage and sets up GridFTP endpoints for use
on globus.org.   It is derived from the
[XSEDE Campus Bridging project's playbook](https://software.xsede.org/cb/centos6/noarch/extras/XCBC_Ansible/globus_playbook.yml)
distribed as part of a ROCKS Roll for an XSEDE Compatible Basic Cluster.   It's
adjusted to apply to the "ScienceDMZ cluster" model implemented in this project.
It also prefers CILogon integration.

```sh
ansible-playbook -i hosts globus_playbook.yml
```

Note, the final playbook step registers the endpoints with globus.org.
This required a pexpect package installed on the DTNs.
The pexpect version 2.3 included with CentOS7 is less than the reported
compatible 3.3+ required by the expect module.  This step can be run by hand if
if required dependency is not met.
