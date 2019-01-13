# aws_ansible

# Prepare the Ansible Host

Connect to instance with ec2-user

    sudo su -
    yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    yum install -y ansible git
    echo 127.0.0.1 `hostname` master >> /etc/hosts

### Get ansible repo from github

    git clone https://github.com/kortstie/aws_ansible.git /opt/ansible
    cd /opt/ansible
    ./aws_master_step1.yml

Save Private Key under `/home/kortstie/.ssh/id_rsa` to access more instances from this host via ssh

    chmod 600 /home/kortstie/.ssh/id_rsa

Reboot the Instance and reconnect with kortstie...

    eval `ssh-agent`; ssh-add
    screen -S korti

## Push Changes to GitHub

    git remote set-url origin git@github.com:kortstie/aws_ansible.git
    git push origin master


# Create EC2 Ansible User

Open IAM Console: [link to aws](https://console.aws.amazon.com/iam/home?region=eu-central-1#)

Create an user for Ansible (Programatic Access) with these rights:
- AmazonEC2FullAccess
- SecurityAudit 

Save key and secret e.g. to a local file (aws.key)
 export AWS_ACCESS_KEY_ID=<key_id>
 export AWS_SECRET_ACCESS_KEY=<secret>

Load Key and secret before ec module start: 
 . ~/aws.key

# Create more EC2 instances with ansible

## Provision for the instances
 ./aws_docker_provisioning.yml -l tdoc*
