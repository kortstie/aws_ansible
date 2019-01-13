# aws_ansible
Connect to instance with ec2-user
sudo su -
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum install -y ansible git

echo 127.0.0.1 `hostname` master >> /etc/hosts

git clone https://github.com/kortstie/aws_ansible.git /opt/ansible
cd /opt/ansible
./aws_master_step1.yml

Reboot the Instance and reconnect with kortstie...
