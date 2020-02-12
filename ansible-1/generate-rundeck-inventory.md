# Generate Rundeck Inventory

Ansible – Generate Rundeck Inventory

I have been doing some testing with Jenkins and Rundeck along with Ansible lately. I wanted a way to number one not have to manually update resources.xml for Rundeck but my biggest desire was to have Rundeck populated with the same hosts that Ansible was managing. vars: \(external in group\_vars/all/configs\) rundeck\_generate\_resources\_xml: true \#will rebuild resources.xml and prepare for delivery to your rundeck\_server rundeck\_resources\_xml\_file: resources.xml rundeck\_resources\_xml\_location: /var/lib/rundeck/var rundeck\_server: ci-pipeline

## Role Task: \(generate\_rundeck\_resources.yml\)

name: identifying hosts action: ping register: id\_hosts

name: checking for exsisting resources.xml stat: path=‘/’ delegate\_to: ‘’ register: resources\_xml run\_once: true

name: creating resources.xml template: src=‘.j2’ dest=‘/’ owner=rundeck group=rundeck mode=0755 delegate\_to: ‘’ run\_once: true when: not resources\_xml.stat.exists

name: creating rundeck resources.xml \(not rundeck\_server\) lineinfile: dest=“/” line=“” insertbefore=“^” state=present delegate\_to: ‘’ with\_items: id\_hosts.results when: \(inventory\_hostname != rundeck\_server\)

name: creating rundeck resources.xml \(rundeck\_server\) lineinfile: dest=“/” line=“” insertbefore=“^” state=present delegate\_to: ‘’ with\_items: id\_hosts.results when: \(inventory\_hostname == rundeck\_server\)

name: cleaning up tags in resources.xml shell: “sed -i -e \”s/\[\]\[\]//g\” /” delegate\_to: ‘’ run\_once: true

name: cleaning up resources.xml shell: “sed -i -e \”s/‘//g\” /” delegate\_to: ‘’ run\_once: true

name: ensuring resources.xml is still owned by rundeck user file: path=“/” owner=rundeck group=rundeck mode=0755 delegate\_to: ‘’ run\_once: true

