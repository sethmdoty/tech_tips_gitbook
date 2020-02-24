# Dockerize a Live System

 I need to run some ansible playbooks to a running \(live\) machine. But, of-course, I can’t use a production server for testing purposes !! So here comes docker! I have ssh access from my docker-server to this production server:

`[docker-server] ssh livebox tar -c / | docker import - centos6:livebox`

Then run the new docker image:

`[docker-server] docker run -t -i —rm -p 2222:22 centos6:livebox bash`

`[root@40b2bab2f306 /]# /usr/sbin/sshd -D`

Create a new entry on your hosts inventory file, that uses ssh port 2222 or create a new separated inventory file and test it with ansible ping module:

`ansible -m ping -i hosts.docker dockerlivebox`

`dockerlivebox | success >> { “changed”: false, “ping”: “pong” }`

