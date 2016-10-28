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

