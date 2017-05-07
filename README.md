# Zooz SRE Technical Task - Avraham Kalvo

#### INTRO
Welcome to my solution for the Zooz technical task. As per task outline, I've implemented a cassandra ring and repair task using ansbile and docker. Getting into the world of the two latter, as approaches to configuration and deployment was an enriching experience for me.

#### Solution Specification
1. zooz.technical-task (site.yml): A Docker install and cassandra node(s) creation in a ring ansible playbook, as well as automatically repair the created node(s)
1. zooz_cassandra: An Ansible role which implements docker installation and cassnadra node creation
1. zooz_cassandra_repair: An Ansible role which implements cassandra over docker node repair
1. cluster_create.sh, node_repair.sh: a couple of shell scripts to issue each of the above Ansible roles seperately

#### Considerations & Guidelines
This solution is built to support a multi node cassandra cluster with each node running on a seperate host,
While the host running ansible is a seperated host from the designated cassandra hosts.

##### Prerequisistes: 
This solution assumes you have the following up and ready:
1. Three (as per task details) up and running Ubuntu 16.04 hosts with:
	1. A user named "ubuntu"
	1. Python pre-installed on each
1. Ports 22, 7000,9042 and 7199 are to be set to allow inbound TCP traffic for each and every one of the above hosts
1. Ansible (of version 2.3.0 or higher) installed on another host with ssh access to all of the above hosts
1. An ansible hosts file or inventory file listing the three hosts from prerequisite #1 under [cassandra_nodes], see example file under 'zooz.technical-task/hosts.example'

##### Docker:    
This solution assumes that apart from the prerequisites listed above, each candidate host to be proivisioned by the provided Ansible playbook(s) will be installed with a fresh installation of Docker (community edition), including whatever dependencies required to have it installed (implemented via roles/zooz_cassandra/tasks/prerequisites.yml)

If this solution was to be deployed on a fully operational production environment, some steps, such as getting the latest version of docker-ce, were to be replaced (in this case - setting a repository to get a specific Docker version)

##### Ansible:
I've chosen to use the built in "docker_container" module in order to implement cassandra cluster creation with this solution.
At first I tried implementing this with the "docker_service" module tested on boot2docker, yet eventualy this solution was shelved.
Also was consiedered was a method of a pre-composed cassandra.yaml j2 template file in order to setup the created cassandra instance with the relevant configuration required for it to be a part of a ring.

##### Ansible Roles & Tasks:
zooz.technical-task -
* site.yml: Invokes the cluster create & repair roles while setting play required configuration and variables

roles/zooz_cassandra -
* tasks/prerequisites.yml: Installs, updates or upgrades any required dependency (such as pip, curl etc.) in order to have Docker successfully installed on played host
* tasks/cassandra.yml: Create a cassandra node docker container and sets it up to open its gossip standard port as well as initializing standard env variables for it to join a ring

roles/zooz_cassandra_repair -
* tasks/main.yml:	Issuing a 'notedool repair' command on the running docker cassandra node container. If played on all cluster nodes, it's run synchrounously and blocks until each and every node in the ring completes it's run successfully one after the other. As per task requirements, I've also included a failure handling implementation, currently commented out, which prevents the entire role to be stopped in case of one node failing to repair, while handling such failure with a debug message for the specific node (can be expanded to be handled in any other way such as retry/reschedule/send mail etc.)

##### Activation Scripts:
* Cluster Create: This script issues an ansible-playbook to run on the zooz_cassandra role
Run by issuing the following command from the solution root directory:
> ./cluster_create.sh

* Node Repair: This script issues an ansible-playbook to run on the zooz_cassandra_repair role
Run by issuing the following command from the solution root directory:
> ./node_repair.sh

#### Summary
Thank you for this challenge, I really enjoyed taking it. Thank you for taking the time to review this solution, would appreciate your feedback when comfortable.

Looking forward to meeting you on Tuesday to review it.

Avraham Kalvo
