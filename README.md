### HDP-2-ambari-installation

> Configuration
- 4 VMs on google compute Engine (3 VM for Hadoop Cluster and 1 VM for data backup and micro service)
- OS: CentOS 6
- RAM: 15Go
- CPU: 4
- Boot disk: 100Go
- Attached disk: 200G

> Cluster model

![MetaStore remote database](https://github.com/gamboabdoulraoufou/hdp-1-host-config/blob/master/img/archi.png)


> Setting Up a Local Repository

```sh
# install yum utilities
yum install -y yum-utils createrepo

# create an HTTP server
yum clean all
yum -y update
yum -y install httpd
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --reload
systemctl start httpd
systemctl enable httpd
systemctl status httpd

# create a directory for your web server
mkdir -p /var/www/html/

# create hdf repo folder
mkdir repo

# get ambari tarball (https://docs.hortonworks.com/HDPDocuments/Ambari-2.5.1.0/bk_ambari-installation/content/ambari_repositories.html)
wget http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.5.1.0/ambari-2.5.1.0-centos7.tar.gz
wget http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.5.1.0/ambari.repo

# get hdp tarball (https://docs.hortonworks.com/HDPDocuments/Ambari-2.5.1.0/bk_ambari-installation/content/hdp_stack_repositories.html)
wget http://public-repo-1.hortonworks.com/HDP/centos7/2.x/updates/2.5.6.0/HDP-2.5.6.0-centos7-rpm.tar.gz
wget http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.21/repos/centos7/HDP-UTILS-1.1.0.21-centos7.tar.gz
wget http://public-repo-1.hortonworks.com/HDP/centos7/2.x/updates/2.5.6.0/hdp.repo

# check files
cat *.repo

# untar file
mkdir HDP-UTILS
tar -xvzf ambari-2.5.1.0-centos7.tar.gz; tar -xvzf HDP-2.5.6.0-centos7-rpm.tar.gz; tar -xvzf HDP-UTILS-1.1.0.21-centos7.tar.gz -C HDP-UTILS

# move file into repo folder
mv ambari/ /var/www/html/repo
mv HDP /var/www/html/repo
mv HDP-UTILS /var/www/html/repo

# check files
ll /var/www/html/repo

drwxr-xr-x  3 cloud users   21 Jul  3 05:54 ambari
drwxr-xr-x  3 cloud users   21 Jun 26 19:06 HDP
drwxr-xr-x 21 root  root  4096 Aug  5 12:00 HDP-UTILS

```


> Go to go to http://IP/repo/ or http://hostname/repo

You should see something like this

> Set ambari repo
```sh
cat ambari.repo

vi ambari.repo

baseurl=http://35.197.46.155/repo/ambari/centos7
```
> Download and install Ambari server `_Ambari server node (instance-1)_`

```sh
# log as root
sudo su
cd

# download
wget http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.4.0.1/ambari.repo
wget http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.5.6.0/ambari.repo

# copy ambari repos into yum repos 
mv ambari.repo /etc/yum.repos.d

# copy ambari repo into all other nodes
scp -i /root/.ssh/id_rsa /etc/yum.repos.d/ambari.repo root@instance-2.c.equipe-1314.internal:/etc/yum.repos.d/ambari.repo
scp -i /root/.ssh/id_rsa /etc/yum.repos.d/ambari.repo root@instance-3.c.equipe-1314.internal:/etc/yum.repos.d/ambari.repo

# installation
yum install -y ambari-server
```

> Install MySQL database `Ambari Server host (_instance-1)_`

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
 
> Configure MySQL database step 1 `Ambari Server host (_instance-1)_` 
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

> Configure MySQL database step 2 `Ambari Server host (_instance-3)_`
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

> Configure MySQL database step 3 `Ambari Server host (_instance-3)_`
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

> Set up MySQL database for Ambari [Ref.](https://docs.hortonworks.com/HDPDocuments/Ambari-2.1.2.1/bk_ambari_reference_guide/content/_using_ambari_with_mysql.html)

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


> Set up MySQL database for Hive [Ref.](https://docs.hortonworks.com/HDPDocuments/Ambari-2.1.2.1/bk_ambari_reference_guide/content/_using_hive_with_mysql.html) `_Ambari server node (instance-1)_`

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

# check users
SELECT User, Host, Password FROM mysql.user;

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

> Set up MySQL database for OOZIE [Ref.](https://docs.hortonworks.com/HDPDocuments/Ambari-2.1.2.1/bk_ambari_reference_guide/content/_using_oozie_with_mysql.html) `_Ambari server node (instance-1)_`

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


> Configure Ambari server `_Ambari server node (instance-3)_`

```sh
# configuration [Ref.](https://community.hortonworks.com/questions/55968/ambari-agent-start-failed-2.html)
ambari-server setup

``` 

![Ambari-config](https://github.com/gamboabdoulraoufou/hdp-2-ambari-and-hadoop-components-installation/blob/master/img/ambari_config2.png)


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
hostname=instance-3.c.equipe-1314.internal
[agent]
hostname_script=/var/lib/ambari-agent/hostname.sh

# save and quit

# create hostname.sh file
vi /var/lib/ambari-agent/hostname.sh

# add the current host name
#!/bin/sh
echo instance-3.c.equipe-1314.internal

# save and quit

# set permission
chmod +x /var/lib/ambari-agent/hostname.sh

```

> Start Ambari server `_Ambari server node (instance-3)_`

```sh
# start Ambari server
ambari-server start
```

> Start Ambari agent `_All nodes_`

```sh
# start Ambari agent
ambari-agent start
```


> Connecte to Ambari web UI
- Go to http://IP:8080 or http://hostname:8080
- Login: admin
- Password: admin

You should see something like this

![Ambari-config](https://github.com/gamboabdoulraoufou/hdp-2-ambari-and-hadoop-components-installation/blob/master/img/ambari_ui.png)





