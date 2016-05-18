These instructions are for installing MariaDB on CentOS 7.  They may work with other Linux distributions.
Since MariaDB is a branch of MySQL, information here may be relevant to MySQL installations.


No warranty, expressed or implied, are provided from the use of this document.


CREATE REPOSITORY

Browse http://mariadb.org/mariadb/repositories/ to get repository information.
Create a repo file, e.g., /etc/yum.repos.d/mariadb.repo based on repository information from site.
  [mariadb]
  name = MariaDB
  baseurl = http://yum.mariadb.org/10.0/centos7-amd64
  gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
  gpgcheck=1
  enabled=1




EPEL REPOSITORY IF NEEDED

# yum install epel-release-7-1.noarch.rpm
# rpm -iUvh http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-6.noarch.rpm




INSTALL MARIADB WITH YUM

# yum install MariaDB-client MariaDB-server
# service mysql start
# mysql_secure_installation (secure the database and create root password)




STARTING MARIADB AT BOOT

# chkconfig --list mysql
# chkconfig --levels 2345 mysql on




ROOT DENIED ERRORS or UPDATE ROOT PASSWORD

# service mysql stop
# mysqld_safe --skip-grant-tables &
# mysql -u root
use mysql;
UPDATE mysql.user SET password=PASSWORD("new-root-password") WHERE user='root';
DELETE FROM mysql.user WHERE user='root' AND host NOT IN ('127.0.0.1', '::1', 'localhost');
exit;




INSTALL MYPHPADMIN AND APACHE

# rpm -iUvh http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-6.noarch.rpm (if needed)
# yum install httpd
# yum install php php-mysql php-gd php-pear
# edit /etc/httpd/conf.d/phpMyAdmin.conf and update usr/share/phpMyAdmin/ section
(you might want to change this configuration further for tighter security)
   <Directory /usr/share/phpMyAdmin/>
      Options Indexes FollowSymLinks MultiViews
      DirectoryIndex index.php
      AllowOverride all
      Deny from All
      Allow from 127.0.0.1
      Require all granted
   </Directory>

# service httpd restart
Browse http://127.0.0.1/phpmyadmin/




FIREWALL UPDATES IF NEEDED

Firewall allow
firewall-cmd --zone=public --add-service=http
  or
firewall-cmd --permanent --zone=public --add-service=http (if you want it permanent)
  or
firewall-cmd --permanent --zone=public --add-port=80/tcp (if you want it permanent)

firewall-cmd --reload


Fireall deny
firewall-cmd --zone=public --remove-service=http
  or
firewall-cmd --permanent --zone=public --remove-service=http (if you want it permanent)
  or 
firewall-cmd --permanent --zone=public --remove-port=80/tcp (if you want it permanent)

firewall-cmd --reload




REPLICATION SETUP
This assumes there are multiple servers or multiple instances

master-slave setup

On first database server
# mysql -u root -p
show master status;
note the log file name (we'll call it LOGNAME1) and position number (we'll call it POS1)
stop slave;
CREATE USER 'replicate'@'%' IDENTIFIED BY 'replicate-user-password';
GRANT REPLICATION SLAVE on *.* TO 'replicate'@'%';

CHANGE MASTER TO
MASTER_HOST='secondservername.domain.com",
MASTER_USER='replicate',
MASTER_PASSWORD='replicate-user-password',
MASTER_LOG_FILE=name-of-LOGNAME2,
MASTER_LOG_POS=number-from-POS2,
MASTER_PORT=3306;
start slave;
show slave status;


On second database server
# mysq -u root -p
show master status;
note the log file name (we'll call it LOGNAME2) and position number (we'll call it POS2)



master-master setup

On first database server
# mysql -u root -p
show master status;
note the log file name (we'll call it LOGNAME1) and position number (we'll call it POS1)
stop slave;

CREATE USER 'replicate'@'%' IDENTIFIED BY 'replicate-user-password';
GRANT REPLICATION SLAVE on *.* TO 'replicate'@'%';

CHANGE MASTER TO
MASTER_HOST='secondservername.domain.com",
MASTER_USER='replicate',
MASTER_PASSWORD='replicate-user-password',
MASTER_LOG_FILE=name-of-LOGNAME2,
MASTER_LOG_POS=number-from-POS2,
MASTER_PORT=3306;
start slave;
show slave status;


On second database server
# mysql -u root -p
show master status;
note the log file name (we'll call it LOGNAME2) and position number (we'll call it POS2)
stop slave;

CREATE USER 'replicate'@'%' IDENTIFIED BY 'replicate-user-password';
GRANT REPLICATION SLAVE on *.* TO 'replicate'@'%';

CHANGE MASTER TO
MASTER_HOST='firstservername.domain.com",
MASTER_USER='replicate',
MASTER_PASSWORD='replicate-user-password',
MASTER_LOG_FILE=name-of-LOGNAME1,
MASTER_LOG_POS=number-from-POS1,
MASTER_PORT=3306;
start slave;
show slave status;




MISC COMMANDS
mysql -u root -p
mysqld_safe --skip-grant-tables --skip-networking &
systemctl start httpd
rpm -iUvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release6-8.noarch.rpm
