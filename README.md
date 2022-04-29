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

# Globus v5 endpoints

In order to use the `globus_5_playbook.yml` playbook, you must register an endpoint at [the globus development portal](https://auth.globus.org/v2/web/developers) and use the associated client id/secret variables in group vars.

## Setting up box connector

Once globus v5 is set up and deployed, you can create a collection that supports box with the following commands:

```bash
globus-connect-server storage-gateway create box ${GATEWAY_NAME} --high-assurance --domain ${DOMAIN_SUFFIX} --authentication-timeout-mins ${TIMEOUT_IN_MINUTES} --box-settings file:${BOX_JSON_FILE}
```

With the following variables:

* `$GATEWAY_NAME` - a short name to refer to the gateway by; in my testing, I had named it `ha-box-connector`
* `${DOMAIN_SUFFIX}` - the domain you want to scope the gateway to; this is only a hard requirement with `--high-assurance`
* `${TIMEOUT_IN_MINUTES}` - the web UI timeout for inactivity, in minutes; this is only a hard requirement with `--high-assurance`
* `${BOX_JSON_FILE}` - a path to the box json file. You should be able to get a json file out of your box apps (found [here](https://uab.app.box.com/developers/console))

This command will give a UUID to use with the next step.

```bash
globus-connect-server collection create ${GATEWAY_UUID} ${BOX_PATH} "${COLLECTION_NAME}"
```

With the following variables

* `${GATEWAY_UUID}` - the UUID from the previous command.
* `${BOX_PATH}` - The path to "scope" within box. For example, if you set the path as `/box`, it will only allow users to transfer files out of their box account if they had a folder in their root *named* `box`. If you set this as `/`, it will allow access to the user's entire box folder
* `${COLLECTION_NAME}` - A friendly human-readable string; this will be searchable through the globus web interface.
