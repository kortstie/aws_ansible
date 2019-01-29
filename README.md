## Overview

This project provides the steps needed to build a demo environment with some AWS EC2 RHEL7 or Amazon Linux 2 instances.
The instances are provisioned and managed by an Ansible host. In this demo the Ansible Host should be named "master".
The "master" host has to be created manually, the configuration of this host is described in the following steps:

## Prepare the Ansible Host

Connect to instance "master" with ec2-user, add the epel repo, install ansible and git.

    sudo su -
    yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    yum install -y ansible git
    echo 127.0.0.1 `hostname` master >> /etc/hosts
    

### Clone this ansible repo from github

    git clone https://github.com/kortstie/aws_ansible.git /opt/ansible
    cd /opt/ansible

Create kortstie user, add some packages, update the system

    ./aws_master_step1.yml

Save Private Key under `/home/kortstie/.ssh/id_rsa` to access more instances from this host via ssh, don't
forget to set the right ownership and access rights on the file... `600`

Reboot the Instance and reconnect with kortstie...

### Push Changes to GitHub

    git remote set-url origin git@github.com:kortstie/aws_ansible.git
    git push origin master


### Create EC2 Ansible User

Open [AWS IAM Console](https://console.aws.amazon.com/iam/home?region=eu-central-1#)

Create an user for Ansible (Programatic Access) with these rights:
- AmazonEC2FullAccess
- SecurityAudit 

Save key and secret e.g. to a local file
```bash
echo <<EOF > ~/aws.key 
export AWS_ACCESS_KEY_ID=<key_id>
export AWS_SECRET_ACCESS_KEY=<secret>
EOF
```

Load Key and secret before ec module start: 

     . ~/aws.key

## Deploy some instances in aws EC2

This Playbook creates (some) EC2 instances, saves the local IPs into /etc/hosts and executes the "firststep" role on the new instances.
If the play-hosts are not limited on the commandline, all hosts listed in the Ansible inventory (inventory/hosts) are created!

Examples:

    ./aws_create_instances.yml
    ./aws_create_instances.yml -l tdoc1
    ./aws_create_instances.yml --extra-vars="os=rhel7"


    
### Provisioning for a Docker host

Installs Docker packages, docker-python modules, assigns dockerroot group ...

     ./aws_docker_provisioning.yml -l tdoc*

### Start swarm

    ./aws_docker_start_swarm.yml

This playbook brings up the swarm with two nodes.

### Deploy a Stack

    ./aws_docker_start_web1.yml

This playbook deploys a webserver into our swarm.
The webserver should start on both nodes and listen on port 8081!

### Put it all together!

    ./aws_create_instances.yml && ./aws_docker_provisioning.yml && ./aws_docker_start_swarm.yml && ./aws_docker_start_web1.yml && watch curl tdoc1:8081

