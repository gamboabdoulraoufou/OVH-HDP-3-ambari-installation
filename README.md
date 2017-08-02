### HDP-2-ambari-and-hadoop-components-installation

> Configuration
- 4 VMs on google compute Engine (3 VM for Hadoop Cluster and 1 VM for data backup and micro service)
- OS: CentOS 6
- RAM: 15Go
- CPU: 4
- Boot disk: 100Go
- Attached disk: 200G

> Cluster model

![MetaStore remote database](https://github.com/gamboabdoulraoufou/hdp-1-host-config/blob/master/img/archi.png)


> Download the Ambari repository file to a directory `_Ambari server node (instance-1)_`

```sh
# Log as root
sudo su
cd

# download
wget -nv http://public-repo-1.hortonworks.com/ambari/centos6/2.x/updates/2.4.1.0/ambari.repo 

# copy into yum repo
mv ambari.repo /etc/yum.repos.d

# copy Ambari repository on each node
scp -i /root/.ssh/id_rsa /etc/yum.repos.d/ambari.repo root@instance-2.c.equipe-1314.internal:/etc/yum.repos.d/ambari.repo
scp -i /root/.ssh/id_rsa /etc/yum.repos.d/ambari.repo root@instance-3.c.equipe-1314.internal:/etc/yum.repos.d/ambari.repo

```


> Install and configure Ambari server `_Ambari server node (instance-1)_`

```sh
# installation
yum install -y ambari-server

# configuration [help](https://community.hortonworks.com/questions/55968/ambari-agent-start-failed-2.html)
ambari-server setup

``` 



