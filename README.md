## Overview

This project provides the steps needed to build a docker environment based on AWS EC2 RHEL7 instances.
The docker hosts are provisioned and managed by an Ansible host.

## Prepare the Ansible Host

Connect to instance with ec2-user, add the epel repo, install ansible and git.

    sudo su -
    yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    yum install -y ansible git
    echo 127.0.0.1 `hostname` master >> /etc/hosts

### Clone this ansible repo from github

    git clone https://github.com/kortstie/aws_ansible.git /opt/ansible
    cd /opt/ansible

Create kortstie user, add some packages, update the system

    ./aws_master_step1.yml

Save Private Key under `/home/kortstie/.ssh/id_rsa` to access more instances from this host via ssh

    chmod 600 /home/kortstie/.ssh/id_rsa

Reboot the Instance and reconnect with kortstie...

    eval `ssh-agent`; ssh-add
    screen -S korti

### Push Changes to GitHub

    git remote set-url origin git@github.com:kortstie/aws_ansible.git
    git push origin master


### Create EC2 Ansible User

Open [AWS IAM Console](https://console.aws.amazon.com/iam/home?region=eu-central-1#)

Create an user for Ansible (Programatic Access) with these rights:
- AmazonEC2FullAccess
- SecurityAudit 

Save key and secret to a local file
```bash
echo <<EOF > ~/aws.key 
export AWS_ACCESS_KEY_ID=<key_id>
export AWS_SECRET_ACCESS_KEY=<secret>
EOF
```


Load Key and secret before ec module start: 

     . ~/aws.key

## Deploy some Docker Hosts

### Create some EC2 instances with ansible

This Playbook creates (some) EC2 instances, saves Local IPs into ~/hosts...

    aws_docker_create_instances.yml

After successful run, the private DNS and IPs should be written in ~/hosts.
Instert the IPs in /etc/hosts and assign the alias names tdoc1 and tdoc2 to them.
    
### Provisioning for the instances with docker

Creates User, Updates the system, installs Docker packages, reboot

     ./aws_docker_provisioning.yml -l tdoc*

### Start swarm

    ./aws_docker_start_swarm.yml

This playbook brings up the swarm with two nodes.

### Deploy a Stack

    ./aws_docker_start_web1.yml

This playbook deploys a webserver into our swarm.
The webserver should start on both nodes and listen on port 8081!


Todo:
- [ ] scale a stack
- [ ] take down a stack
 
