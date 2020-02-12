Ansible Inventories

Ansible Inventories are complex in the following ways:
• After you have decided an Inventory model is hard to change it, in fact you would probably be required to touch all your Playbooks to make everything working again
• There is not a single way of doing Inventories
• Often Inventories are the glue to make a generic Playbook run properly on your specific architecture.
Grouping philosophies
The two main philosophies I saw in the many years I’ve worked with Ansible are:
• Create super specific groups so that all Ansible Playbook run will run against all nodes of that group
• Create fairly broad groups and then at run time use group intersections to identify exactly which hosts to target
In inventories that follow the first philosophy we would be able to find group names like:
• application1-dev-appserver
• application2-uat-dc1-database
• website4-blueprod-dc2-appserver
• US-application3-greenprod-dc2-appserver
In inventories that follow the second philosophy we would be able to find group names like:
• US
• DC1
• production
• application1
• appserver
As you can see, in the second philosophy the names are not different from the ones of the first philosophy, but the real difference is that in the first case you have more names in the same group name.
My philosophy
From my point of view, the second philosophy is often the suggested one, since the problem with the first one is that if you have a group name composed by 5 characteristics, in a very short time you’ll find yourself managing hundred of groups.
Now, which groups are sensible to make and which are not is a completely different and more complex question. My general rules on this are:
• If the company has multiple regions, one group per region is reasonable (ex: US, UK)
• If there are multiple datacenters per region, a datacenter based grouping is reasonable (ex: DC1, DC22)
• If both regions and datacenters are available, could be sensible to put them both in the same collection (ex: US1, US2, UK1, UK2)
• It usually makes sense to have a collection of groups based on the environments (ex: dev, int, uat, qa, preprod, prod, blue, green)
• It usually makes sense to have a collection of groups based on the application (ex: applicationX, websiteY)
• It usually makes sense to have a collection of groups based on the role of the server (ex: loadbalancer, frontend, appserver, database, storage)
Clearly this list is not complete and often speaking with the operations teams they have point out additional needs that need to be addressed.
Be productive using Patterns
If you look the list of collections of groups that we have just discussed, you can notice that they will split your machines in different directions. This is important because Ansible is able to execute operations between groups to choose which modes will be targeted for a specific Playbook thanks to the Patterns.
We can two different kind of Patterns based on the number of parameters they have:
• Patterns with one parameter
• Patterns with two parameter
Patterns with one parameter
Patterns with one parameters are simpler but still very powerful. The main we have are:
• all: Run against all hosts that are listed in the inventory no matter of the groups you have configured
• *: Run against all hosts that are listed in the inventory no matter of the groups you have configured
• 192.168.1.: Run against all hosts that are listed in the inventory no matter of the groups you have configured
• ws[00-99].example.com: Run against all hosts from ws00.example.com until ws99.example.com
Other patterns that respect the same principles are usable as well.
Patterns with two parameters
Ansible Patterns allow us to also do operations between two different groups, so we can do:
```• gr1:gr2: Putting a single column Ansible will use the OR operator, so it will select all nodes that appear in at least one of the listed groups (group union)
• gr1:&gr2: Putting a column and an amp Ansible will use the AND operator, so it will select all nodes that appear in both groups (group intersection)
• gr1:!gr2: Putting a column and an exclamation point, Ansible will use the AND NOT operator, so it will select all nodes that appear in the first group but not in the second```

### Complex Patterns

You can also refine your selection, so for instance you could be using
loadbalancers:frontend:&US:!prod
In this case, Ansible is going to run the Playbook on all loadbalancers and fronternd servers in the US that are not in production.
This allows a pretty big level of detail, as you can see, in fact in the end you have a granularity level comparable to the first philosophy granularity, but without the need of painful maintenance.
As you may immagine, you can also mix patterns with one and two parameters like:
*.example.com:frontend:&US:!prod
In this case, Ansible is going to run the Playbook on all machines that appear under the .example.com in the inventory file, and frontend servers in the US that are not in production.
Regular Expressions
Sometimes simple patterns are not enough, and you want to leverage the power of Regular Expressions. This is possible in Ansible and you can do it specifying a ~ at the begin of the string:
This will match all hosts whose inventory name matches the Regular Expression, such as ws01.example com and db02.example.com.
Select groups at runtime
Sometimes the problem is that we want to create some Playbook that are generic and that can be used with groups chosen at runtime.
In Ansible, you can pass variables instead of explicit group names. So, if we want to create a Playbook that does a specific action on all machines that are hosting a specific application in a specific environment, we can simply write:
'{{ application }}:&{{ environment }}'
Now, setting the proper values for application and environment, we can run this Playbook on any applications and environments.
Leverage limits
Ansible allows us to limit the set of hosts that we are going to target using the --limit option.
You can use the limit option in the following ways:
• With a hostname: for example, --limit ws01.example.com will only run the Playbook on that specific host (if it is in the set of hosts where that Playbook with that inventory file can run)
• With a group: for example, --limit DC2 will behave in the same way of adding :&DC2 to your Playbook hosts directive
• With a file: for example, --limit @retry_hosts.txt will behave in a similar way of the hostname one, but based on the content of the file.
