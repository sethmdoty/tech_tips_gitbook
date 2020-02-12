# Zero Downtime Upgrades with Openshift Ansible

OpenShift Ansible’s upgrade process has been designed to leverage the HA capabilities of OpenShift and allow for performing a complete cluster upgrade, without any application outages. Doing so is heavily dependent on the nature of your application as well as the capacity of your cluster. However, this post will cover how we perform upgrades, and demonstrate one without causing downtime for a sample application. How Openshift Ansible Performs Upgrades The basic steps for an openshift-ansible upgrade are as follows: 1. Pre-Flight Checks 2. Validates the state of your cluster before making any changes. 3. Checks that inventory variables are correct, your control plane is running, relevant rpms/containers are available, and the required version of Docker is either installed or available. 4. Runs in parallel on all hosts. 5. Can be run by itself by specifying --tags pre\_upgrade with your ansible-playbook command.

* Control Plane Upgrade
* Create a timestamped etcd backup in parallel on all hosts.
* Upgrade OpenShift master containers/rpms on all masters in parallel. \(no service restart\)
* Restart master services, performed serially one master at a time. This can be configured to restart the entire host system if desired.
* Because this is performed serially, a load balanced control plane should see no downtime during this upgrade, however we’re mostly interested in application downtime in this scenario.
* Reconcile cluster roles, role bindings, and SCCs.
* Upgrade the default router and registry.
* Node Upgrade
* This entire process is run serially on one node at a time by default.
* Can be configured to run in parallel for a set number or percentage of nodes as of 1.4/3.4 with: -e openshift\_upgrade\_nodes\_serial="5"
* Can be configured to run only on nodes with a specific label as of 1.4/3.4 with: -e openshift\_upgrade\_nodes\_label="region=na"
* For our purposes we will stick to the default and only upgrade one node at a time
* Evacuate the node and mark it unschedulable.
* Upgrade Docker if necessary. \(NOTE: With OpenShift all masters must run the node service as well, so this process covers upgrading Docker there as well\)
* Update node configuration and apply any necessary migrations.
* Stop all OpenShift services.
* Restart Docker.
* Start all OpenShift services.
* Mark node schedulable again.
* Wait for node to report Ready state before proceeding to the next node.

  A Zero Downtime Upgrade Example

  For the above upgrade process to result in no application downtime, we need the node upgrade phase to not take down so many nodes that we do not have capacity for our application to remain running. We also similarly need to ensure the router remains running on at least one of the infra nodes during node upgrade, and when we actually upgrade the router itself.

  For this upgrade we’ll be using a total of 13 AWS systems:

* 3 master+etcd hosts
* 3 infra nodes, 2 in zone ‘east’ and 1 in zone ‘west’ \(NOTE: important caveat here, the OpenShift router runs on each of these, and 2 may not always be enough for reasons explained below\)
* 3 regular nodes in zone ‘east’
* 3 regular nodes in zone ‘west’
* 1 load balancer running haproxy for both the API server, and our sample application

  The ansible inventory file used to create the cluster can be viewed here.

  In testing, if there were only two infra nodes and they were listed consecutively in the inventory, we did see that it was possible to have no running routers for a brief period of time. When the first infra node is being upgraded it has been evacuated leaving us with only one router running. Once the first node upgrade completes we mark it schedulable again and then evacuate the second node. However this could occur before Kubernetes has rescheduled the router back onto the first node, leaving no routers at all.

  For this reason we utilized three infra nodes for this demonstration. In theory two should suffice if you were to control the ordering in your inventory during the upgrade.

  Setup

  For this test I performed a clean installation of OpenShift Enterprise 3.2, however it may be worth noting I did so using a more recent version of openshift-ansible targeted for 1.4/3.4. This allows me to benefit from automatic deployment of the router on all infra nodes, and the latest upgrade work, so results here should be valid for 1.3/3.3 -&gt; 1.4/3.4 upgrades. However, if you use an older openshift-ansible version your experience may differ.

  The lb host in the inventory above causes openshift-ansible to configure a HAProxy service on that host to load balance requests to the API. I then modified / etc / haproxy /haproxy.cfg on that host to also load balance requests to my infra node openshiftip’s where the router will be running.

frontend helloworld bind \*:80 default\_backend helloworld-backend mode tcp option tcplog backend helloworld-backend balance source mode tcp server node0 172.18.5.105:80 check server node1 172.18.5.102:80 check Sample Application My sample application is a simple template using the hello-openshift container: $ cat ha-upgrade-app.yaml kind: Template apiVersion: v1 metadata: name: "ha-helloworld-template" objects: kind: "DeploymentConfig" apiVersion: "v1" metadata: name: "frontend" spec: template: metadata: labels: name: "ha-helloworld" spec: containers: - name: "ha-helloworld" image: "openshift/hello-openshift" ports: - containerPort: 8080 protocol: "TCP" replicas: 4 strategy: type: "Rolling" paused: false minReadySeconds: 0 kind: Service apiVersion: v1 metadata: name: "helloworld-svc" labels: name: "ha-helloworld" spec: type: ClusterIP selector: name: "ha-helloworld" ports: - protocol: TCP port: 8080 targetPort: 8080 kind: Route apiVersion: v1 metadata: name: helloworld-route spec: host: www.example.com to: kind: Service name: helloworld-svc This container just returns a static ‘Hello OpenShift!’ response for each request. In the real world the success of this upgrade would depend heavily on the nature of your application, particularly in relation to databases and storage. The template requests four replicas, which will spread out evenly across our two zones by default, leaving one free node in each zone. $ oc new-project helloworld $ oc new-app ha-upgrade-app.yaml Monitoring We use a fake www.example.com route in our template above, so whatever host we are going to monitor the application and control plane from needs to have an / etc /hosts entry pointing www.example.com to the load balancer public IP. To monitor the application I logged responses from a request every second with a timestamp: $ while :; do curl -s www.example.com -w "status %{http\_code} %{size\_download}\n" \| ts; sleep 1; done To monitor the control plane and log what pods were running every second, I ran a similar command on my master to list all pods in all namespaces: $ while :; do oc get pods --all-namespaces \| ts; sleep 1; done Performing the Upgrade To upgrade we just modify our inventory to set openshift\_release=v3.3, and flip our yum repositories from 3.2 to 3.3: $ ansible OSEv3:children -i ./hosts -a "subscription-manager repos --disable rhel-7-server-ose-3.2-rpms --enable rhel-7-server-ose-3.3-rpms" We are now ready to upgrade: $ ansible-playbook -i ./hosts playbooks/byo/openshift-cluster/upgrades/v3\_3/upgrade.yml Results Logs from this upgrade are available here:

* Ansible Upgrade Log
* Hello World Request Log
* Pod Status Request Log

  The upgrade took 32 minutes. \(amount of time would vary depending on size of your cluster, network speed, and system resources\)

  Six individual requests to our application failed throughout the upgrade at separate points in time, all of which were related to restarts of the router pods on our infra nodes.

  In the logs you will see three with status 000, caused by the load balancer not yet realizing the infra node is not healthy. The other three appear with status 503 where the router responds, but does not yet know about our route, a known issue being tracked here. All six were resolved within 1-2s.

  A future enhancement is planned to support providing hooks which users could leverage to gracefully take infra nodes out of rotation on their load balancer of choice before we evacuate the node, and restore it after we have marked it schedulable again. Combined with a fix for the brief router restart issue above this should eliminate all failed requests we saw above.

  Our actual application however, remained running throughout the upgrade. While a few requests may fail when the load balancer is detecting an infra node being down, other requests will continue to succeed if they land on the other routers. Because we only upgrade one node at a time, there will reliably be 2-3 other replicas of our application up and responding.

  Zero downtime upgrades are possible with openshift-ansible, provided your application is capable, and your cluster is highly available with sufficient capacity.

