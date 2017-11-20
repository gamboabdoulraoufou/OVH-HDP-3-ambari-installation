### HDP-2-ambari-installation

> Configuration
- 4 VMs on google compute Engine (3 VM for Hadoop Cluster and 1 VM for data backup and micro services)
- OS: CentOS 7
- RAM: 15 Go
- CPU: 4
- Boot disk: 200Go
- Attached disk: 2000G

> Cluster model

![MetaStore remote database](https://github.com/gamboabdoulraoufou/hdp-1-host-config/blob/master/img/archi_v2.png)


> Download and install Ambari server `_Ambari server node (hdp-1)_`

```sh
# log as root
sudo su - root

# download
## from internet (https://docs.hortonworks.com/HDPDocuments/Ambari-2.5.1.0/bk_ambari-installation/content/ambari_repositories.html)
wget http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.4.0.1/ambari.repo
wget http://public-repo-1.hortonworks.com/HDP/centos7/2.x/updates/2.5.6.0/hdp.repo
wget http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.21/repos/centos7/HDP-UTILS-1.1.0.21-centos7.tar.gz
mkdir HDP-UTILS
tar -xvzf HDP-UTILS-1.1.0.21-centos7.tar.gz -C HDP-UTILS
cp HDP-UTILS/hdp-util.repo .

## from local repository
wget http://velvet-repo.c.projet-ic-166005.internal/repo/AMBARI/centos7/2.4.0.1-1/ambari.repo
wget http://velvet-repo.c.projet-ic-166005.internal/repo/HDP/centos7/hdp.repo
wget http://velvet-repo.c.projet-ic-166005.internal/repo/HDP-UTILS/hdp-util.repo

# copy ambari repos into yum repos 
mv ambari.repo /etc/yum.repos.d
mv hdp.repo /etc/yum.repos.d
mv hdp-util.repo /etc/yum.repos.d

# copy ambari repo into all other nodes
scp -i /root/.ssh/id_rsa /etc/yum.repos.d/ambari.repo root@poc-stream-processing-2.c.equipe-1314.internal:/etc/yum.repos.d/
scp -i /root/.ssh/id_rsa /etc/yum.repos.d/ambari.repo root@poc-stream-processing-3.c.equipe-1314.internal:/etc/yum.repos.d/

# copy hdp-utils repo into all other nodes
scp -i /root/.ssh/id_rsa /etc/yum.repos.d/hdp-util.repo root@poc-stream-processing-2.c.equipe-1314.internal:/etc/yum.repos.d/
scp -i /root/.ssh/id_rsa /etc/yum.repos.d/hdp-util.repo root@poc-stream-processing-3.c.equipe-1314.internal:/etc/yum.repos.d/

# copy hdp repo into all other nodes
scp -i /root/.ssh/id_rsa /etc/yum.repos.d/hdp.repo root@poc-stream-processing-2.c.equipe-1314.internal:/etc/yum.repos.d/
scp -i /root/.ssh/id_rsa /etc/yum.repos.d/hdp.repo root@poc-stream-processing-3.c.equipe-1314.internal:/etc/yum.repos.d/

# installation
yum clean all
yum -y update
yum install -y ambari-server
```

> Install MySQL database `SQL server (_hdp-1)_`

```sh
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
 
> Configure MySQL database step 1 `Ambari Server host (_hdp-1)_` 
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
bind-address = poc-stream-processing-1.c.equipe-1314.internal
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

> Configure MySQL database step 2 `Ambari Server host (_hdp-1)_`
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

> Configure MySQL database step 3 `Ambari Server host (_hdp-1)_`
```sh
# installing the MySQL JDBC Connector 
yum install -y mysql-connector-java*

# check installation
ll /usr/share/java/mysql-connector-java.jar

# make sure the .jar file has the appropriate permissions - 644

# stage the appropriate MySQL connector for later deployment
ambari-server setup --jdbc-db=mysql --jdbc-driver=/usr/share/java/mysql-connector-java.jar

# restart MySQL
systemctl restart mysqld

# ensure the MySQL server starts at boot
systemctl enable mysqld.service
systemctl list-dependencies mysqld
``` 

> Set up MySQL database for Ambari  `Ambari Server host (_hdp-1)_` [Ref.](https://docs.hortonworks.com/HDPDocuments/Ambari-2.1.2.1/bk_ambari_reference_guide/content/_using_ambari_with_mysql.html)

```sh
# log to MySQL
mysql -u root -p
```

```sql
# Create a user for Ambari and grant it permissions
CREATE USER 'ambari_user'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'ambari_user'@'%';
CREATE USER 'ambari_user'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'ambari_user'@'localhost';
CREATE USER 'ambari_user'@'instance-1.c.equipe-1314.internal' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'ambari_user'@'instance-1.c.equipe-1314.internal';
FLUSH PRIVILEGES;

# quit MySQL
exit
```

```sh
# log to MySQL
mysql -u ambari_user -p
```

```sql
# Load the Ambari Server database schema
CREATE DATABASE ambari_database;
USE ambari_database;
SOURCE /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql;

# quit MySQL
exit
```
When setting up the Ambari Server, select Advanced Database Configuration > Option [3] MySQL and set password


> Set up MySQL database for Hive [Ref.](https://docs.hortonworks.com/HDPDocuments/Ambari-2.1.2.1/bk_ambari_reference_guide/content/_using_hive_with_mysql.html) `_Ambari server node (hdp-1)_`

```sh
# log to MySQL
mysql -u root -p
```

```sql
# create a user for Hive and grant it permissions
CREATE USER 'hive_user'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'hive_user'@'localhost';
CREATE USER 'hive_user'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'hive_user'@'%';
CREATE USER 'hive_user'@'instance-1.c.equipe-1314.internal' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'hive_user'@'instance-1.c.equipe-1314.internal';
FLUSH PRIVILEGES;

# quit MysQL
exit
```

```sh
# log to MySQL
mysql -u hive_user -p
```

```sql
CREATE DATABASE hive_database;
```

> Set up MySQL database for OOZIE [Ref.](https://docs.hortonworks.com/HDPDocuments/Ambari-2.1.2.1/bk_ambari_reference_guide/content/_using_oozie_with_mysql.html) `_Ambari server node (hdp-1)_`

```sh
# log to MySQL
mysql -u root -p
```

```sql
# create a user for oozie and grant it permissions
CREATE USER 'oozie_user'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'oozie_user'@'%';
FLUSH PRIVILEGES;

# quit MysQL
exit
```

```sh
# log to MySQL
mysql -u oozie_user -p
```

```sql
CREATE DATABASE oozie_database;
```


```sh
# log to MySQL
mysql -u root -p
```

```sql
# create a user for oozie and grant it permissions
CREATE USER 'hue_user'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'hue_user'@'%';
FLUSH PRIVILEGES;

# check users
SELECT User, Host, Password FROM mysql.user;

# quit MysQL
exit
```

```sh
# log to MySQL
mysql -u hue_user -p
```

```sql
CREATE DATABASE hue_database;
```

> Configure Ambari server `_Ambari server node (hdp-1)_`

```sh
# configuration [Ref.](https://community.hortonworks.com/questions/55968/ambari-agent-start-failed-2.html)
ambari-server setup

``` 

![Ambari-config](https://github.com/gamboabdoulraoufou/hdp-2-ambari-and-hadoop-components-installation/blob/master/img/ambari_config2.png)


> Disable ...  `_Ambari server node (hdp-1)_`

```sh
sed -i 's@enabled=1@enabled=0@' /var/lib/ambari-server/resources/stacks/HDP/2.0.6/configuration/cluster-env.xml
```


> Disable ...  `_all nodes_`

```sh
sed -i 's/verify=platform_default/verify=disable/' /etc/python/cert-verification.cfg
```



> Install and configure Ambari agent `_All nodes_`

```sh
# install ambari agent
yum -y install ambari-agent

# backup config file 
cp -p /etc/ambari-agent/conf/ambari-agent.ini /etc/ambari-agent/conf/ambari-agent.ini.ORI

# edit file
vi /etc/ambari-agent/conf/ambari-agent.ini

# add then content bellow
[server]
# hostname=localhost
# hostname=ambari_server_host_name for all nodes
hostname=hdp-1.c.projet-ic-166005.internal
[agent]
hostname_script=/var/lib/ambari-agent/hostname.sh

# save and quit

# create hostname.sh file
vi /var/lib/ambari-agent/hostname.sh

# add the current host name
#!/bin/sh
echo hdp-1.c.projet-ic-166005.internal

# save and quit

# set permission
chmod +x /var/lib/ambari-agent/hostname.sh

```

> Start Ambari server  `_Ambari server node (hdp-1)_`

```sh
# start Ambari server
ambari-server start

# start ambari server after reboot
crontab -e

# add the following line
#### START ####
@reboot ambari-server start
#### END ####
```

> Start Ambari agent `_All nodes_`

```sh
# start Ambari agent
ambari-agent start

# start ambari server after reboot
crontab -e

# add the following line
#### START ####
@reboot ambari-agent start
#### END ####

```


> Connecte to Ambari web UI
- Go to http://IP:8080 or http://hostname:8080
- Login: admin
- Password: admin

You should see something like this

![Ambari-config](https://github.com/gamboabdoulraoufou/hdp-2-ambari-and-hadoop-components-installation/blob/master/img/ambari_ui.png)





