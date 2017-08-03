### HDP-2-ambari-and-hadoop-cluster-installation

> Configuration
- 4 VMs on google compute Engine (3 VM for Hadoop Cluster and 1 VM for data backup and micro service)
- OS: CentOS 6
- RAM: 15Go
- CPU: 4
- Boot disk: 100Go
- Attached disk: 200G

> Cluster model

![MetaStore remote database](https://github.com/gamboabdoulraoufou/hdp-1-host-config/blob/master/img/archi.png)


> Install MySQL database `MySQL node (_instance-3)_`

```sh
# log as root
sudo su
 
# check hostname
hostname -f

# download and add the repository, then update
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
rpm -ivh mysql-community-release-el7-5.noarch.rpm
yum -y update

# Install MySQL
yum -y install mysql-server
systemctl start mysqld
``` 
 
> Configure MySQL database step 1 `MySQL node (_instance-3)_` 
```sh  
# Update /etc/my.cnf or /etc/mysql/my.cnf file at least the values shown below

# edit file 
vi /etc/my.cnf
```` 

>  Add or change the conf according the setup bellow
```sh 
### Start 
[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
bind-address = instance-1.c.equipe-1314.internal
#
key_buffer              = 16M
key_buffer_size         = 32M
max_allowed_packet      = 16M
thread_stack            = 256K
thread_cache_size       = 64
query_cache_limit       = 8M
query_cache_size        = 64M
query_cache_type        = 1
# Important: see Configuring the Databases and Setting max_connections
max_connections         = 550
#
read_buffer_size = 2M
read_rnd_buffer_size = 16M
sort_buffer_size = 8M
join_buffer_size = 8M
#
# InnoDB settings
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit  = 2
innodb_log_buffer_size          = 64M
innodb_buffer_pool_size         = 4G
innodb_thread_concurrency       = 8
innodb_flush_method             = O_DIRECT
innodb_log_file_size = 512M
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

# Recommended in standard MySQL setup
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
### End

# restart mysql
systemctl restart mysqld
``` 

> Configure MySQL database step 2 `MySQL node (_instance-3)_`
You will be given the choice to change:
- root password
- remove anonymous user accounts
- disable root logins outside of localhost
- remove test databases
It is recommended that you answer yes to these options

```sh  
# run the mysql_secure_installation script to address several security concerns in a default MySQL installation
mysql_secure_installation

# press enter and set all parameter to yes (change root password)
``` 

![MetaStore remote database](https://github.com/gamboabdoulraoufou/Cloudera-2-Cloudera-Manager-instllation/blob/master/img/mysql_secure_installation.PNG)

> Configure MySQL database step 3 `MySQL node (_instance-3)_`
```sh
# installing the MySQL JDBC Connector
yum install -y mysql-connector-java 

# restart
systemctl restart mysqld

# ensure the MySQL server starts at boot
systemctl enable mysqld.service
systemctl list-dependencies mysqld
``` 

> Configure MySQL database step 4 `MySQL node (_instance-3)_` 
create the required databases in MySQL
- Activity Monitor
- Service Monitor
- Report Manager
- Host Monitor
- Hive metastore
- Cloudera Navigator
 
```sh  
# log into MySQL as the root user
mysql -u root -p


amon, cmserver, hue, metastore, oozie, and rman
# Create a database for the Activity Monitor
create database amon DEFAULT CHARACTER SET utf8;
grant all on amon.* TO 'amon_user'@'%' IDENTIFIED BY 'password';

# create a database for the cmserver
create database cmserver DEFAULT CHARACTER SET utf8;
grant all on cmserver.* TO 'cmserver_user'@'%' IDENTIFIED BY 'password';

# create a database for hue
create database hue DEFAULT CHARACTER SET utf8;
grant all on hue.* TO 'hue_user'@'%' IDENTIFIED BY 'password';

# create a database for the Hive metastore
create database metastore DEFAULT CHARACTER SET utf8;
grant all on metastore.* TO 'metastore_user'@'%' IDENTIFIED BY 'password';

# create a database for oozie
create database oozie DEFAULT CHARACTER SET utf8;
grant all on oozie.* TO 'oozie_user'@'%' IDENTIFIED BY 'password';

# create a database for the ressource manager
create database rman DEFAULT CHARACTER SET utf8;
grant all on rman.* TO 'rman_user'@'%' IDENTIFIED BY 'password';

# quit mysql
exit

# Backing Up the MySQL Databases
#mysqldump -h<hostname> -u<username> -p<password> <database> > /tmp/<database-backup>.sql
#mysqldump -hmyhost.example.com -uroot -pcloudera scm_database > /tmp/scm_database-backup.sql
```

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





