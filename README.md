### Install HDP
This is the second article of series about setup a secured HDP cluster

#Prerequisits
First make sure root is enabled on all Node.
```
sudo vi /root/.ssh/authorized_keys
PermitRootLogin yes

sudo vi /etc/ssh/sshd_config
--make ssh key available
```

Then setup passwordless ssh
```
ssh-keygen -t rsa -b 4096
```
and copy the private and public keys over to nodes, only need to copy private key to ambari node
```
scp id_rsa root@ambari_node:/root/.ssh/id_rsa
scp id_rsa.pub root@all_nodes:/root/.ssh/id_rsa.pub
```
Then add public key to authorized_keys 
```
cd ~/.ssh
cat id_rsa.pub >> authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
cd ~
```
Now you should be able to ssh from Ambari node to all other nodes without password

Edit hosts file include all hosts
```
vi /etc/hosts

192.168.0.3 node1.example.com
192.168.0.4 node2.example.com
192.168.0.5 node3.example.com
```
Then install all the software needed
```
yum -y install wget ntp firewalld git httpd openldap-clients
ntpdate pool.ntp.org 
systemctl is-enabled ntpd
systemctl enable ntpd
systemctl start ntpd

systemctl disable firewalld
systemctl stop firewalld

echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag

setenforce 0

vi /etc/selinux/config
--change the following line
SELINUX=disabled
```

#Install Ambari server
You may need to change the repo based on the latest [document.](http://docs.hortonworks.com/index.html) 
```
wget -nv http://public-repo-1.hortonworks.com/ambari/centos6/2.x/updates/2.4.2.0/ambari.repo -O /etc/yum.repos.d/ambari.repo
yum repolist
yum -y install ambari-server
```
Copy DDL for Ambari folder to MySQL host and load the DDL on MySQL host
```
scp /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql root@mysql_node:/tmp/Ambari-DDL-MySQL-CREATE.sql
mysql -u root -p ambari < /tmp/Ambari-DDL-MySQL-CREATE.sql
```
Copy MySQL jdbc JAR to Ambari server
```
mkdir -p /usr/share/java/
scp root@mysql_node:/usr/share/java/* /usr/share/java/
```
Then run Ambari setup
```
ambari-server setup

ambari-server setup --jdbc-db=mysql --jdbc-driver=/usr/share/java/mysql-connector-java.jar
```
