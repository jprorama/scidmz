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
