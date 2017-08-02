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

![Ambari-config](https://github.com/gamboabdoulraoufou/hdp-2-ambari-and-hadoop-components-installation/blob/master/img/ambari_config.png)


> Disable ...

```sh
sed -i 's@enabled=1@enabled=0@' /var/lib/ambari-server/resources/stacks/HDP/2.0.6/configuration/cluster-env.xml

```


> Install and configure Ambari agent `_All nodes_`

```sh
# install ambari agent
yum install ambari-agent

# backup config file 
cp -p /etc/ambari-agent/conf/ambari-agent.ini /etc/ambari-agent/conf/ambari-agent.ini.ORI

# edit file
vi /etc/ambari-agent/conf/ambari-agent.ini

# add then content bellow
[server]
# hostname=localhost
# hostname=ambari_server_host_name for all nodes
hostname=instance-1.c.equipe-1314.internal
[agent]
hostname_script=/var/lib/ambari-agent/hostname.sh

# save and quit

# create hostname.sh file
vi /var/lib/ambari-agent/hostname.sh

# add the current host name
#!/bin/sh
echo instance-1.c.equipe-1314.internal

# save and quit

# set permission
chmod +x /var/lib/ambari-agent/hostname.sh

# start Ambari server (on ambari server host)
ambari-server start

# start Ambari agent
ambari-agent start

```


> Connecte to Ambari web UI
- Go to http://IP:8080 or http://hostname:8080
- Login: admin
- Password: admin

You should see something like this

![Ambari-config](https://github.com/gamboabdoulraoufou/hdp-2-ambari-and-hadoop-components-installation/blob/master/img/ambari_ui.png)


> Create cluster




