# Deploy Kubernetes with ansible on Atomic

 I’ve been playing with Project Atomic as a platform to run Docker containers for some time now. The reason I like Project Atomic is something for another blogpost. One of the reasons however, is that while it’s a minimal OS, it does come with Python so I can use Ansible to do orchestration and configuration management. Now, running Docker containers on a single host is nice, but the real fun starts when you can run containers spread over a number of hosts. This is easier said than done and requires some extra services like a scheduler, service discovery, overlay networking,… There are several solutions, but one that I particularly like is Kubernetes. ProjectAtomic happens to ship with all necessary pieces needed to deploy a Kubernetes cluster using Flannel for the overlay networking. The only thing left is the configuration. Now this happens to be something Ansible is particularry good at. The following wil describe how you can deploy a 4 node cluster on top of Atomic hosts using Ansible. Let’s start with the Ansible inventory. inventory We will keep things simple here by using a single file-based inventory file where we explicitly specify the ip adresses of the hosts for testing purposes. The important part here are the 2 groups k8s-nodes and k8s-master. The k8s-master group should contain only one host which will become the cluster manager. All nodes under k8s-nodes will become nodes to run containers on.

\[k8s-nodes\] atomic02 ansible\_ssh\_host=10.0.0.2 atomic03 ansible\_ssh\_host=10.0.0.3 atomic04 ansible\_ssh\_host=10.0.0.4

\[k8s-master\] atomic01 ansible\_ssh\_host=10.0.0.1

Variables Currently these roles don’t have many variables that can be configured but we need to provide the variables for the k8s-nodes group. Create a folder group\_vars with a file that has the same name of the group. If you checked out the repository you already have it.

$ tree group\_vars/ group\_vars/ k8s-nodes

The file should have following variables defined.

skydns\_enable: true

IP address of the DNS server. Kubernetes will create a pod with several containers, serving as the DNS server and expose it under this IP address. The IP address must be from the range specified as kubeserviceaddresses. And this is the IP address you should use as address of the DNS server in your containers.

dns\_server: 10.254.0.10

dns\_domain: kubernetes.local

Playbook Now that we have our inventory we can create our playbook. First we configure the k8s master node. Once this is configured we can configure the k8s nodes.

name: Deploy k8s Master hosts: k8s-master remote\_user: centos become: true roles: k8s-master

name: Deploy k8s Nodes hosts: k8s-nodes remote\_user: centos become: true roles: k8s-nodes

Run the playbook.

ansible-playbook -i hosts deploy\_k8s.yml

If all ran without errors you should have your kubernetes cluster running. Lets see if we can connect to it. You will need kubectl. On Fedora you can install the kubernetes-client package.

$ kubectl —server=192.168.124.40:8080 get nodes NAME STATUS AGE 192.168.124.166 Ready 20s 192.168.124.55 Ready 20s 192.168.124.62 Ready 19s

That looks good. Lets see if we can run a container on this cluster.

$ kubectl —server=192.168.124.40:8080 run nginx —image=nginx replicationcontroller “nginx” created

Check the status:

$ kubectl —server=192.168.124.40:8080 get pods NAME READY STATUS RESTARTS AGE nginx-ri1dq 0/1 Pending 0 55s

If you see the pod status in state pending, just wait a few moments. If this is the first time you run the nginx container image, it needs to be downloaded first which can take some time. Once your pod is is running you can try to enter the container.

kubectl —server=192.168.124.40:8080 exec -ti nginx-ri1dq — bash root@nginx-ri1dq:/\#

This a rather basic setup \(no HA masters, no auth, etc..\). The idea is to improve these Ansible roles and add more advanced configuration. If you are interested and want to try it out yourself you can find the source here: https:gitlab.comvincentvdkansible-k8s-atomic.git

