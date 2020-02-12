# ansible-without-inventory

Ansible is a great tool to automate almost anything in IT. However, one of the core concepts of Ansible is the inventory where the to be managed nodes are listed. However, in some situations setting up a dedicated inventory is overkill.

For example there are many situation where admins just want to ssh to a machine or two to figure something out. Ansible modules can often make such SSH calls in a much more efficient way, making them unnecessary – but creating a inventory first is a waste of time for such short tasks. In such cases it is handy to call Ansible or Ansible playbooks without an inventory. In case of plain Ansible this can be done by addressing all nodes while at the same time limiting them to an actual hostslist:

```text
$ ansible all -i jenkins.qxyz.de, -m wait_for -a "host=jenkins.qxyz.de port=8080"
jenkins.qxyz.de | SUCCESS => {
"changed": false, 
"elapsed": 0, 
"path": null, 
"port": 8080, 
"search_regex": null, 
"state": "started"
}
```

The comma is needed since Ansible expects a list of hosts – and a list of one host still needs the comma. For Ansible playbooks the syntax is slightly different: \`$ ansible-playbook -i neo Here the “all” is missing since the playbook already contains a hosts directive. But the comma still needs to be there to mark a list of hosts.

