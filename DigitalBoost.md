CAPSTONE PROJECT: WORDPRESS SITE ON AWS
PROJECT COMPONENTS AND DESCRIPTION OF PROJECT SETUP
1.	VPC SETUP;
•	Firstly, I signed on to my account on AWS management console, I searched for VPC service and my VPC dashboard, I clicked on your VPCs button, then I clicked on create VPC button. I chose VPC only option, I gave it a name, for IPv4 CIDR block, I selected IPv4 CIDR manual input. For IPv4 CIDR: I chose 14.0.0.0/16. For the tenancy, I left it as default, and chose No IPv6 CIDR block. Then I clicked on create VPC. It was created successfully.
•	Secondly, to create six subnets; four private subnets and two public subnets. The public subnets for the Web server, while two of the private subnets are for the app-server and the remaining two subnets are for database server. I clicked on the subnet on the left-hand side under the VPC menu. I clicked on create subnet. For VPC ID, I selected the VPC I earlier created. This means I am connecting the subnets to the VPC that I created. For the three subnets, they are created in one Availability Zone, while the remaining three are created in another availability zone.  I chose us-east-1a and us-east-1b. for IPv4 subnet CIDR block, I chose 14.0.1.0/24 – 14.0.6.0/24, which gives me access to 256 Ips each. Then I clicked on add new subnet button. Note the six subnets are created across two availability zones.
•	Create Internet Gateway; click on internet gateways, create internet gateway. After creating IGW, click on action to attach it to the VPC earlier created.
•	Create NAT gateway; so that my instances in the private subnet will have access to the internet through a secured gateway channel. I clicked on NAT gateway on the left navigation pane, I clicked on create NAT gateway, I gave it a name, selected a public subnet in AZ one, connectivity type is public. Generate an elastic IP, and then click on create NAT gateway. 
•	Configuration of two route tables one each for the public and private subnets. I clicked on route tables on the VPC menu on left-hand pane, I clicked on create route table, give it a name and selected the created VPC to it. Click on create route table. After creating the two route tables, one public and the other private, I clicked on the public route, select routes, edit routes, click on add route, for destination=0.0.0.0/0, and for target, I chose internet gateway that I created and saved changes. Then, I clicked on subnet associations, edit subnet association, I attached the two public subnets that I created and saved changes. I selected the private route I created, select routes, edit routes and add route; destination=0.0.0.0/0 and target=NAT gateway that I created, and saved changes.
•	Creation of Security Group; select the created VPC, add inbound rule; http and https=0.0.0.0/0. This SG is for the ALB, I created another SG for the web-server with inbound rules; http and https=ALB-sg and ssh=ssh-sg. Create SG for app-server; mysql=webserver-sg.

2.	AWS MYSQL RDS SETUP;

•	I went to RDS and selected subnet groups, I clicked on create db subnet group, give it a name and description. I selected the VPC I created; I also selected the two availability zones that I have chosen for my subnets. For the subnets, I chose the two private subnets I created for my database. I clicked create.
•	I went to search bar to search for RDS service, I clicked on db instance, I clicked on create database button. I selected standard create, engine option, I chose MYSQL engine. For engine version, I chose the most recent 8.0.36. for template, I chose free tier. For DB instance identifier, I chose the default: database-1. Under credential settings, I opted for admin as the master username, for credential management, I chose self-managed and created my password. I chose db.t3.micro. I did not enable storage auto scale. I connected the RDS to my created VPC and the created subnet security group. For public access, I selected No. I selected the created VPC security group earlier created.  For AZ I left it as no preference. For authentication, I chose password authentication. I created RDS-MYSQL.

•	I returned to EC2, and clicked on launch instance. I created an instance for my web-server, using amazon image, I chose the created VPC, public subnet in AZ1 and selected the created security group I earlier created for the web-server. I launched the instance.

•	To connect WordPress into my RDS database; to configure WordPress, I ssh into my wps-web-server, and copy the bash script provided for the project, for the update and installation of Apache, php and mysql. On my terminal in the web-server, I created a file using vim and pasted the bash script, I saved and exited. I changed the file permission to enable execution right. I ran the bash execution command and apache, php and mysql were successfully installed. As shown below.

3.	EFS SETUP FOR WORDPRESS FILES;
•	On the AWS management console, I search for EFS service, I clicked on it, and I clicked on create file system button I gave it a name, chose my VPC, for file system type, I chose regional, performance setting I chose bursting. For mount target the two AZ were already selected I attached EFS-SG earlier created. I clicked create. As soon as EFS is available, I clicked on attach button,  I used NFS client to mount wordpress.
4.	CREATE APPLICATION LOAD BALANCER
•	I clicked on create application load balancer, and I configured listener. Also create auto scaling group and integrate same with my ALB.
5.	Testing result below;



sojy@OlasojiUbuntu:~$ cd Downloads
sojy@OlasojiUbuntu:~/Downloads$ chm od 400 "wps-webserver-key.pem"
sojy@OlasojiUbuntu:~/Downloads$ ssh -i "wps-webserver-key.pem" ec2-user@ec2-44-206-34-186.compute-1.amazonaws.com
The authenticity of host &apos;ec2-44-206-34-186.compute-1.amazonaws.com (44.206.34.186)&apos; can&apos;t be established.
ED25519 key fingerprint is SHA256:VNKBgYLdcsaeMA5mLb6p6nvzWH/r9ADtlYjRFBFi4oo.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:67: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added &apos;ec2-44-206-34-186.compute-1.amazonaws.com&apos; (ED25519) to the list of known hosts.
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~&apos; &apos;->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/&apos;
Last login: Mon May  6 19:41:15 2024 from 102.91.52.114
[ec2-user@ip-14-0-1-200 ~]$ 


[ec2-user@ip-14-0-1-200 ~]$ sudo su
[root@ip-14-0-1-200 ec2-user]# vi wordpress
•	I run this bash script;

!#/bin/bash

#1. create the html directory and mount the efs to it
sudo su
yum update -y
mkdir -p /var/www/html
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-03c9b3354880b36a6.efs.us-east-1.amazonaws.com:/ /var/www/html


#2. install apache 
sudo yum install -y httpd httpd-tools mod_ssl
sudo systemctl enable httpd 
sudo systemctl start httpd


#3. install php 7.4
sudo amazon-linux-extras enable php7.4
sudo yum clean metadata
sudo yum install php php-common php-pear -y
sudo yum install php-{cgi,curl,mbstring,gd,mysqlnd,gettext,json,xml,fpm,intl,zip} -y


#4. install mysql5.7
sudo rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
sudo yum install mysql-community-server -y
sudo systemctl enable mysqld
sudo systemctl start mysqld


#5. set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html 


#6. download wordpress files
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
cp -r wordpress/* /var/www/html/


#7. create the wp-config.php file
cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php


#8. edit the wp-config.php file
nano /var/www/html/wp-config.php


#9. restart the webserver
service httpd restart
See feedback;

[root@ip-14-0-1-200 ec2-user]# vi wordpress
[root@ip-14-0-1-200 ec2-user]# ./wordpress
bash: ./wordpress: Permission denied
[root@ip-14-0-1-200 ec2-user]# rm -rf wordpress
[root@ip-14-0-1-200 ec2-user]# vi wordpress.sh


[root@ip-14-0-1-200 ec2-user]# ls
wordpress.sh
[root@ip-14-0-1-200 ec2-user]# cat wordpress.sh
!#/bin/bash

#1. create the html directory and mount the efs to it
sudo su
yum update -y
mkdir -p /var/www/html
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-03c9b3354880b36a6.efs.us-east-1.amazonaws.com:/ /var/www/html


#2. install apache 
sudo yum install -y httpd httpd-tools mod_ssl
sudo systemctl enable httpd 
sudo systemctl start httpd


#3. install php 7.4
sudo amazon-linux-extras enable php7.4
sudo yum clean metadata
sudo yum install php php-common php-pear -y
sudo yum install php-{cgi,curl,mbstring,gd,mysqlnd,gettext,json,xml,fpm,intl,zip} -y


#4. install mysql5.7
sudo rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
sudo yum install mysql-community-server -y
sudo systemctl enable mysqld
sudo systemctl start mysqld


#5. set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html 


#6. download wordpress files
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
cp -r wordpress/* /var/www/html/


#7. create the wp-config.php file
cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php


#8. edit the wp-config.php file
nano /var/www/html/wp-config.php


#9. restart the webserver
service httpd restart
[root@ip-14-0-1-200 ec2-user]# 

#1. create the html directory and mount the efs to it
#!/bin/bash

sudo su
yum update -y
mkdir -p /var/www/html
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-03c9b3354880b36a6.efs.us-east-1.amazonaws.com:/ /var/www/html
#2. install apache 
sudo yum install -y httpd httpd-tools mod_ssl
sudo systemctl enable httpd 
sudo systemctl start httpd


#3. install php 7.4
sudo amazon-linux-extras enable php7.4
sudo yum clean metadata
sudo yum install php php-common php-pear -y
sudo yum install php-{cgi,curl,mbstring,gd,mysqlnd,gettext,json,xml,fpm,intl,zip} -y


#4. install mysql5.7
sudo rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
sudo yum install mysql-community-server -y
sudo systemctl enable mysqld
sudo systemctl start mysqld


#5. set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html 


#6. download wordpress files
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
cp -r wordpress/* /var/www/html/


#7. create the wp-config.php file
cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php


#8. edit the wp-config.php file
nano /var/www/html/wp-config.php


#9. restart the webserver
service httpd restart
see feeback;


[ec2-user@ip-14-0-1-200 ~]$ sudo su
[root@ip-14-0-1-200 ec2-user]# chmod +x wordpress.sh
[root@ip-14-0-1-200 ec2-user]# ./wordpress.sh
./wordpress.sh: line 1: !#/bin/bash: No such file or directory
[root@ip-14-0-1-200 ec2-user]# ls
wordpress.sh
[root@ip-14-0-1-200 ec2-user]# cat wordpress.sh
!#/bin/bash

#1. create the html directory and mount the efs to it
sudo su
yum update -y
mkdir -p /var/www/html
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-03c9b3354880b36a6.efs.us-east-1.amazonaws.com:/ /var/www/html


#2. install apache 
sudo yum install -y httpd httpd-tools mod_ssl
sudo systemctl enable httpd 
sudo systemctl start httpd


#3. install php 7.4
sudo amazon-linux-extras enable php7.4
sudo yum clean metadata
sudo yum install php php-common php-pear -y
sudo yum install php-{cgi,curl,mbstring,gd,mysqlnd,gettext,json,xml,fpm,intl,zip} -y


#4. install mysql5.7
sudo rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
sudo yum install mysql-community-server -y
sudo systemctl enable mysqld
sudo systemctl start mysqld


#5. set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html 


#6. download wordpress files
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
cp -r wordpress/* /var/www/html/


#7. create the wp-config.php file
cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php


#8. edit the wp-config.php file
nano /var/www/html/wp-config.php


#9. restart the webserver
service httpd restart
[root@ip-14-0-1-200 ec2-user]# 
[root@ip-14-0-1-200 ec2-user]# ./wordpress.sh
./wordpress.sh: line 1: !#/bin/bash: No such file or directory
[root@ip-14-0-1-200 ec2-user]# vi wordpress.sh
[root@ip-14-0-1-200 ec2-user]# ./wordpress
bash: ./wordpress: No such file or directory
[root@ip-14-0-1-200 ec2-user]# ls
wordpress.sh
[root@ip-14-0-1-200 ec2-user]# cat wordpress.sh
!#/bin/bash

sudo su
yum update -y
mkdir -p /var/www/html
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-03c9b3354880b36a6.efs.us-east-1.amazonaws.com:/ /var/www/html

[root@ip-14-0-1-200 ec2-user]# vi wordpress.sh
[root@ip-14-0-1-200 ec2-user]# ./wordpress
bash: ./wordpress: No such file or directory
[root@ip-14-0-1-200 ec2-user]# 



[root@ip-14-0-1-200 ec2-user]# rm -rf wordpress.sh
[root@ip-14-0-1-200 ec2-user]# ls
[root@ip-14-0-1-200 ec2-user]# vi wordpress.sh
[root@ip-14-0-1-200 ec2-user]# ./wordpress.sh
bash: ./wordpress.sh: Permission denied
[root@ip-14-0-1-200 ec2-user]# chmod +x wordpress.sh
[root@ip-14-0-1-200 ec2-user]# ./wordpress.sh
[root@ip-14-0-1-200 ec2-user]# 
[root@ip-14-0-1-200 ec2-user]# vi wordpress.sh
[root@ip-14-0-1-200 ec2-user]# ./wordpress.sh
Last metadata expiration check: 2:57:55 ago on Mon May  6 19:10:50 2024.
Dependencies resolved.
================================================================================
 Package               Arch     Version                     Repository     Size
================================================================================
Installing:
 httpd                 x86_64   2.4.59-2.amzn2023           amazonlinux    47 k
 mod_ssl               x86_64   1:2.4.59-2.amzn2023         amazonlinux   112 k
Installing dependencies:
 apr                   x86_64   1.7.2-2.amzn2023.0.2        amazonlinux   129 k
 apr-util              x86_64   1.6.3-1.amzn2023.0.1        amazonlinux    98 k
 generic-logos-httpd   noarch   18.0.0-12.amzn2023.0.3      amazonlinux    19 k
 httpd-core            x86_64   2.4.59-2.amzn2023           amazonlinux   1.4 M
 httpd-filesystem      noarch   2.4.59-2.amzn2023           amazonlinux    14 k
 httpd-tools           x86_64   2.4.59-2.amzn2023           amazonlinux    81 k
 libbrotli             x86_64   1.0.9-4.amzn2023.0.2        amazonlinux   315 k
 mailcap               noarch   2.1.49-3.amzn2023.0.3       amazonlinux    33 k
 sscg                  x86_64   3.0.3-76.amzn2023           amazonlinux    45 k
Installing weak dependencies:
 apr-util-openssl      x86_64   1.6.3-1.amzn2023.0.1        amazonlinux    17 k
 mod_http2             x86_64   2.0.27-1.amzn2023.0.2       amazonlinux   166 k
 mod_lua               x86_64   2.4.59-2.amzn2023           amazonlinux    61 k

Transaction Summary
================================================================================
Install  14 Packages

Total download size: 2.5 M
Installed size: 7.3 M
Downloading Packages:
(1/14): apr-util-openssl-1.6.3-1.amzn2023.0.1.x 278 kB/s |  17 kB     00:00    
(2/14): apr-1.7.2-2.amzn2023.0.2.x86_64.rpm     1.9 MB/s | 129 kB     00:00    
(3/14): apr-util-1.6.3-1.amzn2023.0.1.x86_64.rp 1.3 MB/s |  98 kB     00:00    
(4/14): generic-logos-httpd-18.0.0-12.amzn2023. 1.2 MB/s |  19 kB     00:00    
(5/14): httpd-2.4.59-2.amzn2023.x86_64.rpm      2.9 MB/s |  47 kB     00:00    
(6/14): httpd-filesystem-2.4.59-2.amzn2023.noar 762 kB/s |  14 kB     00:00    
(7/14): httpd-tools-2.4.59-2.amzn2023.x86_64.rp 2.6 MB/s |  81 kB     00:00    
(8/14): httpd-core-2.4.59-2.amzn2023.x86_64.rpm  24 MB/s | 1.4 MB     00:00    
(9/14): libbrotli-1.0.9-4.amzn2023.0.2.x86_64.r 7.3 MB/s | 315 kB     00:00    
(10/14): mailcap-2.1.49-3.amzn2023.0.3.noarch.r 1.2 MB/s |  33 kB     00:00    
(11/14): mod_lua-2.4.59-2.amzn2023.x86_64.rpm   3.1 MB/s |  61 kB     00:00    
(12/14): mod_ssl-2.4.59-2.amzn2023.x86_64.rpm   5.0 MB/s | 112 kB     00:00    
(13/14): mod_http2-2.0.27-1.amzn2023.0.2.x86_64 5.3 MB/s | 166 kB     00:00    
(14/14): sscg-3.0.3-76.amzn2023.x86_64.rpm      2.2 MB/s |  45 kB     00:00    
--------------------------------------------------------------------------------
Total                                            10 MB/s | 2.5 MB     00:00     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : apr-1.7.2-2.amzn2023.0.2.x86_64                       1/14 
  Installing       : apr-util-openssl-1.6.3-1.amzn2023.0.1.x86_64          2/14 
  Installing       : apr-util-1.6.3-1.amzn2023.0.1.x86_64                  3/14 
  Installing       : mailcap-2.1.49-3.amzn2023.0.3.noarch                  4/14 
  Running scriptlet: httpd-filesystem-2.4.59-2.amzn2023.noarch             5/14 
  Installing       : httpd-filesystem-2.4.59-2.amzn2023.noarch             5/14 
  Installing       : httpd-tools-2.4.59-2.amzn2023.x86_64                  6/14 
  Installing       : httpd-core-2.4.59-2.amzn2023.x86_64                   7/14 
  Installing       : mod_http2-2.0.27-1.amzn2023.0.2.x86_64                8/14 
  Installing       : mod_lua-2.4.59-2.amzn2023.x86_64                      9/14 
  Installing       : sscg-3.0.3-76.amzn2023.x86_64                        10/14 
  Installing       : libbrotli-1.0.9-4.amzn2023.0.2.x86_64                11/14 
  Installing       : generic-logos-httpd-18.0.0-12.amzn2023.0.3.noarch    12/14 
  Installing       : httpd-2.4.59-2.amzn2023.x86_64                       13/14 
  Running scriptlet: httpd-2.4.59-2.amzn2023.x86_64                       13/14 
  Installing       : mod_ssl-1:2.4.59-2.amzn2023.x86_64                   14/14 
  Running scriptlet: httpd-2.4.59-2.amzn2023.x86_64                       14/14 
  Running scriptlet: mod_ssl-1:2.4.59-2.amzn2023.x86_64                   14/14 
  Verifying        : apr-1.7.2-2.amzn2023.0.2.x86_64                       1/14 
  Verifying        : apr-util-1.6.3-1.amzn2023.0.1.x86_64                  2/14 
  Verifying        : apr-util-openssl-1.6.3-1.amzn2023.0.1.x86_64          3/14 
  Verifying        : generic-logos-httpd-18.0.0-12.amzn2023.0.3.noarch     4/14 
  Verifying        : httpd-2.4.59-2.amzn2023.x86_64                        5/14 
  Verifying        : httpd-core-2.4.59-2.amzn2023.x86_64                   6/14 
  Verifying        : httpd-filesystem-2.4.59-2.amzn2023.noarch             7/14 
  Verifying        : httpd-tools-2.4.59-2.amzn2023.x86_64                  8/14 
  Verifying        : libbrotli-1.0.9-4.amzn2023.0.2.x86_64                 9/14 
  Verifying        : mailcap-2.1.49-3.amzn2023.0.3.noarch                 10/14 
  Verifying        : mod_http2-2.0.27-1.amzn2023.0.2.x86_64               11/14 
  Verifying        : mod_lua-2.4.59-2.amzn2023.x86_64                     12/14 
  Verifying        : mod_ssl-1:2.4.59-2.amzn2023.x86_64                   13/14 
  Verifying        : sscg-3.0.3-76.amzn2023.x86_64                        14/14 

Installed:
  apr-1.7.2-2.amzn2023.0.2.x86_64                                               
  apr-util-1.6.3-1.amzn2023.0.1.x86_64                                          
  apr-util-openssl-1.6.3-1.amzn2023.0.1.x86_64                                  
  generic-logos-httpd-18.0.0-12.amzn2023.0.3.noarch                             
  httpd-2.4.59-2.amzn2023.x86_64                                                
  httpd-core-2.4.59-2.amzn2023.x86_64                                           
  httpd-filesystem-2.4.59-2.amzn2023.noarch                                     
  httpd-tools-2.4.59-2.amzn2023.x86_64                                          
  libbrotli-1.0.9-4.amzn2023.0.2.x86_64                                         
  mailcap-2.1.49-3.amzn2023.0.3.noarch                                          
  mod_http2-2.0.27-1.amzn2023.0.2.x86_64                                        
  mod_lua-2.4.59-2.amzn2023.x86_64                                              
  mod_ssl-1:2.4.59-2.amzn2023.x86_64                                            
  sscg-3.0.3-76.amzn2023.x86_64                                                 

Complete!
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
sudo: amazon-linux-extras: command not found
Cache was expired
17 files removed
Amazon Linux 2023 repository                     33 MB/s |  24 MB     00:00    
Amazon Linux 2023 Kernel Livepatch repository   540 kB/s | 165 kB     00:00    
Dependencies resolved.
================================================================================
 Package             Arch      Version                     Repository      Size
================================================================================
Installing:
 php-pear            noarch    1:1.10.13-2.amzn2023.0.4    amazonlinux    309 k
 php8.2              x86_64    8.2.15-1.amzn2023.0.2       amazonlinux     11 k
Installing dependencies:
 libsodium           x86_64    1.0.19-4.amzn2023           amazonlinux    176 k
 libxslt             x86_64    1.1.34-5.amzn2023.0.2       amazonlinux    241 k
 nginx-filesystem    noarch    1:1.24.0-1.amzn2023.0.2     amazonlinux    9.1 k
 php8.2-cli          x86_64    8.2.15-1.amzn2023.0.2       amazonlinux    3.6 M
 php8.2-common       x86_64    8.2.15-1.amzn2023.0.2       amazonlinux    703 k
 php8.2-process      x86_64    8.2.15-1.amzn2023.0.2       amazonlinux     44 k
 php8.2-xml          x86_64    8.2.15-1.amzn2023.0.2       amazonlinux    148 k
Installing weak dependencies:
 php8.2-fpm          x86_64    8.2.15-1.amzn2023.0.2       amazonlinux    1.9 M
 php8.2-mbstring     x86_64    8.2.15-1.amzn2023.0.2       amazonlinux    524 k
 php8.2-opcache      x86_64    8.2.15-1.amzn2023.0.2       amazonlinux    379 k
 php8.2-pdo          x86_64    8.2.15-1.amzn2023.0.2       amazonlinux     89 k
 php8.2-sodium       x86_64    8.2.15-1.amzn2023.0.2       amazonlinux     42 k

Transaction Summary
================================================================================
Install  14 Packages

Total download size: 8.1 M
Installed size: 39 M
Downloading Packages:
(1/14): libsodium-1.0.19-4.amzn2023.x86_64.rpm  1.7 MB/s | 176 kB     00:00    
(2/14): nginx-filesystem-1.24.0-1.amzn2023.0.2.  81 kB/s | 9.1 kB     00:00    
(3/14): php8.2-8.2.15-1.amzn2023.0.2.x86_64.rpm 688 kB/s |  11 kB     00:00    
(4/14): libxslt-1.1.34-5.amzn2023.0.2.x86_64.rp 1.7 MB/s | 241 kB     00:00    
(5/14): php8.2-cli-8.2.15-1.amzn2023.0.2.x86_64  41 MB/s | 3.6 MB     00:00    
(6/14): php-pear-1.10.13-2.amzn2023.0.4.noarch. 2.4 MB/s | 309 kB     00:00    
(7/14): php8.2-common-8.2.15-1.amzn2023.0.2.x86 6.6 MB/s | 703 kB     00:00    
(8/14): php8.2-fpm-8.2.15-1.amzn2023.0.2.x86_64  30 MB/s | 1.9 MB     00:00    
(9/14): php8.2-mbstring-8.2.15-1.amzn2023.0.2.x 8.0 MB/s | 524 kB     00:00    
(10/14): php8.2-opcache-8.2.15-1.amzn2023.0.2.x 6.6 MB/s | 379 kB     00:00    
(11/14): php8.2-pdo-8.2.15-1.amzn2023.0.2.x86_6 3.0 MB/s |  89 kB     00:00    
(12/14): php8.2-sodium-8.2.15-1.amzn2023.0.2.x8 2.7 MB/s |  42 kB     00:00    
(13/14): php8.2-process-8.2.15-1.amzn2023.0.2.x 1.0 MB/s |  44 kB     00:00    
(14/14): php8.2-xml-8.2.15-1.amzn2023.0.2.x86_6 5.7 MB/s | 148 kB     00:00    
--------------------------------------------------------------------------------
Total                                            20 MB/s | 8.1 MB     00:00     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : php8.2-common-8.2.15-1.amzn2023.0.2.x86_64            1/14 
  Installing       : php8.2-cli-8.2.15-1.amzn2023.0.2.x86_64               2/14 
  Installing       : php8.2-process-8.2.15-1.amzn2023.0.2.x86_64           3/14 
  Installing       : php8.2-mbstring-8.2.15-1.amzn2023.0.2.x86_64          4/14 
  Installing       : php8.2-opcache-8.2.15-1.amzn2023.0.2.x86_64           5/14 
  Installing       : php8.2-pdo-8.2.15-1.amzn2023.0.2.x86_64               6/14 
  Running scriptlet: nginx-filesystem-1:1.24.0-1.amzn2023.0.2.noarch       7/14 
  Installing       : nginx-filesystem-1:1.24.0-1.amzn2023.0.2.noarch       7/14 
  Installing       : php8.2-fpm-8.2.15-1.amzn2023.0.2.x86_64               8/14 
  Running scriptlet: php8.2-fpm-8.2.15-1.amzn2023.0.2.x86_64               8/14 
  Installing       : libxslt-1.1.34-5.amzn2023.0.2.x86_64                  9/14 
  Installing       : php8.2-xml-8.2.15-1.amzn2023.0.2.x86_64              10/14 
  Installing       : libsodium-1.0.19-4.amzn2023.x86_64                   11/14 
  Installing       : php8.2-sodium-8.2.15-1.amzn2023.0.2.x86_64           12/14 
  Installing       : php8.2-8.2.15-1.amzn2023.0.2.x86_64                  13/14 
  Installing       : php-pear-1:1.10.13-2.amzn2023.0.4.noarch             14/14 
  Running scriptlet: php-pear-1:1.10.13-2.amzn2023.0.4.noarch             14/14 
  Verifying        : libsodium-1.0.19-4.amzn2023.x86_64                    1/14 
  Verifying        : libxslt-1.1.34-5.amzn2023.0.2.x86_64                  2/14 
  Verifying        : nginx-filesystem-1:1.24.0-1.amzn2023.0.2.noarch       3/14 
  Verifying        : php-pear-1:1.10.13-2.amzn2023.0.4.noarch              4/14 
  Verifying        : php8.2-8.2.15-1.amzn2023.0.2.x86_64                   5/14 
  Verifying        : php8.2-cli-8.2.15-1.amzn2023.0.2.x86_64               6/14 
  Verifying        : php8.2-common-8.2.15-1.amzn2023.0.2.x86_64            7/14 
  Verifying        : php8.2-fpm-8.2.15-1.amzn2023.0.2.x86_64               8/14 
  Verifying        : php8.2-mbstring-8.2.15-1.amzn2023.0.2.x86_64          9/14 
  Verifying        : php8.2-opcache-8.2.15-1.amzn2023.0.2.x86_64          10/14 
  Verifying        : php8.2-pdo-8.2.15-1.amzn2023.0.2.x86_64              11/14 
  Verifying        : php8.2-process-8.2.15-1.amzn2023.0.2.x86_64          12/14 
  Verifying        : php8.2-sodium-8.2.15-1.amzn2023.0.2.x86_64           13/14 
  Verifying        : php8.2-xml-8.2.15-1.amzn2023.0.2.x86_64              14/14 

Installed:
  libsodium-1.0.19-4.amzn2023.x86_64                                            
  libxslt-1.1.34-5.amzn2023.0.2.x86_64                                          
  nginx-filesystem-1:1.24.0-1.amzn2023.0.2.noarch                               
  php-pear-1:1.10.13-2.amzn2023.0.4.noarch                                      
  php8.2-8.2.15-1.amzn2023.0.2.x86_64                                           
  php8.2-cli-8.2.15-1.amzn2023.0.2.x86_64                                       
  php8.2-common-8.2.15-1.amzn2023.0.2.x86_64                                    
  php8.2-fpm-8.2.15-1.amzn2023.0.2.x86_64                                       
  php8.2-mbstring-8.2.15-1.amzn2023.0.2.x86_64                                  
  php8.2-opcache-8.2.15-1.amzn2023.0.2.x86_64                                   
  php8.2-pdo-8.2.15-1.amzn2023.0.2.x86_64                                       
  php8.2-process-8.2.15-1.amzn2023.0.2.x86_64                                   
  php8.2-sodium-8.2.15-1.amzn2023.0.2.x86_64                                    
  php8.2-xml-8.2.15-1.amzn2023.0.2.x86_64                                       

Complete!
Last metadata expiration check: 0:00:07 ago on Mon May  6 22:09:07 2024.
Package php8.2-cli-8.2.15-1.amzn2023.0.2.x86_64 is already installed.
Package php8.2-common-8.2.15-1.amzn2023.0.2.x86_64 is already installed.
Package php8.2-mbstring-8.2.15-1.amzn2023.0.2.x86_64 is already installed.
Package php8.2-common-8.2.15-1.amzn2023.0.2.x86_64 is already installed.
Package php8.2-common-8.2.15-1.amzn2023.0.2.x86_64 is already installed.
Package php8.2-xml-8.2.15-1.amzn2023.0.2.x86_64 is already installed.
Package php8.2-fpm-8.2.15-1.amzn2023.0.2.x86_64 is already installed.
Dependencies resolved.
================================================================================
 Package                   Arch   Version                     Repository   Size
================================================================================
Installing:
 php8.2-gd                 x86_64 8.2.15-1.amzn2023.0.2       amazonlinux  44 k
 php8.2-intl               x86_64 8.2.15-1.amzn2023.0.2       amazonlinux 170 k
 php8.2-mysqlnd            x86_64 8.2.15-1.amzn2023.0.2       amazonlinux 148 k
 php8.2-zip                x86_64 8.2.15-1.amzn2023.0.2       amazonlinux  40 k
Installing dependencies:
 cairo                     x86_64 1.17.6-2.amzn2023.0.1       amazonlinux 684 k
 fontconfig                x86_64 2.13.94-2.amzn2023.0.2      amazonlinux 273 k
 fonts-filesystem          noarch 1:2.0.5-12.amzn2023.0.2     amazonlinux 9.5 k
 freetype                  x86_64 2.13.0-2.amzn2023.0.1       amazonlinux 422 k
 gd                        x86_64 2.3.3-5.amzn2023.0.3        amazonlinux 139 k
 google-noto-fonts-common  noarch 20201206-2.amzn2023.0.2     amazonlinux  15 k
 google-noto-sans-vf-fonts noarch 20201206-2.amzn2023.0.2     amazonlinux 492 k
 graphite2                 x86_64 1.3.14-7.amzn2023.0.2       amazonlinux  97 k
 harfbuzz                  x86_64 7.0.0-2.amzn2023.0.1        amazonlinux 868 k
 jbigkit-libs              x86_64 2.1-21.amzn2023.0.2         amazonlinux  54 k
 langpacks-core-font-en    noarch 3.0-21.amzn2023.0.4         amazonlinux  10 k
 libX11                    x86_64 1.7.2-3.amzn2023.0.4        amazonlinux 657 k
 libX11-common             noarch 1.7.2-3.amzn2023.0.4        amazonlinux 152 k
 libXau                    x86_64 1.0.9-6.amzn2023.0.2        amazonlinux  31 k
 libXext                   x86_64 1.3.4-6.amzn2023.0.2        amazonlinux  41 k
 libXpm                    x86_64 3.5.15-2.amzn2023.0.3       amazonlinux  65 k
 libXrender                x86_64 0.9.10-14.amzn2023.0.2      amazonlinux  28 k
 libicu                    x86_64 67.1-7.amzn2023.0.3         amazonlinux 9.6 M
 libjpeg-turbo             x86_64 2.1.4-2.amzn2023.0.5        amazonlinux 190 k
 libpng                    x86_64 2:1.6.37-10.amzn2023.0.6    amazonlinux 128 k
 libtiff                   x86_64 4.4.0-4.amzn2023.0.18       amazonlinux 213 k
 libwebp                   x86_64 1.2.4-1.amzn2023.0.6        amazonlinux 341 k
 libxcb                    x86_64 1.13.1-7.amzn2023.0.2       amazonlinux 230 k
 libzip                    x86_64 1.7.3-4.amzn2023.0.3        amazonlinux  63 k
 pixman                    x86_64 0.40.0-3.amzn2023.0.3       amazonlinux 295 k
 xml-common                noarch 0.6.3-56.amzn2023.0.2       amazonlinux  32 k

Transaction Summary
================================================================================
Install  30 Packages

Total download size: 15 M
Installed size: 49 M
Downloading Packages:
(1/30): fonts-filesystem-2.0.5-12.amzn2023.0.2. 103 kB/s | 9.5 kB     00:00    
(2/30): fontconfig-2.13.94-2.amzn2023.0.2.x86_6 2.6 MB/s | 273 kB     00:00    
(3/30): gd-2.3.3-5.amzn2023.0.3.x86_64.rpm      5.3 MB/s | 139 kB     00:00    
(4/30): freetype-2.13.0-2.amzn2023.0.1.x86_64.r 8.4 MB/s | 422 kB     00:00    
(5/30): google-noto-fonts-common-20201206-2.amz 810 kB/s |  15 kB     00:00    
(6/30): cairo-1.17.6-2.amzn2023.0.1.x86_64.rpm  4.0 MB/s | 684 kB     00:00    
(7/30): graphite2-1.3.14-7.amzn2023.0.2.x86_64. 4.1 MB/s |  97 kB     00:00    
(8/30): google-noto-sans-vf-fonts-20201206-2.am  11 MB/s | 492 kB     00:00    
(9/30): jbigkit-libs-2.1-21.amzn2023.0.2.x86_64 2.3 MB/s |  54 kB     00:00    
(10/30): harfbuzz-7.0.0-2.amzn2023.0.1.x86_64.r  17 MB/s | 868 kB     00:00    
(11/30): langpacks-core-font-en-3.0-21.amzn2023 338 kB/s |  10 kB     00:00    
(12/30): libXau-1.0.9-6.amzn2023.0.2.x86_64.rpm 1.7 MB/s |  31 kB     00:00    
(13/30): libX11-common-1.7.2-3.amzn2023.0.4.noa 5.4 MB/s | 152 kB     00:00    
(14/30): libX11-1.7.2-3.amzn2023.0.4.x86_64.rpm  10 MB/s | 657 kB     00:00    
(15/30): libXext-1.3.4-6.amzn2023.0.2.x86_64.rp 1.8 MB/s |  41 kB     00:00    
(16/30): libXrender-0.9.10-14.amzn2023.0.2.x86_ 1.6 MB/s |  28 kB     00:00    
(17/30): libXpm-3.5.15-2.amzn2023.0.3.x86_64.rp 1.0 MB/s |  65 kB     00:00    
(18/30): libjpeg-turbo-2.1.4-2.amzn2023.0.5.x86 5.0 MB/s | 190 kB     00:00    
(19/30): libtiff-4.4.0-4.amzn2023.0.18.x86_64.r 5.8 MB/s | 213 kB     00:00    
(20/30): libicu-67.1-7.amzn2023.0.3.x86_64.rpm   40 MB/s | 9.6 MB     00:00    
(21/30): libpng-1.6.37-10.amzn2023.0.6.x86_64.r 637 kB/s | 128 kB     00:00    
(22/30): libwebp-1.2.4-1.amzn2023.0.6.x86_64.rp 2.1 MB/s | 341 kB     00:00    
(23/30): libzip-1.7.3-4.amzn2023.0.3.x86_64.rpm 2.9 MB/s |  63 kB     00:00    
(24/30): libxcb-1.13.1-7.amzn2023.0.2.x86_64.rp 5.1 MB/s | 230 kB     00:00    
(25/30): php8.2-gd-8.2.15-1.amzn2023.0.2.x86_64 993 kB/s |  44 kB     00:00    
(26/30): php8.2-mysqlnd-8.2.15-1.amzn2023.0.2.x 2.5 MB/s | 148 kB     00:00    
(27/30): php8.2-zip-8.2.15-1.amzn2023.0.2.x86_6 831 kB/s |  40 kB     00:00    
(28/30): php8.2-intl-8.2.15-1.amzn2023.0.2.x86_ 1.8 MB/s | 170 kB     00:00    
(29/30): xml-common-0.6.3-56.amzn2023.0.2.noarc 1.4 MB/s |  32 kB     00:00    
(30/30): pixman-0.40.0-3.amzn2023.0.3.x86_64.rp 9.0 MB/s | 295 kB     00:00    
--------------------------------------------------------------------------------
Total                                            21 MB/s |  15 MB     00:00     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : libpng-2:1.6.37-10.amzn2023.0.6.x86_64                1/30 
  Installing       : libwebp-1.2.4-1.amzn2023.0.6.x86_64                   2/30 
  Installing       : libjpeg-turbo-2.1.4-2.amzn2023.0.5.x86_64             3/30 
  Installing       : fonts-filesystem-1:2.0.5-12.amzn2023.0.2.noarch       4/30 
  Running scriptlet: xml-common-0.6.3-56.amzn2023.0.2.noarch               5/30 
  Installing       : xml-common-0.6.3-56.amzn2023.0.2.noarch               5/30 
  Installing       : pixman-0.40.0-3.amzn2023.0.3.x86_64                   6/30 
  Installing       : libzip-1.7.3-4.amzn2023.0.3.x86_64                    7/30 
  Installing       : libicu-67.1-7.amzn2023.0.3.x86_64                     8/30 
  Installing       : libXau-1.0.9-6.amzn2023.0.2.x86_64                    9/30 
  Installing       : libxcb-1.13.1-7.amzn2023.0.2.x86_64                  10/30 
  Installing       : libX11-common-1.7.2-3.amzn2023.0.4.noarch            11/30 
  Installing       : libX11-1.7.2-3.amzn2023.0.4.x86_64                   12/30 
  Installing       : libXext-1.3.4-6.amzn2023.0.2.x86_64                  13/30 
  Installing       : libXpm-3.5.15-2.amzn2023.0.3.x86_64                  14/30 
  Installing       : libXrender-0.9.10-14.amzn2023.0.2.x86_64             15/30 
  Installing       : jbigkit-libs-2.1-21.amzn2023.0.2.x86_64              16/30 
  Installing       : libtiff-4.4.0-4.amzn2023.0.18.x86_64                 17/30 
  Installing       : graphite2-1.3.14-7.amzn2023.0.2.x86_64               18/30 
  Installing       : google-noto-fonts-common-20201206-2.amzn2023.0.2.n   19/30 
  Installing       : google-noto-sans-vf-fonts-20201206-2.amzn2023.0.2.   20/30 
  Installing       : langpacks-core-font-en-3.0-21.amzn2023.0.4.noarch    21/30 
  Installing       : cairo-1.17.6-2.amzn2023.0.1.x86_64                   22/30 
  Installing       : harfbuzz-7.0.0-2.amzn2023.0.1.x86_64                 23/30 
  Installing       : freetype-2.13.0-2.amzn2023.0.1.x86_64                24/30 
  Installing       : fontconfig-2.13.94-2.amzn2023.0.2.x86_64             25/30 
  Running scriptlet: fontconfig-2.13.94-2.amzn2023.0.2.x86_64             25/30 
  Installing       : gd-2.3.3-5.amzn2023.0.3.x86_64                       26/30 
  Installing       : php8.2-gd-8.2.15-1.amzn2023.0.2.x86_64               27/30 
  Installing       : php8.2-intl-8.2.15-1.amzn2023.0.2.x86_64             28/30 
  Installing       : php8.2-zip-8.2.15-1.amzn2023.0.2.x86_64              29/30 
  Installing       : php8.2-mysqlnd-8.2.15-1.amzn2023.0.2.x86_64          30/30 
  Running scriptlet: fontconfig-2.13.94-2.amzn2023.0.2.x86_64             30/30 
  Running scriptlet: php8.2-mysqlnd-8.2.15-1.amzn2023.0.2.x86_64          30/30 
  Verifying        : cairo-1.17.6-2.amzn2023.0.1.x86_64                    1/30 
  Verifying        : fontconfig-2.13.94-2.amzn2023.0.2.x86_64              2/30 
  Verifying        : fonts-filesystem-1:2.0.5-12.amzn2023.0.2.noarch       3/30 
  Verifying        : freetype-2.13.0-2.amzn2023.0.1.x86_64                 4/30 
  Verifying        : gd-2.3.3-5.amzn2023.0.3.x86_64                        5/30 
  Verifying        : google-noto-fonts-common-20201206-2.amzn2023.0.2.n    6/30 
  Verifying        : google-noto-sans-vf-fonts-20201206-2.amzn2023.0.2.    7/30 
  Verifying        : graphite2-1.3.14-7.amzn2023.0.2.x86_64                8/30 
  Verifying        : harfbuzz-7.0.0-2.amzn2023.0.1.x86_64                  9/30 
  Verifying        : jbigkit-libs-2.1-21.amzn2023.0.2.x86_64              10/30 
  Verifying        : langpacks-core-font-en-3.0-21.amzn2023.0.4.noarch    11/30 
  Verifying        : libX11-1.7.2-3.amzn2023.0.4.x86_64                   12/30 
  Verifying        : libX11-common-1.7.2-3.amzn2023.0.4.noarch            13/30 
  Verifying        : libXau-1.0.9-6.amzn2023.0.2.x86_64                   14/30 
  Verifying        : libXext-1.3.4-6.amzn2023.0.2.x86_64                  15/30 
  Verifying        : libXpm-3.5.15-2.amzn2023.0.3.x86_64                  16/30 
  Verifying        : libXrender-0.9.10-14.amzn2023.0.2.x86_64             17/30 
  Verifying        : libicu-67.1-7.amzn2023.0.3.x86_64                    18/30 
  Verifying        : libjpeg-turbo-2.1.4-2.amzn2023.0.5.x86_64            19/30 
  Verifying        : libpng-2:1.6.37-10.amzn2023.0.6.x86_64               20/30 
  Verifying        : libtiff-4.4.0-4.amzn2023.0.18.x86_64                 21/30 
  Verifying        : libwebp-1.2.4-1.amzn2023.0.6.x86_64                  22/30 
  Verifying        : libxcb-1.13.1-7.amzn2023.0.2.x86_64                  23/30 
  Verifying        : libzip-1.7.3-4.amzn2023.0.3.x86_64                   24/30 
  Verifying        : php8.2-gd-8.2.15-1.amzn2023.0.2.x86_64               25/30 
  Verifying        : php8.2-intl-8.2.15-1.amzn2023.0.2.x86_64             26/30 
  Verifying        : php8.2-mysqlnd-8.2.15-1.amzn2023.0.2.x86_64          27/30 
  Verifying        : php8.2-zip-8.2.15-1.amzn2023.0.2.x86_64              28/30 
  Verifying        : pixman-0.40.0-3.amzn2023.0.3.x86_64                  29/30 
  Verifying        : xml-common-0.6.3-56.amzn2023.0.2.noarch              30/30 

Installed:
  cairo-1.17.6-2.amzn2023.0.1.x86_64                                            
  fontconfig-2.13.94-2.amzn2023.0.2.x86_64                                      
  fonts-filesystem-1:2.0.5-12.amzn2023.0.2.noarch                               
  freetype-2.13.0-2.amzn2023.0.1.x86_64                                         
  gd-2.3.3-5.amzn2023.0.3.x86_64                                                
  google-noto-fonts-common-20201206-2.amzn2023.0.2.noarch                       
  google-noto-sans-vf-fonts-20201206-2.amzn2023.0.2.noarch                      
  graphite2-1.3.14-7.amzn2023.0.2.x86_64                                        
  harfbuzz-7.0.0-2.amzn2023.0.1.x86_64                                          
  jbigkit-libs-2.1-21.amzn2023.0.2.x86_64                                       
  langpacks-core-font-en-3.0-21.amzn2023.0.4.noarch                             
  libX11-1.7.2-3.amzn2023.0.4.x86_64                                            
  libX11-common-1.7.2-3.amzn2023.0.4.noarch                                     
  libXau-1.0.9-6.amzn2023.0.2.x86_64                                            
  libXext-1.3.4-6.amzn2023.0.2.x86_64                                           
  libXpm-3.5.15-2.amzn2023.0.3.x86_64                                           
  libXrender-0.9.10-14.amzn2023.0.2.x86_64                                      
  libicu-67.1-7.amzn2023.0.3.x86_64                                             
  libjpeg-turbo-2.1.4-2.amzn2023.0.5.x86_64                                     
  libpng-2:1.6.37-10.amzn2023.0.6.x86_64                                        
  libtiff-4.4.0-4.amzn2023.0.18.x86_64                                          
  libwebp-1.2.4-1.amzn2023.0.6.x86_64                                           
  libxcb-1.13.1-7.amzn2023.0.2.x86_64                                           
  libzip-1.7.3-4.amzn2023.0.3.x86_64                                            
  php8.2-gd-8.2.15-1.amzn2023.0.2.x86_64                                        
  php8.2-intl-8.2.15-1.amzn2023.0.2.x86_64                                      
  php8.2-mysqlnd-8.2.15-1.amzn2023.0.2.x86_64                                   
  php8.2-zip-8.2.15-1.amzn2023.0.2.x86_64                                       
  pixman-0.40.0-3.amzn2023.0.3.x86_64                                           
  xml-common-0.6.3-56.amzn2023.0.2.noarch                                       

Complete!
Retrieving https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
warning: /var/tmp/rpm-tmp.MU4r7p: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
Verifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql57-community-release-el7-11 ################################# [100%]
MySQL Connectors Community                      1.5 MB/s |  72 kB     00:00    
MySQL Tools Community                           6.6 MB/s | 1.2 MB     00:00    
MySQL 5.7 Community Server                       26 MB/s | 3.1 MB     00:00    
Dependencies resolved.
================================================================================
 Package                Arch   Version                  Repository         Size
================================================================================
Installing:
 mysql-community-server x86_64 5.7.44-1.el7             mysql57-community 184 M
Installing dependencies:
 libxcrypt-compat       x86_64 4.4.33-7.amzn2023        amazonlinux        92 k
 mysql-community-client x86_64 5.7.44-1.el7             mysql57-community  31 M
 mysql-community-common x86_64 5.7.44-1.el7             mysql57-community 313 k
 mysql-community-libs   x86_64 5.7.44-1.el7             mysql57-community 3.0 M
 ncurses-compat-libs    x86_64 6.2-4.20200222.amzn2023.0.6
                                                        amazonlinux       323 k

Transaction Summary
================================================================================
Install  6 Packages

Total download size: 219 M
Installed size: 931 M
Downloading Packages:
(1/6): libxcrypt-compat-4.4.33-7.amzn2023.x86_6 1.3 MB/s |  92 kB     00:00    
(2/6): ncurses-compat-libs-6.2-4.20200222.amzn2 3.1 MB/s | 323 kB     00:00    
(3/6): mysql-community-common-5.7.44-1.el7.x86_ 4.2 MB/s | 313 kB     00:00    
(4/6): mysql-community-libs-5.7.44-1.el7.x86_64  15 MB/s | 3.0 MB     00:00    
(5/6): mysql-community-client-5.7.44-1.el7.x86_  36 MB/s |  31 MB     00:00    
(6/6): mysql-community-server-5.7.44-1.el7.x86_  60 MB/s | 184 MB     00:03    
--------------------------------------------------------------------------------
Total                                            67 MB/s | 219 MB     00:03     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : mysql-community-common-5.7.44-1.el7.x86_64             1/6 
  Installing       : mysql-community-libs-5.7.44-1.el7.x86_64               2/6 
  Running scriptlet: mysql-community-libs-5.7.44-1.el7.x86_64               2/6 
  Installing       : ncurses-compat-libs-6.2-4.20200222.amzn2023.0.6.x86_   3/6 
  Installing       : mysql-community-client-5.7.44-1.el7.x86_64             4/6 
  Installing       : libxcrypt-compat-4.4.33-7.amzn2023.x86_64              5/6 
  Running scriptlet: mysql-community-server-5.7.44-1.el7.x86_64             6/6 
  Installing       : mysql-community-server-5.7.44-1.el7.x86_64             6/6 
  Running scriptlet: mysql-community-server-5.7.44-1.el7.x86_64             6/6 
/usr/lib/tmpfiles.d/mysql.conf:23: Line references path below legacy directory /var/run/, updating /var/run/mysqld → /run/mysqld; please update the tmpfiles.d/ drop-in file accordingly.

  Verifying        : libxcrypt-compat-4.4.33-7.amzn2023.x86_64              1/6 
  Verifying        : ncurses-compat-libs-6.2-4.20200222.amzn2023.0.6.x86_   2/6 
  Verifying        : mysql-community-client-5.7.44-1.el7.x86_64             3/6 
  Verifying        : mysql-community-common-5.7.44-1.el7.x86_64             4/6 
  Verifying        : mysql-community-libs-5.7.44-1.el7.x86_64               5/6 
  Verifying        : mysql-community-server-5.7.44-1.el7.x86_64             6/6 

Installed:
  libxcrypt-compat-4.4.33-7.amzn2023.x86_64                                     
  mysql-community-client-5.7.44-1.el7.x86_64                                    
  mysql-community-common-5.7.44-1.el7.x86_64                                    
  mysql-community-libs-5.7.44-1.el7.x86_64                                      
  mysql-community-server-5.7.44-1.el7.x86_64                                    
  ncurses-compat-libs-6.2-4.20200222.amzn2023.0.6.x86_64                        

Complete!
--2024-05-06 22:10:19--  https://wordpress.org/latest.tar.gz
Resolving wordpress.org (wordpress.org)... 198.143.164.252
Connecting to wordpress.org (wordpress.org)|198.143.164.252|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 24697732 (24M) [application/octet-stream]
Saving to: ‘latest.tar.gz’

latest.tar.gz       100%[===================>]  23.55M  25.9MB/s    in 0.9s    

2024-05-06 22:10:20 (25.9 MB/s) - ‘latest.tar.gz’ saved [24697732/24697732]

Redirecting to /bin/systemctl restart httpd.service
[root@ip-14-0-1-200 ec2-user]# 

5. to create application load balancer;
•	I clicked on create application load balancer. I chose the VPC and two subnets I created. It will listen to the target group, so I created the target group I included the two instances as pending. After creating my ALB it was in provisioning state. I waited for it provision.

Summary
Review and confirm your configurations. Estimate cost 
Basic configuration
Edit
wps-alb
•	Internet-facing
•	IPv4
Security groups
Edit
•	default
sg-0bb0900f986b07329 
db-server-sg
sg-09cc30be789c6949e 
myweb-server-sg
sg-0d62763220953a26a 
Network mapping
Edit
VPC
vpc-0fafdd692994fe750 
my-wps-vpc
•	us-east-1a
subnet-0f8f4c3353d0dc7ee 
•  my-wps-sbn-public-1a
•  us-east-1b
subnet-0cdbfd4020c7b8aad 
•	my-wps-sbn-private-1b
Listeners and routing
Edit
•	HTTP:80 
defaults to
wps-tg 


[ec2-user@ip-14-0-1-200 ~]$ sudo systemctl enable httpd
[ec2-user@ip-14-0-1-200 ~]$ sudo systemctl start httpd

 /home/mobaxterm/desktop 
  07/05/2024   14:36.39  chmod 400 "wordpress-key.pem"
                                                                               ✓

  /home/mobaxterm/desktop 
  07/05/2024   14:38.46  ssh -i "wordpress-key.pem" ec2-user@18.209.164.195
X11 forwarding request failed on channel 0
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
Last login: Tue May  7 13:29:53 2024 from 197.210.71.234
[ec2-user@ip-14-0-2-14 ~]$
[ec2-user@ip-14-0-2-14 ~]$ nano app-key
[ec2-user@ip-14-0-2-14 ~]$ chom app-key
-bash: chom: command not found
[ec2-user@ip-14-0-2-14 ~]$ chmod app-key
chmod: missing operand after ‘app-key’
Try 'chmod --help' for more information.
[ec2-user@ip-14-0-2-14 ~]$ chmod 400 app-key
[ec2-user@ip-14-0-2-14 ~]$ ssh -i "wordpress-key.pem" ec2-user@14.0.3.21
Warning: Identity file wordpress-key.pem not accessible: No such file or directory.
The authenticity of host '14.0.3.21 (14.0.3.21)' can't be established.
ED25519 key fingerprint is SHA256:5eblB5hG4M1WncO8iEmk+z0r/X8w9JxOGp94A7bZR9A.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '14.0.3.21' (ED25519) to the list of known hosts.
ec2-user@14.0.3.21: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
[ec2-user@ip-14-0-2-14 ~]$ ssh -i app-key ec2-user@14.0.3.21
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
[ec2-user@ip-14-0-3-21 ~]$


sojy@OlasojiUbuntu:~$ cd Downloads
sojy@OlasojiUbuntu:~/Downloads$ ssh -i "wordpress-key.pem" ec2-user@ec2-54-209-39-201.compute-1.amazonaws.com
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~&apos; &apos;->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/&apos;
Last login: Wed May  8 16:03:39 2024 from 102.91.5.209
[ec2-user@ip-14-0-1-75 ~]$ sudo su
[root@ip-14-0-1-75 ec2-user]# vi webserver.sh
[root@ip-14-0-1-75 ec2-user]# chmod +x webserver.sh
[root@ip-14-0-1-75 ec2-user]# ./webserver.sh
Last metadata expiration check: 1:38:12 ago on Wed May  8 14:47:05 2024.
Dependencies resolved.
Nothing to do.
Complete!
Last metadata expiration check: 1:38:12 ago on Wed May  8 14:47:05 2024.
Dependencies resolved.
================================================================================
 Package               Arch     Version                     Repository     Size
================================================================================
Installing:
 httpd                 x86_64   2.4.59-2.amzn2023           amazonlinux    47 k
 mod_ssl               x86_64   1:2.4.59-2.amzn2023         amazonlinux   112 k
Installing dependencies:
 apr                   x86_64   1.7.2-2.amzn2023.0.2        amazonlinux   129 k
 apr-util              x86_64   1.6.3-1.amzn2023.0.1        amazonlinux    98 k
 generic-logos-httpd   noarch   18.0.0-12.amzn2023.0.3      amazonlinux    19 k
 httpd-core            x86_64   2.4.59-2.amzn2023           amazonlinux   1.4 M
 httpd-filesystem      noarch   2.4.59-2.amzn2023           amazonlinux    14 k
 httpd-tools           x86_64   2.4.59-2.amzn2023           amazonlinux    81 k
 libbrotli             x86_64   1.0.9-4.amzn2023.0.2        amazonlinux   315 k
 mailcap               noarch   2.1.49-3.amzn2023.0.3       amazonlinux    33 k
 sscg                  x86_64   3.0.3-76.amzn2023           amazonlinux    45 k
Installing weak dependencies:
 apr-util-openssl      x86_64   1.6.3-1.amzn2023.0.1        amazonlinux    17 k
 mod_http2             x86_64   2.0.27-1.amzn2023.0.2       amazonlinux   166 k
 mod_lua               x86_64   2.4.59-2.amzn2023           amazonlinux    61 k

Transaction Summary
================================================================================
Install  14 Packages

Total download size: 2.5 M
Installed size: 7.3 M
Downloading Packages:
(1/14): apr-util-openssl-1.6.3-1.amzn2023.0.1.x 295 kB/s |  17 kB     00:00    
(2/14): apr-1.7.2-2.amzn2023.0.2.x86_64.rpm     1.7 MB/s | 129 kB     00:00    
(3/14): generic-logos-httpd-18.0.0-12.amzn2023. 1.0 MB/s |  19 kB     00:00    
(4/14): apr-util-1.6.3-1.amzn2023.0.1.x86_64.rp 1.2 MB/s |  98 kB     00:00    
(5/14): httpd-2.4.59-2.amzn2023.x86_64.rpm      3.3 MB/s |  47 kB     00:00    
(6/14): httpd-filesystem-2.4.59-2.amzn2023.noar 744 kB/s |  14 kB     00:00    
(7/14): httpd-tools-2.4.59-2.amzn2023.x86_64.rp 3.4 MB/s |  81 kB     00:00    
(8/14): httpd-core-2.4.59-2.amzn2023.x86_64.rpm  28 MB/s | 1.4 MB     00:00    
(9/14): mailcap-2.1.49-3.amzn2023.0.3.noarch.rp 1.8 MB/s |  33 kB     00:00    
(10/14): libbrotli-1.0.9-4.amzn2023.0.2.x86_64. 8.7 MB/s | 315 kB     00:00    
(11/14): mod_lua-2.4.59-2.amzn2023.x86_64.rpm   3.7 MB/s |  61 kB     00:00    
(12/14): mod_http2-2.0.27-1.amzn2023.0.2.x86_64 6.8 MB/s | 166 kB     00:00    
(13/14): mod_ssl-2.4.59-2.amzn2023.x86_64.rpm   3.6 MB/s | 112 kB     00:00    
(14/14): sscg-3.0.3-76.amzn2023.x86_64.rpm      636 kB/s |  45 kB     00:00    
--------------------------------------------------------------------------------
Total                                           8.0 MB/s | 2.5 MB     00:00     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : apr-1.7.2-2.amzn2023.0.2.x86_64                       1/14 
  Installing       : apr-util-openssl-1.6.3-1.amzn2023.0.1.x86_64          2/14 
  Installing       : apr-util-1.6.3-1.amzn2023.0.1.x86_64                  3/14 
  Installing       : mailcap-2.1.49-3.amzn2023.0.3.noarch                  4/14 
  Running scriptlet: httpd-filesystem-2.4.59-2.amzn2023.noarch             5/14 
  Installing       : httpd-filesystem-2.4.59-2.amzn2023.noarch             5/14 
  Installing       : httpd-tools-2.4.59-2.amzn2023.x86_64                  6/14 
  Installing       : httpd-core-2.4.59-2.amzn2023.x86_64                   7/14 
  Installing       : mod_http2-2.0.27-1.amzn2023.0.2.x86_64                8/14 
  Installing       : mod_lua-2.4.59-2.amzn2023.x86_64                      9/14 
  Installing       : sscg-3.0.3-76.amzn2023.x86_64                        10/14 
  Installing       : libbrotli-1.0.9-4.amzn2023.0.2.x86_64                11/14 
  Installing       : generic-logos-httpd-18.0.0-12.amzn2023.0.3.noarch    12/14 
  Installing       : httpd-2.4.59-2.amzn2023.x86_64                       13/14 
  Running scriptlet: httpd-2.4.59-2.amzn2023.x86_64                       13/14 
  Installing       : mod_ssl-1:2.4.59-2.amzn2023.x86_64                   14/14 
  Running scriptlet: httpd-2.4.59-2.amzn2023.x86_64                       14/14 
  Running scriptlet: mod_ssl-1:2.4.59-2.amzn2023.x86_64                   14/14 
  Verifying        : apr-1.7.2-2.amzn2023.0.2.x86_64                       1/14 
  Verifying        : apr-util-1.6.3-1.amzn2023.0.1.x86_64                  2/14 
  Verifying        : apr-util-openssl-1.6.3-1.amzn2023.0.1.x86_64          3/14 
  Verifying        : generic-logos-httpd-18.0.0-12.amzn2023.0.3.noarch     4/14 
  Verifying        : httpd-2.4.59-2.amzn2023.x86_64                        5/14 
  Verifying        : httpd-core-2.4.59-2.amzn2023.x86_64                   6/14 
  Verifying        : httpd-filesystem-2.4.59-2.amzn2023.noarch             7/14 
  Verifying        : httpd-tools-2.4.59-2.amzn2023.x86_64                  8/14 
  Verifying        : libbrotli-1.0.9-4.amzn2023.0.2.x86_64                 9/14 
  Verifying        : mailcap-2.1.49-3.amzn2023.0.3.noarch                 10/14 
  Verifying        : mod_http2-2.0.27-1.amzn2023.0.2.x86_64               11/14 
  Verifying        : mod_lua-2.4.59-2.amzn2023.x86_64                     12/14 
  Verifying        : mod_ssl-1:2.4.59-2.amzn2023.x86_64                   13/14 
  Verifying        : sscg-3.0.3-76.amzn2023.x86_64                        14/14 

Installed:
  apr-1.7.2-2.amzn2023.0.2.x86_64                                               
  apr-util-1.6.3-1.amzn2023.0.1.x86_64                                          
  apr-util-openssl-1.6.3-1.amzn2023.0.1.x86_64                                  
  generic-logos-httpd-18.0.0-12.amzn2023.0.3.noarch                             
  httpd-2.4.59-2.amzn2023.x86_64                                                
  httpd-core-2.4.59-2.amzn2023.x86_64                                           
  httpd-filesystem-2.4.59-2.amzn2023.noarch                                     
  httpd-tools-2.4.59-2.amzn2023.x86_64                                          
  libbrotli-1.0.9-4.amzn2023.0.2.x86_64                                         
  mailcap-2.1.49-3.amzn2023.0.3.noarch                                          
  mod_http2-2.0.27-1.amzn2023.0.2.x86_64                                        
  mod_lua-2.4.59-2.amzn2023.x86_64                                              
  mod_ssl-1:2.4.59-2.amzn2023.x86_64                                            
  sscg-3.0.3-76.amzn2023.x86_64                                                 

Complete!
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
sudo: amazon-linux-extras: command not found
Cache was expired
17 files removed
Amazon Linux 2023 repository                     35 MB/s |  24 MB     00:00    
Amazon Linux 2023 Kernel Livepatch repository   552 kB/s | 165 kB     00:00    
Dependencies resolved.
================================================================================
 Package             Arch      Version                     Repository      Size
================================================================================
Installing:
 php-pear            noarch    1:1.10.13-2.amzn2023.0.4    amazonlinux    309 k
 php8.2              x86_64    8.2.15-1.amzn2023.0.2       amazonlinux     11 k
Installing dependencies:
 libsodium           x86_64    1.0.19-4.amzn2023           amazonlinux    176 k
 libxslt             x86_64    1.1.34-5.amzn2023.0.2       amazonlinux    241 k
 nginx-filesystem    noarch    1:1.24.0-1.amzn2023.0.2     amazonlinux    9.1 k
 php8.2-cli          x86_64    8.2.15-1.amzn2023.0.2       amazonlinux    3.6 M
 php8.2-common       x86_64    8.2.15-1.amzn2023.0.2       amazonlinux    703 k
 php8.2-process      x86_64    8.2.15-1.amzn2023.0.2       amazonlinux     44 k
 php8.2-xml          x86_64    8.2.15-1.amzn2023.0.2       amazonlinux    148 k
Installing weak dependencies:
 php8.2-fpm          x86_64    8.2.15-1.amzn2023.0.2       amazonlinux    1.9 M
 php8.2-mbstring     x86_64    8.2.15-1.amzn2023.0.2       amazonlinux    524 k
 php8.2-opcache      x86_64    8.2.15-1.amzn2023.0.2       amazonlinux    379 k
 php8.2-pdo          x86_64    8.2.15-1.amzn2023.0.2       amazonlinux     89 k
 php8.2-sodium       x86_64    8.2.15-1.amzn2023.0.2       amazonlinux     42 k

Transaction Summary
================================================================================
Install  14 Packages

Total download size: 8.1 M
Installed size: 39 M
Downloading Packages:
(1/14): libsodium-1.0.19-4.amzn2023.x86_64.rpm  1.9 MB/s | 176 kB     00:00    
(2/14): libxslt-1.1.34-5.amzn2023.0.2.x86_64.rp 2.3 MB/s | 241 kB     00:00    
(3/14): nginx-filesystem-1.24.0-1.amzn2023.0.2.  88 kB/s | 9.1 kB     00:00    
(4/14): php8.2-8.2.15-1.amzn2023.0.2.x86_64.rpm 696 kB/s |  11 kB     00:00    
(5/14): php8.2-common-8.2.15-1.amzn2023.0.2.x86  15 MB/s | 703 kB     00:00    
(6/14): php8.2-cli-8.2.15-1.amzn2023.0.2.x86_64  36 MB/s | 3.6 MB     00:00    
(7/14): php-pear-1.10.13-2.amzn2023.0.4.noarch. 2.5 MB/s | 309 kB     00:00    
(8/14): php8.2-fpm-8.2.15-1.amzn2023.0.2.x86_64  26 MB/s | 1.9 MB     00:00    
(9/14): php8.2-opcache-8.2.15-1.amzn2023.0.2.x8  11 MB/s | 379 kB     00:00    
(10/14): php8.2-mbstring-8.2.15-1.amzn2023.0.2.  10 MB/s | 524 kB     00:00    
(11/14): php8.2-process-8.2.15-1.amzn2023.0.2.x 2.7 MB/s |  44 kB     00:00    
(12/14): php8.2-sodium-8.2.15-1.amzn2023.0.2.x8 2.3 MB/s |  42 kB     00:00    
(13/14): php8.2-xml-8.2.15-1.amzn2023.0.2.x86_6 5.3 MB/s | 148 kB     00:00    
(14/14): php8.2-pdo-8.2.15-1.amzn2023.0.2.x86_6 694 kB/s |  89 kB     00:00    
--------------------------------------------------------------------------------
Total                                            18 MB/s | 8.1 MB     00:00     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : php8.2-common-8.2.15-1.amzn2023.0.2.x86_64            1/14 
  Installing       : php8.2-cli-8.2.15-1.amzn2023.0.2.x86_64               2/14 
  Installing       : php8.2-process-8.2.15-1.amzn2023.0.2.x86_64           3/14 
  Installing       : php8.2-mbstring-8.2.15-1.amzn2023.0.2.x86_64          4/14 
  Installing       : php8.2-opcache-8.2.15-1.amzn2023.0.2.x86_64           5/14 
  Installing       : php8.2-pdo-8.2.15-1.amzn2023.0.2.x86_64               6/14 
  Running scriptlet: nginx-filesystem-1:1.24.0-1.amzn2023.0.2.noarch       7/14 
  Installing       : nginx-filesystem-1:1.24.0-1.amzn2023.0.2.noarch       7/14 
  Installing       : php8.2-fpm-8.2.15-1.amzn2023.0.2.x86_64               8/14 
  Running scriptlet: php8.2-fpm-8.2.15-1.amzn2023.0.2.x86_64               8/14 
  Installing       : libxslt-1.1.34-5.amzn2023.0.2.x86_64                  9/14 
  Installing       : php8.2-xml-8.2.15-1.amzn2023.0.2.x86_64              10/14 
  Installing       : libsodium-1.0.19-4.amzn2023.x86_64                   11/14 
  Installing       : php8.2-sodium-8.2.15-1.amzn2023.0.2.x86_64           12/14 
  Installing       : php8.2-8.2.15-1.amzn2023.0.2.x86_64                  13/14 
  Installing       : php-pear-1:1.10.13-2.amzn2023.0.4.noarch             14/14 
  Running scriptlet: php-pear-1:1.10.13-2.amzn2023.0.4.noarch             14/14 
  Verifying        : libsodium-1.0.19-4.amzn2023.x86_64                    1/14 
  Verifying        : libxslt-1.1.34-5.amzn2023.0.2.x86_64                  2/14 
  Verifying        : nginx-filesystem-1:1.24.0-1.amzn2023.0.2.noarch       3/14 
  Verifying        : php-pear-1:1.10.13-2.amzn2023.0.4.noarch              4/14 
  Verifying        : php8.2-8.2.15-1.amzn2023.0.2.x86_64                   5/14 
  Verifying        : php8.2-cli-8.2.15-1.amzn2023.0.2.x86_64               6/14 
  Verifying        : php8.2-common-8.2.15-1.amzn2023.0.2.x86_64            7/14 
  Verifying        : php8.2-fpm-8.2.15-1.amzn2023.0.2.x86_64               8/14 
  Verifying        : php8.2-mbstring-8.2.15-1.amzn2023.0.2.x86_64          9/14 
  Verifying        : php8.2-opcache-8.2.15-1.amzn2023.0.2.x86_64          10/14 
  Verifying        : php8.2-pdo-8.2.15-1.amzn2023.0.2.x86_64              11/14 
  Verifying        : php8.2-process-8.2.15-1.amzn2023.0.2.x86_64          12/14 
  Verifying        : php8.2-sodium-8.2.15-1.amzn2023.0.2.x86_64           13/14 
  Verifying        : php8.2-xml-8.2.15-1.amzn2023.0.2.x86_64              14/14 

Installed:
  libsodium-1.0.19-4.amzn2023.x86_64                                            
  libxslt-1.1.34-5.amzn2023.0.2.x86_64                                          
  nginx-filesystem-1:1.24.0-1.amzn2023.0.2.noarch                               
  php-pear-1:1.10.13-2.amzn2023.0.4.noarch                                      
  php8.2-8.2.15-1.amzn2023.0.2.x86_64                                           
  php8.2-cli-8.2.15-1.amzn2023.0.2.x86_64                                       
  php8.2-common-8.2.15-1.amzn2023.0.2.x86_64                                    
  php8.2-fpm-8.2.15-1.amzn2023.0.2.x86_64                                       
  php8.2-mbstring-8.2.15-1.amzn2023.0.2.x86_64                                  
  php8.2-opcache-8.2.15-1.amzn2023.0.2.x86_64                                   
  php8.2-pdo-8.2.15-1.amzn2023.0.2.x86_64                                       
  php8.2-process-8.2.15-1.amzn2023.0.2.x86_64                                   
  php8.2-sodium-8.2.15-1.amzn2023.0.2.x86_64                                    
  php8.2-xml-8.2.15-1.amzn2023.0.2.x86_64                                       

Complete!
Last metadata expiration check: 0:00:06 ago on Wed May  8 16:25:37 2024.
Package php8.2-cli-8.2.15-1.amzn2023.0.2.x86_64 is already installed.
Package php8.2-common-8.2.15-1.amzn2023.0.2.x86_64 is already installed.
Package php8.2-mbstring-8.2.15-1.amzn2023.0.2.x86_64 is already installed.
Package php8.2-common-8.2.15-1.amzn2023.0.2.x86_64 is already installed.
Package php8.2-common-8.2.15-1.amzn2023.0.2.x86_64 is already installed.
Package php8.2-xml-8.2.15-1.amzn2023.0.2.x86_64 is already installed.
Package php8.2-fpm-8.2.15-1.amzn2023.0.2.x86_64 is already installed.
Dependencies resolved.
================================================================================
 Package                   Arch   Version                     Repository   Size
================================================================================
Installing:
 php8.2-gd                 x86_64 8.2.15-1.amzn2023.0.2       amazonlinux  44 k
 php8.2-intl               x86_64 8.2.15-1.amzn2023.0.2       amazonlinux 170 k
 php8.2-mysqlnd            x86_64 8.2.15-1.amzn2023.0.2       amazonlinux 148 k
 php8.2-zip                x86_64 8.2.15-1.amzn2023.0.2       amazonlinux  40 k
Installing dependencies:
 cairo                     x86_64 1.17.6-2.amzn2023.0.1       amazonlinux 684 k
 fontconfig                x86_64 2.13.94-2.amzn2023.0.2      amazonlinux 273 k
 fonts-filesystem          noarch 1:2.0.5-12.amzn2023.0.2     amazonlinux 9.5 k
 freetype                  x86_64 2.13.0-2.amzn2023.0.1       amazonlinux 422 k
 gd                        x86_64 2.3.3-5.amzn2023.0.3        amazonlinux 139 k
 google-noto-fonts-common  noarch 20201206-2.amzn2023.0.2     amazonlinux  15 k
 google-noto-sans-vf-fonts noarch 20201206-2.amzn2023.0.2     amazonlinux 492 k
 graphite2                 x86_64 1.3.14-7.amzn2023.0.2       amazonlinux  97 k
 harfbuzz                  x86_64 7.0.0-2.amzn2023.0.1        amazonlinux 868 k
 jbigkit-libs              x86_64 2.1-21.amzn2023.0.2         amazonlinux  54 k
 langpacks-core-font-en    noarch 3.0-21.amzn2023.0.4         amazonlinux  10 k
 libX11                    x86_64 1.7.2-3.amzn2023.0.4        amazonlinux 657 k
 libX11-common             noarch 1.7.2-3.amzn2023.0.4        amazonlinux 152 k
 libXau                    x86_64 1.0.9-6.amzn2023.0.2        amazonlinux  31 k
 libXext                   x86_64 1.3.4-6.amzn2023.0.2        amazonlinux  41 k
 libXpm                    x86_64 3.5.15-2.amzn2023.0.3       amazonlinux  65 k
 libXrender                x86_64 0.9.10-14.amzn2023.0.2      amazonlinux  28 k
 libicu                    x86_64 67.1-7.amzn2023.0.3         amazonlinux 9.6 M
 libjpeg-turbo             x86_64 2.1.4-2.amzn2023.0.5        amazonlinux 190 k
 libpng                    x86_64 2:1.6.37-10.amzn2023.0.6    amazonlinux 128 k
 libtiff                   x86_64 4.4.0-4.amzn2023.0.18       amazonlinux 213 k
 libwebp                   x86_64 1.2.4-1.amzn2023.0.6        amazonlinux 341 k
 libxcb                    x86_64 1.13.1-7.amzn2023.0.2       amazonlinux 230 k
 libzip                    x86_64 1.7.3-4.amzn2023.0.3        amazonlinux  63 k
 pixman                    x86_64 0.40.0-3.amzn2023.0.3       amazonlinux 295 k
 xml-common                noarch 0.6.3-56.amzn2023.0.2       amazonlinux  32 k

Transaction Summary
================================================================================
Install  30 Packages

Total download size: 15 M
Installed size: 49 M
Downloading Packages:
(1/30): fonts-filesystem-2.0.5-12.amzn2023.0.2. 135 kB/s | 9.5 kB     00:00    
(2/30): fontconfig-2.13.94-2.amzn2023.0.2.x86_6 3.2 MB/s | 273 kB     00:00    
(3/30): cairo-1.17.6-2.amzn2023.0.1.x86_64.rpm  6.7 MB/s | 684 kB     00:00    
(4/30): gd-2.3.3-5.amzn2023.0.3.x86_64.rpm      5.3 MB/s | 139 kB     00:00    
(5/30): freetype-2.13.0-2.amzn2023.0.1.x86_64.r 9.0 MB/s | 422 kB     00:00    
(6/30): google-noto-fonts-common-20201206-2.amz 750 kB/s |  15 kB     00:00    
(7/30): graphite2-1.3.14-7.amzn2023.0.2.x86_64. 5.4 MB/s |  97 kB     00:00    
(8/30): google-noto-sans-vf-fonts-20201206-2.am  13 MB/s | 492 kB     00:00    
(9/30): jbigkit-libs-2.1-21.amzn2023.0.2.x86_64 2.4 MB/s |  54 kB     00:00    
(10/30): harfbuzz-7.0.0-2.amzn2023.0.1.x86_64.r  17 MB/s | 868 kB     00:00    
(11/30): langpacks-core-font-en-3.0-21.amzn2023 435 kB/s |  10 kB     00:00    
(12/30): libX11-1.7.2-3.amzn2023.0.4.x86_64.rpm  19 MB/s | 657 kB     00:00    
(13/30): libX11-common-1.7.2-3.amzn2023.0.4.noa 4.6 MB/s | 152 kB     00:00    
(14/30): libXau-1.0.9-6.amzn2023.0.2.x86_64.rpm 899 kB/s |  31 kB     00:00    
(15/30): libXext-1.3.4-6.amzn2023.0.2.x86_64.rp 2.1 MB/s |  41 kB     00:00    
(16/30): libXrender-0.9.10-14.amzn2023.0.2.x86_ 1.7 MB/s |  28 kB     00:00    
(17/30): libXpm-3.5.15-2.amzn2023.0.3.x86_64.rp 2.4 MB/s |  65 kB     00:00    
(18/30): libjpeg-turbo-2.1.4-2.amzn2023.0.5.x86 7.0 MB/s | 190 kB     00:00    
(19/30): libpng-1.6.37-10.amzn2023.0.6.x86_64.r 4.3 MB/s | 128 kB     00:00    
(20/30): libtiff-4.4.0-4.amzn2023.0.18.x86_64.r 5.3 MB/s | 213 kB     00:00    
(21/30): libwebp-1.2.4-1.amzn2023.0.6.x86_64.rp 7.6 MB/s | 341 kB     00:00    
(22/30): libxcb-1.13.1-7.amzn2023.0.2.x86_64.rp  11 MB/s | 230 kB     00:00    
(23/30): libicu-67.1-7.amzn2023.0.3.x86_64.rpm   53 MB/s | 9.6 MB     00:00    
(24/30): libzip-1.7.3-4.amzn2023.0.3.x86_64.rpm 710 kB/s |  63 kB     00:00    
(25/30): php8.2-gd-8.2.15-1.amzn2023.0.2.x86_64 551 kB/s |  44 kB     00:00    
(26/30): php8.2-mysqlnd-8.2.15-1.amzn2023.0.2.x 6.6 MB/s | 148 kB     00:00    
(27/30): php8.2-zip-8.2.15-1.amzn2023.0.2.x86_6 1.1 MB/s |  40 kB     00:00    
(28/30): pixman-0.40.0-3.amzn2023.0.3.x86_64.rp  10 MB/s | 295 kB     00:00    
(29/30): xml-common-0.6.3-56.amzn2023.0.2.noarc 1.5 MB/s |  32 kB     00:00    
(30/30): php8.2-intl-8.2.15-1.amzn2023.0.2.x86_ 1.6 MB/s | 170 kB     00:00    
--------------------------------------------------------------------------------
Total                                            28 MB/s |  15 MB     00:00     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : libpng-2:1.6.37-10.amzn2023.0.6.x86_64                1/30 
  Installing       : libwebp-1.2.4-1.amzn2023.0.6.x86_64                   2/30 
  Installing       : libjpeg-turbo-2.1.4-2.amzn2023.0.5.x86_64             3/30 
  Installing       : fonts-filesystem-1:2.0.5-12.amzn2023.0.2.noarch       4/30 
  Running scriptlet: xml-common-0.6.3-56.amzn2023.0.2.noarch               5/30 
  Installing       : xml-common-0.6.3-56.amzn2023.0.2.noarch               5/30 
  Installing       : pixman-0.40.0-3.amzn2023.0.3.x86_64                   6/30 
  Installing       : libzip-1.7.3-4.amzn2023.0.3.x86_64                    7/30 
  Installing       : libicu-67.1-7.amzn2023.0.3.x86_64                     8/30 
  Installing       : libXau-1.0.9-6.amzn2023.0.2.x86_64                    9/30 
  Installing       : libxcb-1.13.1-7.amzn2023.0.2.x86_64                  10/30 
  Installing       : libX11-common-1.7.2-3.amzn2023.0.4.noarch            11/30 
  Installing       : libX11-1.7.2-3.amzn2023.0.4.x86_64                   12/30 
  Installing       : libXext-1.3.4-6.amzn2023.0.2.x86_64                  13/30 
  Installing       : libXpm-3.5.15-2.amzn2023.0.3.x86_64                  14/30 
  Installing       : libXrender-0.9.10-14.amzn2023.0.2.x86_64             15/30 
  Installing       : jbigkit-libs-2.1-21.amzn2023.0.2.x86_64              16/30 
  Installing       : libtiff-4.4.0-4.amzn2023.0.18.x86_64                 17/30 
  Installing       : graphite2-1.3.14-7.amzn2023.0.2.x86_64               18/30 
  Installing       : google-noto-fonts-common-20201206-2.amzn2023.0.2.n   19/30 
  Installing       : google-noto-sans-vf-fonts-20201206-2.amzn2023.0.2.   20/30 
  Installing       : langpacks-core-font-en-3.0-21.amzn2023.0.4.noarch    21/30 
  Installing       : cairo-1.17.6-2.amzn2023.0.1.x86_64                   22/30 
  Installing       : harfbuzz-7.0.0-2.amzn2023.0.1.x86_64                 23/30 
  Installing       : freetype-2.13.0-2.amzn2023.0.1.x86_64                24/30 
  Installing       : fontconfig-2.13.94-2.amzn2023.0.2.x86_64             25/30 
  Running scriptlet: fontconfig-2.13.94-2.amzn2023.0.2.x86_64             25/30 
  Installing       : gd-2.3.3-5.amzn2023.0.3.x86_64                       26/30 
  Installing       : php8.2-gd-8.2.15-1.amzn2023.0.2.x86_64               27/30 
  Installing       : php8.2-intl-8.2.15-1.amzn2023.0.2.x86_64             28/30 
  Installing       : php8.2-zip-8.2.15-1.amzn2023.0.2.x86_64              29/30 
  Installing       : php8.2-mysqlnd-8.2.15-1.amzn2023.0.2.x86_64          30/30 
  Running scriptlet: fontconfig-2.13.94-2.amzn2023.0.2.x86_64             30/30 
  Running scriptlet: php8.2-mysqlnd-8.2.15-1.amzn2023.0.2.x86_64          30/30 
  Verifying        : cairo-1.17.6-2.amzn2023.0.1.x86_64                    1/30 
  Verifying        : fontconfig-2.13.94-2.amzn2023.0.2.x86_64              2/30 
  Verifying        : fonts-filesystem-1:2.0.5-12.amzn2023.0.2.noarch       3/30 
  Verifying        : freetype-2.13.0-2.amzn2023.0.1.x86_64                 4/30 
  Verifying        : gd-2.3.3-5.amzn2023.0.3.x86_64                        5/30 
  Verifying        : google-noto-fonts-common-20201206-2.amzn2023.0.2.n    6/30 
  Verifying        : google-noto-sans-vf-fonts-20201206-2.amzn2023.0.2.    7/30 
  Verifying        : graphite2-1.3.14-7.amzn2023.0.2.x86_64                8/30 
  Verifying        : harfbuzz-7.0.0-2.amzn2023.0.1.x86_64                  9/30 
  Verifying        : jbigkit-libs-2.1-21.amzn2023.0.2.x86_64              10/30 
  Verifying        : langpacks-core-font-en-3.0-21.amzn2023.0.4.noarch    11/30 
  Verifying        : libX11-1.7.2-3.amzn2023.0.4.x86_64                   12/30 
  Verifying        : libX11-common-1.7.2-3.amzn2023.0.4.noarch            13/30 
  Verifying        : libXau-1.0.9-6.amzn2023.0.2.x86_64                   14/30 
  Verifying        : libXext-1.3.4-6.amzn2023.0.2.x86_64                  15/30 
  Verifying        : libXpm-3.5.15-2.amzn2023.0.3.x86_64                  16/30 
  Verifying        : libXrender-0.9.10-14.amzn2023.0.2.x86_64             17/30 
  Verifying        : libicu-67.1-7.amzn2023.0.3.x86_64                    18/30 
  Verifying        : libjpeg-turbo-2.1.4-2.amzn2023.0.5.x86_64            19/30 
  Verifying        : libpng-2:1.6.37-10.amzn2023.0.6.x86_64               20/30 
  Verifying        : libtiff-4.4.0-4.amzn2023.0.18.x86_64                 21/30 
  Verifying        : libwebp-1.2.4-1.amzn2023.0.6.x86_64                  22/30 
  Verifying        : libxcb-1.13.1-7.amzn2023.0.2.x86_64                  23/30 
  Verifying        : libzip-1.7.3-4.amzn2023.0.3.x86_64                   24/30 
  Verifying        : php8.2-gd-8.2.15-1.amzn2023.0.2.x86_64               25/30 
  Verifying        : php8.2-intl-8.2.15-1.amzn2023.0.2.x86_64             26/30 
  Verifying        : php8.2-mysqlnd-8.2.15-1.amzn2023.0.2.x86_64          27/30 
  Verifying        : php8.2-zip-8.2.15-1.amzn2023.0.2.x86_64              28/30 
  Verifying        : pixman-0.40.0-3.amzn2023.0.3.x86_64                  29/30 
  Verifying        : xml-common-0.6.3-56.amzn2023.0.2.noarch              30/30 

Installed:
  cairo-1.17.6-2.amzn2023.0.1.x86_64                                            
  fontconfig-2.13.94-2.amzn2023.0.2.x86_64                                      
  fonts-filesystem-1:2.0.5-12.amzn2023.0.2.noarch                               
  freetype-2.13.0-2.amzn2023.0.1.x86_64                                         
  gd-2.3.3-5.amzn2023.0.3.x86_64                                                
  google-noto-fonts-common-20201206-2.amzn2023.0.2.noarch                       
  google-noto-sans-vf-fonts-20201206-2.amzn2023.0.2.noarch                      
  graphite2-1.3.14-7.amzn2023.0.2.x86_64                                        
  harfbuzz-7.0.0-2.amzn2023.0.1.x86_64                                          
  jbigkit-libs-2.1-21.amzn2023.0.2.x86_64                                       
  langpacks-core-font-en-3.0-21.amzn2023.0.4.noarch                             
  libX11-1.7.2-3.amzn2023.0.4.x86_64                                            
  libX11-common-1.7.2-3.amzn2023.0.4.noarch                                     
  libXau-1.0.9-6.amzn2023.0.2.x86_64                                            
  libXext-1.3.4-6.amzn2023.0.2.x86_64                                           
  libXpm-3.5.15-2.amzn2023.0.3.x86_64                                           
  libXrender-0.9.10-14.amzn2023.0.2.x86_64                                      
  libicu-67.1-7.amzn2023.0.3.x86_64                                             
  libjpeg-turbo-2.1.4-2.amzn2023.0.5.x86_64                                     
  libpng-2:1.6.37-10.amzn2023.0.6.x86_64                                        
  libtiff-4.4.0-4.amzn2023.0.18.x86_64                                          
  libwebp-1.2.4-1.amzn2023.0.6.x86_64                                           
  libxcb-1.13.1-7.amzn2023.0.2.x86_64                                           
  libzip-1.7.3-4.amzn2023.0.3.x86_64                                            
  php8.2-gd-8.2.15-1.amzn2023.0.2.x86_64                                        
  php8.2-intl-8.2.15-1.amzn2023.0.2.x86_64                                      
  php8.2-mysqlnd-8.2.15-1.amzn2023.0.2.x86_64                                   
  php8.2-zip-8.2.15-1.amzn2023.0.2.x86_64                                       
  pixman-0.40.0-3.amzn2023.0.3.x86_64                                           
  xml-common-0.6.3-56.amzn2023.0.2.noarch                                       

Complete!
Retrieving https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
warning: /var/tmp/rpm-tmp.BXEF7m: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
Verifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql57-community-release-el7-11 ################################# [100%]
MySQL Connectors Community                      1.5 MB/s |  72 kB     00:00    
MySQL Tools Community                            11 MB/s | 1.2 MB     00:00    
MySQL 5.7 Community Server                       25 MB/s | 3.1 MB     00:00    
Dependencies resolved.
================================================================================
 Package                Arch   Version                  Repository         Size
================================================================================
Installing:
 mysql-community-server x86_64 5.7.44-1.el7             mysql57-community 184 M
Installing dependencies:
 libxcrypt-compat       x86_64 4.4.33-7.amzn2023        amazonlinux        92 k
 mysql-community-client x86_64 5.7.44-1.el7             mysql57-community  31 M
 mysql-community-common x86_64 5.7.44-1.el7             mysql57-community 313 k
 mysql-community-libs   x86_64 5.7.44-1.el7             mysql57-community 3.0 M
 ncurses-compat-libs    x86_64 6.2-4.20200222.amzn2023.0.6
                                                        amazonlinux       323 k

Transaction Summary
================================================================================
Install  6 Packages

Total download size: 219 M
Installed size: 931 M
Downloading Packages:
(1/6): libxcrypt-compat-4.4.33-7.amzn2023.x86_6 1.2 MB/s |  92 kB     00:00    
(2/6): ncurses-compat-libs-6.2-4.20200222.amzn2 2.6 MB/s | 323 kB     00:00    
(3/6): mysql-community-common-5.7.44-1.el7.x86_ 3.4 MB/s | 313 kB     00:00    
(4/6): mysql-community-libs-5.7.44-1.el7.x86_64  15 MB/s | 3.0 MB     00:00    
(5/6): mysql-community-client-5.7.44-1.el7.x86_  47 MB/s |  31 MB     00:00    
(6/6): mysql-community-server-5.7.44-1.el7.x86_  60 MB/s | 184 MB     00:03    
--------------------------------------------------------------------------------
Total                                            66 MB/s | 219 MB     00:03     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : mysql-community-common-5.7.44-1.el7.x86_64             1/6 
  Installing       : mysql-community-libs-5.7.44-1.el7.x86_64               2/6 
  Running scriptlet: mysql-community-libs-5.7.44-1.el7.x86_64               2/6 
  Installing       : ncurses-compat-libs-6.2-4.20200222.amzn2023.0.6.x86_   3/6 
  Installing       : mysql-community-client-5.7.44-1.el7.x86_64             4/6 
  Installing       : libxcrypt-compat-4.4.33-7.amzn2023.x86_64              5/6 
  Running scriptlet: mysql-community-server-5.7.44-1.el7.x86_64             6/6 
  Installing       : mysql-community-server-5.7.44-1.el7.x86_64             6/6 
  Running scriptlet: mysql-community-server-5.7.44-1.el7.x86_64             6/6 
/usr/lib/tmpfiles.d/mysql.conf:23: Line references path below legacy directory /var/run/, updating /var/run/mysqld → /run/mysqld; please update the tmpfiles.d/ drop-in file accordingly.

  Verifying        : libxcrypt-compat-4.4.33-7.amzn2023.x86_64              1/6 
  Verifying        : ncurses-compat-libs-6.2-4.20200222.amzn2023.0.6.x86_   2/6 
  Verifying        : mysql-community-client-5.7.44-1.el7.x86_64             3/6 
  Verifying        : mysql-community-common-5.7.44-1.el7.x86_64             4/6 
  Verifying        : mysql-community-libs-5.7.44-1.el7.x86_64               5/6 
  Verifying        : mysql-community-server-5.7.44-1.el7.x86_64             6/6 

Installed:
  libxcrypt-compat-4.4.33-7.amzn2023.x86_64                                     
  mysql-community-client-5.7.44-1.el7.x86_64                                    
  mysql-community-common-5.7.44-1.el7.x86_64                                    
  mysql-community-libs-5.7.44-1.el7.x86_64                                      
  mysql-community-server-5.7.44-1.el7.x86_64                                    
  ncurses-compat-libs-6.2-4.20200222.amzn2023.0.6.x86_64                        

Complete!
mount.nfs4: Failed to resolve server fs-07d0a7f8f87044530.efs.eu-west-2.amazonaws.com: Name or service not known
Redirecting to /bin/systemctl restart httpd.service
[root@ip-14-0-1-75 ec2-user]# 

•	To check Apache status;


[root@ip-14-0-1-75 ec2-user]# sudo systemctl status httpd
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; preset: di>
    Drop-In: /usr/lib/systemd/system/httpd.service.d
             └─php-fpm.conf
     Active: active (running) since Wed 2024-05-08 16:26:42 UTC; 9min ago
       Docs: man:httpd.service(8)
   Main PID: 29524 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes>
      Tasks: 177 (limit: 1114)
     Memory: 14.2M
        CPU: 391ms
     CGroup: /system.slice/httpd.service
             ├─29524 /usr/sbin/httpd -DFOREGROUND
             ├─29526 /usr/sbin/httpd -DFOREGROUND
             ├─29527 /usr/sbin/httpd -DFOREGROUND
             ├─29528 /usr/sbin/httpd -DFOREGROUND
             └─29529 /usr/sbin/httpd -DFOREGROUND

May 08 16:26:42 ip-14-0-1-75.ec2.internal systemd[1]: Stopped httpd.service - T>
May 08 16:26:42 ip-14-0-1-75.ec2.internal systemd[1]: Starting httpd.service - >
May 08 16:26:42 ip-14-0-1-75.ec2.internal systemd[1]: Started httpd.service - T>
May 08 16:26:42 ip-14-0-1-75.ec2.internal httpd[29524]: Server configured, list>
lines 1-22/22 (END)

2. To check MySQL;

[ec2-user@ip-14-0-1-75 ~]$ sudo systemctl start httpd
[ec2-user@ip-14-0-1-75 ~]$ sudo systemctl status mysqld
● mysqld.service - MySQL Server
     Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; preset: d>
     Active: active (running) since Wed 2024-05-08 16:26:41 UTC; 29min ago
       Docs: man:mysqld(8)
             http://dev.mysql.com/doc/refman/en/using-systemd.html
    Process: 29420 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, statu>
    Process: 29472 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/m>
   Main PID: 29474 (mysqld)
      Tasks: 27 (limit: 1114)
     Memory: 317.6M
        CPU: 4.603s
     CGroup: /system.slice/mysqld.service
             └─29474 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/my>

May 08 16:26:32 ip-14-0-1-75.ec2.internal systemd[1]: Starting mysqld.service ->
May 08 16:26:34 ip-14-0-1-75.ec2.internal mysqld_pre_systemd[29444]: mysqld: Ou>
May 08 16:26:41 ip-14-0-1-75.ec2.internal systemd[1]: Started mysqld.service - >
lines 1-17/17 (END)


Mounting EFS files on EC2


sojy@OlasojiUbuntu:~/Downloads$ chmod 400 "wordpress-key.pem"
sojy@OlasojiUbuntu:~/Downloads$ ssh -i "wordpress-key.pem" ec2-user@ec2-54-209-39-201.compute-1.amazonaws.com
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~&apos; &apos;->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/&apos;
Last login: Wed May  8 16:43:56 2024 from 102.91.4.28
[ec2-user@ip-14-0-1-75 ~]$ 







[ec2-user@ip-14-0-1-75 ~]$ sudo su
[root@ip-14-0-1-75 ec2-user]# df -khp
df: invalid option -- &apos;p&apos;
Try &apos;df --help&apos; for more information.
[root@ip-14-0-1-75 ec2-user]# df -khP
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        4.0M     0  4.0M   0% /dev
tmpfs           475M     0  475M   0% /dev/shm
tmpfs           190M  2.9M  188M   2% /run
/dev/xvda1      8.0G  2.7G  5.3G  34% /
tmpfs           475M     0  475M   0% /tmp
/dev/xvda128     10M  1.3M  8.7M  13% /boot/efi
tmpfs            95M     0   95M   0% /run/user/1000
[root@ip-14-0-1-75 ec2-user]# mkdir newdir1
[root@ip-14-0-1-75 ec2-user]# mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-01b792b1a3c695317.efs.us-east-1.amazonaws.com:/ newdir1
[root@ip-14-0-1-75 ec2-user]# df -khP
Filesystem                                          Size  Used Avail Use% Mounted on
devtmpfs                                            4.0M     0  4.0M   0% /dev
tmpfs                                               475M     0  475M   0% /dev/shm
tmpfs                                               190M  2.9M  188M   2% /run
/dev/xvda1                                          8.0G  2.7G  5.3G  34% /
tmpfs                                               475M     0  475M   0% /tmp
/dev/xvda128                                         10M  1.3M  8.7M  13% /boot/efi
tmpfs                                                95M     0   95M   0% /run/user/1000
fs-01b792b1a3c695317.efs.us-east-1.amazonaws.com:/  8.0E     0  8.0E   0% /home/ec2-user/newdir1
[root@ip-14-0-1-75 ec2-user]# 

Download WordPress and copy WordPress to var/www/html


[ec2-user@ip-14-0-1-75 ~]$ cd /home/ec2-user/newdir1
[ec2-user@ip-14-0-1-75 newdir1]$ pwd
/home/ec2-user/newdir1
[ec2-user@ip-14-0-1-75 newdir1]$ cd ..
[ec2-user@ip-14-0-1-75 ~]$      mkdir wordpress

        cd   wordpress

        sudo wget http://wordpress.org/latest.tar.gz
 
        sudo tar -xzvf latest.tar.gz
 
        sudo rm -rf latest.tar.gz
 
        sudo cp -R wordpress /var/www/html/

        sudo systemctl restart httpd
--2024-05-09 01:17:15--  http://wordpress.org/latest.tar.gz
Resolving wordpress.org (wordpress.org)... 198.143.164.252
Connecting to wordpress.org (wordpress.org)|198.143.164.252|:80... connected.
HTTP request sent, awaiting response... 301 Moved Permanently
Location: https://wordpress.org/latest.tar.gz [following]
--2024-05-09 01:17:15--  https://wordpress.org/latest.tar.gz
Connecting to wordpress.org (wordpress.org)|198.143.164.252|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 24696379 (24M) [application/octet-stream]
Saving to: ‘latest.tar.gz’

latest.tar.gz       100%[===================>]  23.55M  52.3MB/s    in 0.5s    

2024-05-09 01:17:15 (52.3 MB/s) - ‘latest.tar.gz’ saved [24696379/24696379]

wordpress/
wordpress/xmlrpc.php
wordpress/wp-blog-header.php
wordpress/readme.html
wordpress/wp-signup.php
wordpress/index.php
wordpress/wp-cron.php
wordpress/wp-config-sample.php
wordpress/wp-login.php
wordpress/wp-settings.php
wordpress/license.txt
wordpress/wp-content/
wordpress/wp-content/themes/
wordpress/wp-content/themes/twentytwentythree/
wordpress/wp-content/themes/twentytwentythree/theme.json
wordpress/wp-content/themes/twentytwentythree/parts/
wordpress/wp-content/themes/twentytwentythree/parts/footer.html
wordpress/wp-content/themes/twentytwentythree/parts/comments.html
wordpress/wp-content/themes/twentytwentythree/parts/header.html
wordpress/wp-content/themes/twentytwentythree/parts/post-meta.html
wordpress/wp-content/themes/twentytwentythree/patterns/
wordpress/wp-content/themes/twentytwentythree/patterns/hidden-404.php
wordpress/wp-content/themes/twentytwentythree/patterns/post-meta.php
wordpress/wp-content/themes/twentytwentythree/patterns/hidden-no-results.php
wordpress/wp-content/themes/twentytwentythree/patterns/call-to-action.php
wordpress/wp-content/themes/twentytwentythree/patterns/footer-default.php
wordpress/wp-content/themes/twentytwentythree/patterns/hidden-comments.php
wordpress/wp-content/themes/twentytwentythree/styles/
wordpress/wp-content/themes/twentytwentythree/styles/sherbet.json
wordpress/wp-content/themes/twentytwentythree/styles/grapes.json
wordpress/wp-content/themes/twentytwentythree/styles/canary.json
wordpress/wp-content/themes/twentytwentythree/styles/electric.json
wordpress/wp-content/themes/twentytwentythree/styles/pitch.json
wordpress/wp-content/themes/twentytwentythree/styles/block-out.json
wordpress/wp-content/themes/twentytwentythree/styles/whisper.json
wordpress/wp-content/themes/twentytwentythree/styles/marigold.json
wordpress/wp-content/themes/twentytwentythree/styles/aubergine.json
wordpress/wp-content/themes/twentytwentythree/styles/pilgrimage.json
wordpress/wp-content/themes/twentytwentythree/assets/
wordpress/wp-content/themes/twentytwentythree/assets/fonts/
wordpress/wp-content/themes/twentytwentythree/assets/fonts/source-serif-pro/
wordpress/wp-content/themes/twentytwentythree/assets/fonts/source-serif-pro/SourceSerif4Variable-Roman.ttf.woff2
wordpress/wp-content/themes/twentytwentythree/assets/fonts/source-serif-pro/SourceSerif4Variable-Italic.otf.woff2
wordpress/wp-content/themes/twentytwentythree/assets/fonts/source-serif-pro/SourceSerif4Variable-Italic.ttf.woff2
wordpress/wp-content/themes/twentytwentythree/assets/fonts/source-serif-pro/SourceSerif4Variable-Roman.otf.woff2
wordpress/wp-content/themes/twentytwentythree/assets/fonts/source-serif-pro/LICENSE.md
wordpress/wp-content/themes/twentytwentythree/assets/fonts/ibm-plex-mono/
wordpress/wp-content/themes/twentytwentythree/assets/fonts/ibm-plex-mono/IBMPlexMono-Italic.woff2
wordpress/wp-content/themes/twentytwentythree/assets/fonts/ibm-plex-mono/IBMPlexMono-Regular.woff2
wordpress/wp-content/themes/twentytwentythree/assets/fonts/ibm-plex-mono/OFL.txt
wordpress/wp-content/themes/twentytwentythree/assets/fonts/ibm-plex-mono/IBMPlexMono-Light.woff2
wordpress/wp-content/themes/twentytwentythree/assets/fonts/ibm-plex-mono/IBMPlexMono-Bold.woff2
wordpress/wp-content/themes/twentytwentythree/assets/fonts/inter/
wordpress/wp-content/themes/twentytwentythree/assets/fonts/inter/LICENSE.txt
wordpress/wp-content/themes/twentytwentythree/assets/fonts/inter/Inter-VariableFont_slnt,wght.ttf
wordpress/wp-content/themes/twentytwentythree/assets/fonts/dm-sans/
wordpress/wp-content/themes/twentytwentythree/assets/fonts/dm-sans/DMSans-Regular-Italic.woff2
wordpress/wp-content/themes/twentytwentythree/assets/fonts/dm-sans/DMSans-Bold.woff2
wordpress/wp-content/themes/twentytwentythree/assets/fonts/dm-sans/LICENSE.txt
wordpress/wp-content/themes/twentytwentythree/assets/fonts/dm-sans/DMSans-Bold-Italic.woff2
wordpress/wp-content/themes/twentytwentythree/assets/fonts/dm-sans/DMSans-Regular.woff2
wordpress/wp-content/themes/twentytwentythree/screenshot.png
wordpress/wp-content/themes/twentytwentythree/readme.txt
wordpress/wp-content/themes/twentytwentythree/templates/
wordpress/wp-content/themes/twentytwentythree/templates/blank.html
wordpress/wp-content/themes/twentytwentythree/templates/page.html
wordpress/wp-content/themes/twentytwentythree/templates/index.html
wordpress/wp-content/themes/twentytwentythree/templates/blog-alternative.html
wordpress/wp-content/themes/twentytwentythree/templates/archive.html
wordpress/wp-content/themes/twentytwentythree/templates/search.html
wordpress/wp-content/themes/twentytwentythree/templates/single.html
wordpress/wp-content/themes/twentytwentythree/templates/404.html
wordpress/wp-content/themes/twentytwentythree/templates/home.html
wordpress/wp-content/themes/twentytwentythree/style.css
wordpress/wp-content/themes/twentytwentytwo/
wordpress/wp-content/themes/twentytwentytwo/theme.json
wordpress/wp-content/themes/twentytwentytwo/inc/
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/hidden-404.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/header-text-only-with-tagline-black-background.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/page-layout-two-columns.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/general-wide-image-intro-buttons.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/header-logo-navigation-social-black-background.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/query-simple-blog.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/footer-social-copyright.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/header-logo-navigation-offset-tagline.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/footer-navigation.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/query-large-titles.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/page-sidebar-blog-posts.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/query-image-grid.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/query-grid.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/page-about-media-left.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/header-small-dark.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/header-with-tagline.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/page-sidebar-poster.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/general-featured-posts.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/footer-query-title-citation.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/general-list-events.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/general-subscribe.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/general-image-with-caption.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/page-sidebar-grid-posts.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/page-about-links.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/header-centered-logo.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/general-video-header-details.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/header-text-only-salmon-background.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/header-title-navigation-social.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/footer-dark.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/header-text-only-green-background.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/footer-default.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/general-video-trailer.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/footer-query-images-title-citation.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/page-about-links-dark.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/hidden-bird.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/page-layout-image-and-text.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/header-large-dark.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/page-about-large-image-and-buttons.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/general-pricing-table.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/page-about-media-right.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/general-layered-images-with-duotone.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/header-logo-navigation-gray-background.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/footer-title-tagline-social.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/header-centered-title-navigation-social.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/general-divider-dark.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/page-about-simple-dark.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/query-default.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/general-two-images-text.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/query-text-grid.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/header-stacked.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/footer-about-title-logo.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/header-default.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/page-about-solid-color.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/query-irregular-grid.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/footer-blog.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/general-large-list-names.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/header-title-and-button.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/hidden-heading-and-bird.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/header-centered-logo-black-background.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/footer-navigation-copyright.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/header-image-background-overlay.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/page-sidebar-blog-posts-right.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/general-divider-light.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/footer-logo.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/page-layout-image-text-and-video.php
wordpress/wp-content/themes/twentytwentytwo/inc/patterns/header-image-background.php
wordpress/wp-content/themes/twentytwentytwo/inc/block-patterns.php
wordpress/wp-content/themes/twentytwentytwo/parts/
wordpress/wp-content/themes/twentytwentytwo/parts/header-small-dark.html
wordpress/wp-content/themes/twentytwentytwo/parts/header-large-dark.html
wordpress/wp-content/themes/twentytwentytwo/parts/footer.html
wordpress/wp-content/themes/twentytwentytwo/parts/header.html
wordpress/wp-content/themes/twentytwentytwo/styles/
wordpress/wp-content/themes/twentytwentytwo/styles/swiss.json
wordpress/wp-content/themes/twentytwentytwo/styles/blue.json
wordpress/wp-content/themes/twentytwentytwo/styles/pink.json
wordpress/wp-content/themes/twentytwentytwo/assets/
wordpress/wp-content/themes/twentytwentytwo/assets/images/
wordpress/wp-content/themes/twentytwentytwo/assets/images/divider-white.png
wordpress/wp-content/themes/twentytwentytwo/assets/images/flight-path-on-gray-a.jpg
wordpress/wp-content/themes/twentytwentytwo/assets/images/bird-on-gray.jpg
wordpress/wp-content/themes/twentytwentytwo/assets/images/flight-path-on-transparent-d.png
wordpress/wp-content/themes/twentytwentytwo/assets/images/icon-bird.jpg
wordpress/wp-content/themes/twentytwentytwo/assets/images/flight-path-on-gray-c.jpg
wordpress/wp-content/themes/twentytwentytwo/assets/images/flight-path-on-transparent-b.png
wordpress/wp-content/themes/twentytwentytwo/assets/images/flight-path-on-transparent-a.png
wordpress/wp-content/themes/twentytwentytwo/assets/images/bird-on-green.jpg
wordpress/wp-content/themes/twentytwentytwo/assets/images/flight-path-on-gray-b.jpg
wordpress/wp-content/themes/twentytwentytwo/assets/images/bird-on-salmon.jpg
wordpress/wp-content/themes/twentytwentytwo/assets/images/flight-path-on-salmon.jpg
wordpress/wp-content/themes/twentytwentytwo/assets/images/divider-black.png
wordpress/wp-content/themes/twentytwentytwo/assets/images/ducks.jpg
wordpress/wp-content/themes/twentytwentytwo/assets/images/icon-binoculars.png
wordpress/wp-content/themes/twentytwentytwo/assets/images/flight-path-on-transparent-c.png
wordpress/wp-content/themes/twentytwentytwo/assets/images/bird-on-black.jpg
wordpress/wp-content/themes/twentytwentytwo/assets/videos/
wordpress/wp-content/themes/twentytwentytwo/assets/videos/birds.mp4
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/SourceSerif4Variable-Roman.ttf.woff2
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/SourceSerif4Variable-Italic.otf.woff2
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/SourceSerif4Variable-Italic.ttf.woff2
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/source-serif-pro/
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/source-serif-pro/SourceSerif4Variable-Roman.ttf.woff2
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/source-serif-pro/SourceSerif4Variable-Italic.otf.woff2
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/source-serif-pro/SourceSerif4Variable-Italic.ttf.woff2
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/source-serif-pro/SourceSerif4Variable-Roman.otf.woff2
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/source-serif-pro/LICENSE.md
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/ibm-plex/
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/ibm-plex/IBMPlexSans-ExtraLightItalic.woff2
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/ibm-plex/IBMPlexSans-LightItalic.woff2
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/ibm-plex/IBMPlexSans-Light.woff2
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/ibm-plex/IBMPlexMono-BoldItalic.woff2
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/ibm-plex/IBMPlexMono-TextItalic.woff2
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/ibm-plex/IBMPlexMono-Text.woff2
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/ibm-plex/LICENSE.txt
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/ibm-plex/IBMPlexMono-Bold.woff2
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/ibm-plex/IBMPlexSans-ExtraLight.woff2
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/SourceSerif4Variable-Roman.otf.woff2
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/inter/
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/inter/Inter.ttf
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/inter/LICENSE.txt
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/dm-sans/
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/dm-sans/DMSans-BoldItalic.ttf
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/dm-sans/DMSans-Bold.ttf
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/dm-sans/DMSans-Italic.ttf
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/dm-sans/DMSans-Regular.ttf
wordpress/wp-content/themes/twentytwentytwo/assets/fonts/dm-sans/LICENSE.txt
wordpress/wp-content/themes/twentytwentytwo/screenshot.png
wordpress/wp-content/themes/twentytwentytwo/index.php
wordpress/wp-content/themes/twentytwentytwo/functions.php
wordpress/wp-content/themes/twentytwentytwo/readme.txt
wordpress/wp-content/themes/twentytwentytwo/templates/
wordpress/wp-content/themes/twentytwentytwo/templates/blank.html
wordpress/wp-content/themes/twentytwentytwo/templates/page-no-separators.html
wordpress/wp-content/themes/twentytwentytwo/templates/page.html
wordpress/wp-content/themes/twentytwentytwo/templates/index.html
wordpress/wp-content/themes/twentytwentytwo/templates/single-no-separators.html
wordpress/wp-content/themes/twentytwentytwo/templates/archive.html
wordpress/wp-content/themes/twentytwentytwo/templates/search.html
wordpress/wp-content/themes/twentytwentytwo/templates/page-large-header.html
wordpress/wp-content/themes/twentytwentytwo/templates/single.html
wordpress/wp-content/themes/twentytwentytwo/templates/404.html
wordpress/wp-content/themes/twentytwentytwo/templates/home.html
wordpress/wp-content/themes/twentytwentytwo/style.css
wordpress/wp-content/themes/index.php
wordpress/wp-content/themes/twentytwentyfour/
wordpress/wp-content/themes/twentytwentyfour/theme.json
wordpress/wp-content/themes/twentytwentyfour/parts/
wordpress/wp-content/themes/twentytwentyfour/parts/sidebar.html
wordpress/wp-content/themes/twentytwentyfour/parts/footer.html
wordpress/wp-content/themes/twentytwentyfour/parts/header.html
wordpress/wp-content/themes/twentytwentyfour/parts/post-meta.html
wordpress/wp-content/themes/twentytwentyfour/patterns/
wordpress/wp-content/themes/twentytwentyfour/patterns/cta-pricing.php
wordpress/wp-content/themes/twentytwentyfour/patterns/page-home-portfolio-gallery.php
wordpress/wp-content/themes/twentytwentyfour/patterns/template-home-blogging.php
wordpress/wp-content/themes/twentytwentyfour/patterns/template-home-portfolio.php
wordpress/wp-content/themes/twentytwentyfour/patterns/hidden-404.php
wordpress/wp-content/themes/twentytwentyfour/patterns/posts-1-col.php
wordpress/wp-content/themes/twentytwentyfour/patterns/footer.php
wordpress/wp-content/themes/twentytwentyfour/patterns/template-search-blogging.php
wordpress/wp-content/themes/twentytwentyfour/patterns/template-search-portfolio.php
wordpress/wp-content/themes/twentytwentyfour/patterns/gallery-project-layout.php
wordpress/wp-content/themes/twentytwentyfour/patterns/hidden-post-navigation.php
wordpress/wp-content/themes/twentytwentyfour/patterns/team-4-col.php
wordpress/wp-content/themes/twentytwentyfour/patterns/text-centered-statement-small.php
wordpress/wp-content/themes/twentytwentyfour/patterns/page-home-business.php
wordpress/wp-content/themes/twentytwentyfour/patterns/banner-project-description.php
wordpress/wp-content/themes/twentytwentyfour/patterns/text-centered-statement.php
wordpress/wp-content/themes/twentytwentyfour/patterns/template-index-blogging.php
wordpress/wp-content/themes/twentytwentyfour/patterns/posts-images-only-offset-4-col.php
wordpress/wp-content/themes/twentytwentyfour/patterns/page-home-blogging.php
wordpress/wp-content/themes/twentytwentyfour/patterns/hidden-search.php
wordpress/wp-content/themes/twentytwentyfour/patterns/template-archive-blogging.php
wordpress/wp-content/themes/twentytwentyfour/patterns/banner-hero.php
wordpress/wp-content/themes/twentytwentyfour/patterns/gallery-offset-images-grid-3-col.php
wordpress/wp-content/themes/twentytwentyfour/patterns/text-project-details.php
wordpress/wp-content/themes/twentytwentyfour/patterns/hidden-no-results.php
wordpress/wp-content/themes/twentytwentyfour/patterns/gallery-full-screen-image.php
wordpress/wp-content/themes/twentytwentyfour/patterns/posts-list.php
wordpress/wp-content/themes/twentytwentyfour/patterns/text-title-left-image-right.php
wordpress/wp-content/themes/twentytwentyfour/patterns/page-portfolio-overview.php
wordpress/wp-content/themes/twentytwentyfour/patterns/template-home-business.php
wordpress/wp-content/themes/twentytwentyfour/patterns/page-home-portfolio.php
wordpress/wp-content/themes/twentytwentyfour/patterns/page-newsletter-landing.php
wordpress/wp-content/themes/twentytwentyfour/patterns/hidden-post-meta.php
wordpress/wp-content/themes/twentytwentyfour/patterns/posts-grid-2-col.php
wordpress/wp-content/themes/twentytwentyfour/patterns/cta-subscribe-centered.php
wordpress/wp-content/themes/twentytwentyfour/patterns/page-rsvp-landing.php
wordpress/wp-content/themes/twentytwentyfour/patterns/cta-services-image-left.php
wordpress/wp-content/themes/twentytwentyfour/patterns/template-archive-portfolio.php
wordpress/wp-content/themes/twentytwentyfour/patterns/cta-content-image-on-right.php
wordpress/wp-content/themes/twentytwentyfour/patterns/gallery-offset-images-grid-4-col.php
wordpress/wp-content/themes/twentytwentyfour/patterns/footer-colophon-3-col.php
wordpress/wp-content/themes/twentytwentyfour/patterns/text-alternating-images.php
wordpress/wp-content/themes/twentytwentyfour/patterns/hidden-comments.php
wordpress/wp-content/themes/twentytwentyfour/patterns/footer-centered-logo-nav.php
wordpress/wp-content/themes/twentytwentyfour/patterns/text-feature-grid-3-col.php
wordpress/wp-content/themes/twentytwentyfour/patterns/posts-3-col.php
wordpress/wp-content/themes/twentytwentyfour/patterns/hidden-portfolio-hero.php
wordpress/wp-content/themes/twentytwentyfour/patterns/cta-rsvp.php
wordpress/wp-content/themes/twentytwentyfour/patterns/template-index-portfolio.php
wordpress/wp-content/themes/twentytwentyfour/patterns/testimonial-centered.php
wordpress/wp-content/themes/twentytwentyfour/patterns/template-single-portfolio.php
wordpress/wp-content/themes/twentytwentyfour/patterns/gallery-offset-images-grid-2-col.php
wordpress/wp-content/themes/twentytwentyfour/patterns/hidden-sidebar.php
wordpress/wp-content/themes/twentytwentyfour/patterns/posts-images-only-3-col.php
wordpress/wp-content/themes/twentytwentyfour/patterns/page-about-business.php
wordpress/wp-content/themes/twentytwentyfour/patterns/text-faq.php
wordpress/wp-content/themes/twentytwentyfour/styles/
wordpress/wp-content/themes/twentytwentyfour/styles/rust.json
wordpress/wp-content/themes/twentytwentyfour/styles/maelstrom.json
wordpress/wp-content/themes/twentytwentyfour/styles/onyx.json
wordpress/wp-content/themes/twentytwentyfour/styles/fossil.json
wordpress/wp-content/themes/twentytwentyfour/styles/ice.json
wordpress/wp-content/themes/twentytwentyfour/styles/mint.json
wordpress/wp-content/themes/twentytwentyfour/styles/ember.json
wordpress/wp-content/themes/twentytwentyfour/assets/
wordpress/wp-content/themes/twentytwentyfour/assets/images/
wordpress/wp-content/themes/twentytwentyfour/assets/images/windows.webp
wordpress/wp-content/themes/twentytwentyfour/assets/images/museum.webp
wordpress/wp-content/themes/twentytwentyfour/assets/images/art-gallery.webp
wordpress/wp-content/themes/twentytwentyfour/assets/images/abstract-geometric-art.webp
wordpress/wp-content/themes/twentytwentyfour/assets/images/green-staircase.webp
wordpress/wp-content/themes/twentytwentyfour/assets/images/icon-message.webp
wordpress/wp-content/themes/twentytwentyfour/assets/images/building-exterior.webp
wordpress/wp-content/themes/twentytwentyfour/assets/images/tourist-and-building.webp
wordpress/wp-content/themes/twentytwentyfour/assets/images/hotel-facade.webp
wordpress/wp-content/themes/twentytwentyfour/assets/images/angular-roof.webp
wordpress/wp-content/themes/twentytwentyfour/assets/css/
wordpress/wp-content/themes/twentytwentyfour/assets/css/button-outline.css
wordpress/wp-content/themes/twentytwentyfour/assets/fonts/
wordpress/wp-content/themes/twentytwentyfour/assets/fonts/instrument-sans/
wordpress/wp-content/themes/twentytwentyfour/assets/fonts/instrument-sans/InstrumentSans-VariableFont_wdth,wght.woff2
wordpress/wp-content/themes/twentytwentyfour/assets/fonts/instrument-sans/InstrumentSans-Italic-VariableFont_wdth,wght.woff2
wordpress/wp-content/themes/twentytwentyfour/assets/fonts/instrument-sans/OFL.txt
wordpress/wp-content/themes/twentytwentyfour/assets/fonts/jost/
wordpress/wp-content/themes/twentytwentyfour/assets/fonts/jost/Jost-Italic-VariableFont_wght.woff2
wordpress/wp-content/themes/twentytwentyfour/assets/fonts/jost/OFL.txt
wordpress/wp-content/themes/twentytwentyfour/assets/fonts/jost/Jost-VariableFont_wght.woff2
wordpress/wp-content/themes/twentytwentyfour/assets/fonts/cardo/
wordpress/wp-content/themes/twentytwentyfour/assets/fonts/cardo/cardo_normal_400.woff2
wordpress/wp-content/themes/twentytwentyfour/assets/fonts/cardo/LICENSE.txt
wordpress/wp-content/themes/twentytwentyfour/assets/fonts/cardo/cardo_italic_400.woff2
wordpress/wp-content/themes/twentytwentyfour/assets/fonts/cardo/cardo_normal_700.woff2
wordpress/wp-content/themes/twentytwentyfour/assets/fonts/inter/
wordpress/wp-content/themes/twentytwentyfour/assets/fonts/inter/LICENSE.txt
wordpress/wp-content/themes/twentytwentyfour/assets/fonts/inter/Inter-VariableFont_slnt,wght.woff2
wordpress/wp-content/themes/twentytwentyfour/screenshot.png
wordpress/wp-content/themes/twentytwentyfour/functions.php
wordpress/wp-content/themes/twentytwentyfour/readme.txt
wordpress/wp-content/themes/twentytwentyfour/templates/
wordpress/wp-content/themes/twentytwentyfour/templates/page.html
wordpress/wp-content/themes/twentytwentyfour/templates/index.html
wordpress/wp-content/themes/twentytwentyfour/templates/single-with-sidebar.html
wordpress/wp-content/themes/twentytwentyfour/templates/page-wide.html
wordpress/wp-content/themes/twentytwentyfour/templates/page-with-sidebar.html
wordpress/wp-content/themes/twentytwentyfour/templates/archive.html
wordpress/wp-content/themes/twentytwentyfour/templates/page-no-title.html
wordpress/wp-content/themes/twentytwentyfour/templates/search.html
wordpress/wp-content/themes/twentytwentyfour/templates/single.html
wordpress/wp-content/themes/twentytwentyfour/templates/404.html
wordpress/wp-content/themes/twentytwentyfour/templates/home.html
wordpress/wp-content/themes/twentytwentyfour/style.css
wordpress/wp-content/index.php
wordpress/wp-content/plugins/
wordpress/wp-content/plugins/akismet/
wordpress/wp-content/plugins/akismet/akismet.php
wordpress/wp-content/plugins/akismet/.htaccess
wordpress/wp-content/plugins/akismet/class.akismet-admin.php
wordpress/wp-content/plugins/akismet/class.akismet-cli.php
wordpress/wp-content/plugins/akismet/class.akismet-widget.php
wordpress/wp-content/plugins/akismet/class.akismet.php
wordpress/wp-content/plugins/akismet/views/
wordpress/wp-content/plugins/akismet/views/title.php
wordpress/wp-content/plugins/akismet/views/connect-jp.php
wordpress/wp-content/plugins/akismet/views/notice.php
wordpress/wp-content/plugins/akismet/views/config.php
wordpress/wp-content/plugins/akismet/views/stats.php
wordpress/wp-content/plugins/akismet/views/logo.php
wordpress/wp-content/plugins/akismet/views/activate.php
wordpress/wp-content/plugins/akismet/views/get.php
wordpress/wp-content/plugins/akismet/views/setup.php
wordpress/wp-content/plugins/akismet/views/predefined.php
wordpress/wp-content/plugins/akismet/views/enter.php
wordpress/wp-content/plugins/akismet/views/start.php
wordpress/wp-content/plugins/akismet/_inc/
wordpress/wp-content/plugins/akismet/_inc/rtl/
wordpress/wp-content/plugins/akismet/_inc/rtl/akismet-rtl.css
wordpress/wp-content/plugins/akismet/_inc/rtl/akismet-admin-rtl.css
wordpress/wp-content/plugins/akismet/_inc/akismet.css
wordpress/wp-content/plugins/akismet/_inc/img/
wordpress/wp-content/plugins/akismet/_inc/img/akismet-refresh-logo@2x.png
wordpress/wp-content/plugins/akismet/_inc/img/akismet-refresh-logo.svg
wordpress/wp-content/plugins/akismet/_inc/img/arrow-left.svg
wordpress/wp-content/plugins/akismet/_inc/img/logo-full-2x.png
wordpress/wp-content/plugins/akismet/_inc/img/logo-a-2x.png
wordpress/wp-content/plugins/akismet/_inc/img/icon-external.svg
wordpress/wp-content/plugins/akismet/_inc/fonts/
wordpress/wp-content/plugins/akismet/_inc/fonts/inter.css
wordpress/wp-content/plugins/akismet/_inc/akismet-admin.css
wordpress/wp-content/plugins/akismet/_inc/akismet-frontend.js
wordpress/wp-content/plugins/akismet/_inc/akismet-admin.js
wordpress/wp-content/plugins/akismet/_inc/akismet.js
wordpress/wp-content/plugins/akismet/changelog.txt
wordpress/wp-content/plugins/akismet/wrapper.php
wordpress/wp-content/plugins/akismet/index.php
wordpress/wp-content/plugins/akismet/LICENSE.txt
wordpress/wp-content/plugins/akismet/class.akismet-rest-api.php
wordpress/wp-content/plugins/akismet/readme.txt
wordpress/wp-content/plugins/index.php
wordpress/wp-content/plugins/hello.php
wordpress/wp-mail.php
wordpress/wp-links-opml.php
wordpress/wp-load.php
wordpress/wp-includes/
wordpress/wp-includes/class-wp-styles.php
wordpress/wp-includes/class-wp-user-query.php
wordpress/wp-includes/l10n.php
wordpress/wp-includes/date.php
wordpress/wp-includes/php-compat/
wordpress/wp-includes/php-compat/readonly.php
wordpress/wp-includes/class-wp-oembed.php
wordpress/wp-includes/images/
wordpress/wp-includes/images/w-logo-blue-white-bg.png
wordpress/wp-includes/images/blank.gif
wordpress/wp-includes/images/down_arrow.gif
wordpress/wp-includes/images/spinner.gif
wordpress/wp-includes/images/media/
wordpress/wp-includes/images/media/document.svg
wordpress/wp-includes/images/media/interactive.png
wordpress/wp-includes/images/media/text.svg
wordpress/wp-includes/images/media/video.svg
wordpress/wp-includes/images/media/default.png
wordpress/wp-includes/images/media/code.svg
wordpress/wp-includes/images/media/code.png
wordpress/wp-includes/images/media/audio.png
wordpress/wp-includes/images/media/text.png
wordpress/wp-includes/images/media/default.svg
wordpress/wp-includes/images/media/interactive.svg
wordpress/wp-includes/images/media/archive.png
wordpress/wp-includes/images/media/video.png
wordpress/wp-includes/images/media/spreadsheet.png
wordpress/wp-includes/images/media/audio.svg
wordpress/wp-includes/images/media/document.png
wordpress/wp-includes/images/media/archive.svg
wordpress/wp-includes/images/media/spreadsheet.svg
wordpress/wp-includes/images/uploader-icons.png
wordpress/wp-includes/images/xit-2x.gif
wordpress/wp-includes/images/w-logo-blue.png
wordpress/wp-includes/images/wpicons-2x.png
wordpress/wp-includes/images/admin-bar-sprite-2x.png
wordpress/wp-includes/images/rss-2x.png
wordpress/wp-includes/images/spinner-2x.gif
wordpress/wp-includes/images/wpicons.png
wordpress/wp-includes/images/icon-pointer-flag.png
wordpress/wp-includes/images/down_arrow-2x.gif
wordpress/wp-includes/images/arrow-pointer-blue-2x.png
wordpress/wp-includes/images/rss.png
wordpress/wp-includes/images/toggle-arrow-2x.png
wordpress/wp-includes/images/xit.gif
wordpress/wp-includes/images/smilies/
wordpress/wp-includes/images/smilies/icon_rolleyes.gif
wordpress/wp-includes/images/smilies/icon_razz.gif
wordpress/wp-includes/images/smilies/icon_question.gif
wordpress/wp-includes/images/smilies/rolleyes.png
wordpress/wp-includes/images/smilies/icon_surprised.gif
wordpress/wp-includes/images/smilies/icon_exclaim.gif
wordpress/wp-includes/images/smilies/icon_lol.gif
wordpress/wp-includes/images/smilies/icon_eek.gif
wordpress/wp-includes/images/smilies/icon_neutral.gif
wordpress/wp-includes/images/smilies/icon_confused.gif
wordpress/wp-includes/images/smilies/icon_sad.gif
wordpress/wp-includes/images/smilies/icon_mad.gif
wordpress/wp-includes/images/smilies/icon_mrgreen.gif
wordpress/wp-includes/images/smilies/mrgreen.png
wordpress/wp-includes/images/smilies/frownie.png
wordpress/wp-includes/images/smilies/icon_wink.gif
wordpress/wp-includes/images/smilies/icon_redface.gif
wordpress/wp-includes/images/smilies/icon_idea.gif
wordpress/wp-includes/images/smilies/icon_twisted.gif
wordpress/wp-includes/images/smilies/icon_biggrin.gif
wordpress/wp-includes/images/smilies/icon_evil.gif
wordpress/wp-includes/images/smilies/simple-smile.png
wordpress/wp-includes/images/smilies/icon_cry.gif
wordpress/wp-includes/images/smilies/icon_cool.gif
wordpress/wp-includes/images/smilies/icon_arrow.gif
wordpress/wp-includes/images/smilies/icon_smile.gif
wordpress/wp-includes/images/wpspin-2x.gif
wordpress/wp-includes/images/wpspin.gif
wordpress/wp-includes/images/uploader-icons-2x.png
wordpress/wp-includes/images/crystal/
wordpress/wp-includes/images/crystal/interactive.png
wordpress/wp-includes/images/crystal/default.png
wordpress/wp-includes/images/crystal/code.png
wordpress/wp-includes/images/crystal/audio.png
wordpress/wp-includes/images/crystal/text.png
wordpress/wp-includes/images/crystal/archive.png
wordpress/wp-includes/images/crystal/video.png
wordpress/wp-includes/images/crystal/spreadsheet.png
wordpress/wp-includes/images/crystal/license.txt
wordpress/wp-includes/images/crystal/document.png
wordpress/wp-includes/images/icon-pointer-flag-2x.png
wordpress/wp-includes/images/admin-bar-sprite.png
wordpress/wp-includes/images/toggle-arrow.png
wordpress/wp-includes/images/arrow-pointer-blue.png
wordpress/wp-includes/locale.php
wordpress/wp-includes/class-wp-dependencies.php
wordpress/wp-includes/class-wp-feed-cache-transient.php
wordpress/wp-includes/class-wp-recovery-mode-email-service.php
wordpress/wp-includes/block-supports/
wordpress/wp-includes/block-supports/position.php
wordpress/wp-includes/block-supports/background.php
wordpress/wp-includes/block-supports/custom-classname.php
wordpress/wp-includes/block-supports/border.php
wordpress/wp-includes/block-supports/spacing.php
wordpress/wp-includes/block-supports/align.php
wordpress/wp-includes/block-supports/generated-classname.php
wordpress/wp-includes/block-supports/typography.php
wordpress/wp-includes/block-supports/elements.php
wordpress/wp-includes/block-supports/duotone.php
wordpress/wp-includes/block-supports/shadow.php
wordpress/wp-includes/block-supports/settings.php
wordpress/wp-includes/block-supports/colors.php
wordpress/wp-includes/block-supports/utils.php
wordpress/wp-includes/block-supports/dimensions.php
wordpress/wp-includes/block-supports/layout.php
wordpress/wp-includes/class-wp-comment-query.php
wordpress/wp-includes/theme-compat/
wordpress/wp-includes/theme-compat/footer.php
wordpress/wp-includes/theme-compat/sidebar.php
wordpress/wp-includes/theme-compat/footer-embed.php
wordpress/wp-includes/theme-compat/embed.php
wordpress/wp-includes/theme-compat/header-embed.php
wordpress/wp-includes/theme-compat/header.php
wordpress/wp-includes/theme-compat/embed-content.php
wordpress/wp-includes/theme-compat/comments.php
wordpress/wp-includes/theme-compat/embed-404.php
wordpress/wp-includes/class-wp-customize-control.php
wordpress/wp-includes/SimplePie/
wordpress/wp-includes/SimplePie/Parser.php
wordpress/wp-includes/SimplePie/Sanitize.php
wordpress/wp-includes/SimplePie/Core.php
wordpress/wp-includes/SimplePie/Decode/
wordpress/wp-includes/SimplePie/Decode/HTML/
wordpress/wp-includes/SimplePie/Decode/HTML/Entities.php
wordpress/wp-includes/SimplePie/Category.php
wordpress/wp-includes/SimplePie/gzdecode.php
wordpress/wp-includes/SimplePie/Locator.php
wordpress/wp-includes/SimplePie/Parse/
wordpress/wp-includes/SimplePie/Parse/Date.php
wordpress/wp-includes/SimplePie/Author.php
wordpress/wp-includes/SimplePie/Cache.php
wordpress/wp-includes/SimplePie/IRI.php
wordpress/wp-includes/SimplePie/Credit.php
wordpress/wp-includes/SimplePie/Content/
wordpress/wp-includes/SimplePie/Content/Type/
wordpress/wp-includes/SimplePie/Content/Type/Sniffer.php
wordpress/wp-includes/SimplePie/Restriction.php
wordpress/wp-includes/SimplePie/Item.php
wordpress/wp-includes/SimplePie/XML/
wordpress/wp-includes/SimplePie/XML/Declaration/
wordpress/wp-includes/SimplePie/XML/Declaration/Parser.php
wordpress/wp-includes/SimplePie/Cache/
wordpress/wp-includes/SimplePie/Cache/Base.php
wordpress/wp-includes/SimplePie/Cache/Memcached.php
wordpress/wp-includes/SimplePie/Cache/MySQL.php
wordpress/wp-includes/SimplePie/Cache/Memcache.php
wordpress/wp-includes/SimplePie/Cache/Redis.php
wordpress/wp-includes/SimplePie/Cache/DB.php
wordpress/wp-includes/SimplePie/Cache/File.php
wordpress/wp-includes/SimplePie/Source.php
wordpress/wp-includes/SimplePie/Registry.php
wordpress/wp-includes/SimplePie/Rating.php
wordpress/wp-includes/SimplePie/Copyright.php
wordpress/wp-includes/SimplePie/Exception.php
wordpress/wp-includes/SimplePie/Misc.php
wordpress/wp-includes/SimplePie/Caption.php
wordpress/wp-includes/SimplePie/Net/
wordpress/wp-includes/SimplePie/Net/IPv6.php
wordpress/wp-includes/SimplePie/File.php
wordpress/wp-includes/SimplePie/HTTP/
wordpress/wp-includes/SimplePie/HTTP/Parser.php
wordpress/wp-includes/SimplePie/Enclosure.php
wordpress/wp-includes/media-template.php
wordpress/wp-includes/cache.php
wordpress/wp-includes/class-wp-widget.php
wordpress/wp-includes/compat.php
wordpress/wp-includes/default-filters.php
wordpress/wp-includes/Requests/
wordpress/wp-includes/Requests/src/
wordpress/wp-includes/Requests/src/Transport/
wordpress/wp-includes/Requests/src/Transport/Fsockopen.php
wordpress/wp-includes/Requests/src/Transport/Curl.php
wordpress/wp-includes/Requests/src/Cookie/
wordpress/wp-includes/Requests/src/Cookie/Jar.php
wordpress/wp-includes/Requests/src/Requests.php
wordpress/wp-includes/Requests/src/Auth.php
wordpress/wp-includes/Requests/src/Port.php
wordpress/wp-includes/Requests/src/HookManager.php
wordpress/wp-includes/Requests/src/Exception/
wordpress/wp-includes/Requests/src/Exception/Transport/
wordpress/wp-includes/Requests/src/Exception/Transport/Curl.php
wordpress/wp-includes/Requests/src/Exception/ArgumentCount.php
wordpress/wp-includes/Requests/src/Exception/Transport.php
wordpress/wp-includes/Requests/src/Exception/Http.php
wordpress/wp-includes/Requests/src/Exception/InvalidArgument.php
wordpress/wp-includes/Requests/src/Exception/Http/
wordpress/wp-includes/Requests/src/Exception/Http/Status304.php
wordpress/wp-includes/Requests/src/Exception/Http/Status403.php
wordpress/wp-includes/Requests/src/Exception/Http/Status306.php
wordpress/wp-includes/Requests/src/Exception/Http/Status401.php
wordpress/wp-includes/Requests/src/Exception/Http/Status402.php
wordpress/wp-includes/Requests/src/Exception/Http/Status417.php
wordpress/wp-includes/Requests/src/Exception/Http/Status505.php
wordpress/wp-includes/Requests/src/Exception/Http/Status418.php
wordpress/wp-includes/Requests/src/Exception/Http/Status414.php
wordpress/wp-includes/Requests/src/Exception/Http/Status504.php
wordpress/wp-includes/Requests/src/Exception/Http/Status413.php
wordpress/wp-includes/Requests/src/Exception/Http/Status429.php
wordpress/wp-includes/Requests/src/Exception/Http/Status503.php
wordpress/wp-includes/Requests/src/Exception/Http/Status415.php
wordpress/wp-includes/Requests/src/Exception/Http/Status502.php
wordpress/wp-includes/Requests/src/Exception/Http/Status416.php
wordpress/wp-includes/Requests/src/Exception/Http/Status305.php
wordpress/wp-includes/Requests/src/Exception/Http/Status408.php
wordpress/wp-includes/Requests/src/Exception/Http/Status511.php
wordpress/wp-includes/Requests/src/Exception/Http/Status407.php
wordpress/wp-includes/Requests/src/Exception/Http/Status411.php
wordpress/wp-includes/Requests/src/Exception/Http/Status410.php
wordpress/wp-includes/Requests/src/Exception/Http/Status431.php
wordpress/wp-includes/Requests/src/Exception/Http/Status409.php
wordpress/wp-includes/Requests/src/Exception/Http/Status428.php
wordpress/wp-includes/Requests/src/Exception/Http/Status405.php
wordpress/wp-includes/Requests/src/Exception/Http/Status501.php
wordpress/wp-includes/Requests/src/Exception/Http/Status406.php
wordpress/wp-includes/Requests/src/Exception/Http/Status404.php
wordpress/wp-includes/Requests/src/Exception/Http/Status400.php
wordpress/wp-includes/Requests/src/Exception/Http/Status412.php
wordpress/wp-includes/Requests/src/Exception/Http/Status500.php
wordpress/wp-includes/Requests/src/Exception/Http/StatusUnknown.php
wordpress/wp-includes/Requests/src/Autoload.php
wordpress/wp-includes/Requests/src/Response.php
wordpress/wp-includes/Requests/src/Utility/
wordpress/wp-includes/Requests/src/Utility/InputValidator.php
wordpress/wp-includes/Requests/src/Utility/CaseInsensitiveDictionary.php
wordpress/wp-includes/Requests/src/Utility/FilteredIterator.php
wordpress/wp-includes/Requests/src/Response/
wordpress/wp-includes/Requests/src/Response/Headers.php
wordpress/wp-includes/Requests/src/Cookie.php
wordpress/wp-includes/Requests/src/Auth/
wordpress/wp-includes/Requests/src/Auth/Basic.php
wordpress/wp-includes/Requests/src/Proxy.php
wordpress/wp-includes/Requests/src/Iri.php
wordpress/wp-includes/Requests/src/Ipv6.php
wordpress/wp-includes/Requests/src/Transport.php
wordpress/wp-includes/Requests/src/Session.php
wordpress/wp-includes/Requests/src/Capability.php
wordpress/wp-includes/Requests/src/Hooks.php
wordpress/wp-includes/Requests/src/Proxy/
wordpress/wp-includes/Requests/src/Proxy/Http.php
wordpress/wp-includes/Requests/src/Ssl.php
wordpress/wp-includes/Requests/src/Exception.php
wordpress/wp-includes/Requests/src/IdnaEncoder.php
wordpress/wp-includes/Requests/library/
wordpress/wp-includes/Requests/library/Requests.php
wordpress/wp-includes/embed.php
wordpress/wp-includes/block-template.php
wordpress/wp-includes/class-wp-fatal-error-handler.php
wordpress/wp-includes/theme.json
wordpress/wp-includes/css/
wordpress/wp-includes/css/admin-bar.min.css
wordpress/wp-includes/css/editor.min.css
wordpress/wp-includes/css/admin-bar-rtl.min.css
wordpress/wp-includes/css/media-views.min.css
wordpress/wp-includes/css/classic-themes.css
wordpress/wp-includes/css/customize-preview.css
wordpress/wp-includes/css/classic-themes.min.css
wordpress/wp-includes/css/admin-bar-rtl.css
wordpress/wp-includes/css/editor.css
wordpress/wp-includes/css/wp-embed-template.min.css
wordpress/wp-includes/css/wp-pointer.css
wordpress/wp-includes/css/customize-preview-rtl.css
wordpress/wp-includes/css/wp-pointer-rtl.css
wordpress/wp-includes/css/jquery-ui-dialog-rtl.min.css
wordpress/wp-includes/css/dist/
wordpress/wp-includes/css/dist/preferences/
wordpress/wp-includes/css/dist/preferences/style-rtl.min.css
wordpress/wp-includes/css/dist/preferences/style.min.css
wordpress/wp-includes/css/dist/preferences/style-rtl.css
wordpress/wp-includes/css/dist/preferences/style.css
wordpress/wp-includes/css/dist/edit-widgets/
wordpress/wp-includes/css/dist/edit-widgets/style-rtl.min.css
wordpress/wp-includes/css/dist/edit-widgets/style.min.css
wordpress/wp-includes/css/dist/edit-widgets/style-rtl.css
wordpress/wp-includes/css/dist/edit-widgets/style.css
wordpress/wp-includes/css/dist/patterns/
wordpress/wp-includes/css/dist/patterns/style-rtl.min.css
wordpress/wp-includes/css/dist/patterns/style.min.css
wordpress/wp-includes/css/dist/patterns/style-rtl.css
wordpress/wp-includes/css/dist/patterns/style.css
wordpress/wp-includes/css/dist/block-directory/
wordpress/wp-includes/css/dist/block-directory/style-rtl.min.css
wordpress/wp-includes/css/dist/block-directory/style.min.css
wordpress/wp-includes/css/dist/block-directory/style-rtl.css
wordpress/wp-includes/css/dist/block-directory/style.css
wordpress/wp-includes/css/dist/edit-site/
wordpress/wp-includes/css/dist/edit-site/style-rtl.min.css
wordpress/wp-includes/css/dist/edit-site/style.min.css
wordpress/wp-includes/css/dist/edit-site/style-rtl.css
wordpress/wp-includes/css/dist/edit-site/style.css
wordpress/wp-includes/css/dist/block-library/
wordpress/wp-includes/css/dist/block-library/classic-rtl.css
wordpress/wp-includes/css/dist/block-library/editor.min.css
wordpress/wp-includes/css/dist/block-library/elements-rtl.css
wordpress/wp-includes/css/dist/block-library/reset-rtl.css
wordpress/wp-includes/css/dist/block-library/style-rtl.min.css
wordpress/wp-includes/css/dist/block-library/theme.css
wordpress/wp-includes/css/dist/block-library/theme-rtl.min.css
wordpress/wp-includes/css/dist/block-library/editor-elements-rtl.css
wordpress/wp-includes/css/dist/block-library/common-rtl.min.css
wordpress/wp-includes/css/dist/block-library/elements-rtl.min.css
wordpress/wp-includes/css/dist/block-library/editor.css
wordpress/wp-includes/css/dist/block-library/classic-rtl.min.css
wordpress/wp-includes/css/dist/block-library/editor-elements.min.css
wordpress/wp-includes/css/dist/block-library/reset-rtl.min.css
wordpress/wp-includes/css/dist/block-library/common.css
wordpress/wp-includes/css/dist/block-library/theme.min.css
wordpress/wp-includes/css/dist/block-library/elements.min.css
wordpress/wp-includes/css/dist/block-library/style.min.css
wordpress/wp-includes/css/dist/block-library/theme-rtl.css
wordpress/wp-includes/css/dist/block-library/classic.css
wordpress/wp-includes/css/dist/block-library/editor-rtl.css
wordpress/wp-includes/css/dist/block-library/editor-rtl.min.css
wordpress/wp-includes/css/dist/block-library/style-rtl.css
wordpress/wp-includes/css/dist/block-library/classic.min.css
wordpress/wp-includes/css/dist/block-library/common.min.css
wordpress/wp-includes/css/dist/block-library/reset.min.css
wordpress/wp-includes/css/dist/block-library/elements.css
wordpress/wp-includes/css/dist/block-library/reset.css
wordpress/wp-includes/css/dist/block-library/editor-elements.css
wordpress/wp-includes/css/dist/block-library/editor-elements-rtl.min.css
wordpress/wp-includes/css/dist/block-library/common-rtl.css
wordpress/wp-includes/css/dist/block-library/style.css
wordpress/wp-includes/css/dist/format-library/
wordpress/wp-includes/css/dist/format-library/style-rtl.min.css
wordpress/wp-includes/css/dist/format-library/style.min.css
wordpress/wp-includes/css/dist/format-library/style-rtl.css
wordpress/wp-includes/css/dist/format-library/style.css
wordpress/wp-includes/css/dist/edit-post/
wordpress/wp-includes/css/dist/edit-post/classic-rtl.css
wordpress/wp-includes/css/dist/edit-post/style-rtl.min.css
wordpress/wp-includes/css/dist/edit-post/classic-rtl.min.css
wordpress/wp-includes/css/dist/edit-post/style.min.css
wordpress/wp-includes/css/dist/edit-post/classic.css
wordpress/wp-includes/css/dist/edit-post/style-rtl.css
wordpress/wp-includes/css/dist/edit-post/classic.min.css
wordpress/wp-includes/css/dist/edit-post/style.css
wordpress/wp-includes/css/dist/widgets/
wordpress/wp-includes/css/dist/widgets/style-rtl.min.css
wordpress/wp-includes/css/dist/widgets/style.min.css
wordpress/wp-includes/css/dist/widgets/style-rtl.css
wordpress/wp-includes/css/dist/widgets/style.css
wordpress/wp-includes/css/dist/commands/
wordpress/wp-includes/css/dist/commands/style-rtl.min.css
wordpress/wp-includes/css/dist/commands/style.min.css
wordpress/wp-includes/css/dist/commands/style-rtl.css
wordpress/wp-includes/css/dist/commands/style.css
wordpress/wp-includes/css/dist/customize-widgets/
wordpress/wp-includes/css/dist/customize-widgets/style-rtl.min.css
wordpress/wp-includes/css/dist/customize-widgets/style.min.css
wordpress/wp-includes/css/dist/customize-widgets/style-rtl.css
wordpress/wp-includes/css/dist/customize-widgets/style.css
wordpress/wp-includes/css/dist/reusable-blocks/
wordpress/wp-includes/css/dist/reusable-blocks/style-rtl.min.css
wordpress/wp-includes/css/dist/reusable-blocks/style.min.css
wordpress/wp-includes/css/dist/reusable-blocks/style-rtl.css
wordpress/wp-includes/css/dist/reusable-blocks/style.css
wordpress/wp-includes/css/dist/editor/
wordpress/wp-includes/css/dist/editor/style-rtl.min.css
wordpress/wp-includes/css/dist/editor/style.min.css
wordpress/wp-includes/css/dist/editor/style-rtl.css
wordpress/wp-includes/css/dist/editor/style.css
wordpress/wp-includes/css/dist/nux/
wordpress/wp-includes/css/dist/nux/style-rtl.min.css
wordpress/wp-includes/css/dist/nux/style.min.css
wordpress/wp-includes/css/dist/nux/style-rtl.css
wordpress/wp-includes/css/dist/nux/style.css
wordpress/wp-includes/css/dist/list-reusable-blocks/
wordpress/wp-includes/css/dist/list-reusable-blocks/style-rtl.min.css
wordpress/wp-includes/css/dist/list-reusable-blocks/style.min.css
wordpress/wp-includes/css/dist/list-reusable-blocks/style-rtl.css
wordpress/wp-includes/css/dist/list-reusable-blocks/style.css
wordpress/wp-includes/css/dist/components/
wordpress/wp-includes/css/dist/components/style-rtl.min.css
wordpress/wp-includes/css/dist/components/style.min.css
wordpress/wp-includes/css/dist/components/style-rtl.css
wordpress/wp-includes/css/dist/components/style.css
wordpress/wp-includes/css/dist/block-editor/
wordpress/wp-includes/css/dist/block-editor/content.min.css
wordpress/wp-includes/css/dist/block-editor/style-rtl.min.css
wordpress/wp-includes/css/dist/block-editor/default-editor-styles-rtl.css
wordpress/wp-includes/css/dist/block-editor/default-editor-styles-rtl.min.css
wordpress/wp-includes/css/dist/block-editor/content.css
wordpress/wp-includes/css/dist/block-editor/content-rtl.min.css
wordpress/wp-includes/css/dist/block-editor/default-editor-styles.min.css
wordpress/wp-includes/css/dist/block-editor/default-editor-styles.css
wordpress/wp-includes/css/dist/block-editor/style.min.css
wordpress/wp-includes/css/dist/block-editor/content-rtl.css
wordpress/wp-includes/css/dist/block-editor/style-rtl.css
wordpress/wp-includes/css/dist/block-editor/style.css
wordpress/wp-includes/css/media-views-rtl.css
wordpress/wp-includes/css/customize-preview-rtl.min.css
wordpress/wp-includes/css/jquery-ui-dialog.css
wordpress/wp-includes/css/admin-bar.css
wordpress/wp-includes/css/media-views-rtl.min.css
wordpress/wp-includes/css/jquery-ui-dialog.min.css
wordpress/wp-includes/css/wp-pointer-rtl.min.css
wordpress/wp-includes/css/buttons-rtl.css
wordpress/wp-includes/css/buttons.css
wordpress/wp-includes/css/wp-embed-template-ie.css
wordpress/wp-includes/css/wp-embed-template-ie.min.css
wordpress/wp-includes/css/media-views.css
wordpress/wp-includes/css/jquery-ui-dialog-rtl.css
wordpress/wp-includes/css/dashicons.css
wordpress/wp-includes/css/editor-rtl.css
wordpress/wp-includes/css/wp-auth-check-rtl.min.css
wordpress/wp-includes/css/editor-rtl.min.css
wordpress/wp-includes/css/customize-preview.min.css
wordpress/wp-includes/css/buttons-rtl.min.css
wordpress/wp-includes/css/wp-pointer.min.css
wordpress/wp-includes/css/wp-auth-check.min.css
wordpress/wp-includes/css/wp-auth-check.css
wordpress/wp-includes/css/wp-auth-check-rtl.css
wordpress/wp-includes/css/wp-embed-template.css
wordpress/wp-includes/css/buttons.min.css
wordpress/wp-includes/css/dashicons.min.css
wordpress/wp-includes/theme-templates.php
wordpress/wp-includes/class-wp-http-curl.php
wordpress/wp-includes/class-wp-duotone.php
wordpress/wp-includes/class-smtp.php
wordpress/wp-includes/class-simplepie.php
wordpress/wp-includes/class-wpdb.php
wordpress/wp-includes/class-wp-role.php
wordpress/wp-includes/feed.php
wordpress/wp-includes/class-wp-simplepie-file.php
wordpress/wp-includes/class-wp-theme.php
wordpress/wp-includes/class-wp-theme-json.php
wordpress/wp-includes/class-walker-page.php
wordpress/wp-includes/class-wp-paused-extensions-storage.php
wordpress/wp-includes/class-wp-block-editor-context.php
wordpress/wp-includes/ms-default-filters.php
wordpress/wp-includes/class-wp-theme-json-data.php
wordpress/wp-includes/class-wp-http-requests-hooks.php
wordpress/wp-includes/ms-files.php
wordpress/wp-includes/class-wp-block-pattern-categories-registry.php
wordpress/wp-includes/class-snoopy.php
wordpress/wp-includes/class-wp-http.php
wordpress/wp-includes/class-wp-post.php
wordpress/wp-includes/class-http.php
wordpress/wp-includes/media.php
wordpress/wp-includes/script-loader.php
wordpress/wp-includes/class-pop3.php
wordpress/wp-includes/category.php
wordpress/wp-includes/class-wp-recovery-mode-link-service.php
wordpress/wp-includes/class-wp-customize-widgets.php
wordpress/wp-includes/block-i18n.json
wordpress/wp-includes/block-template-utils.php
wordpress/wp-includes/Text/
wordpress/wp-includes/Text/Diff/
wordpress/wp-includes/Text/Diff/Renderer/
wordpress/wp-includes/Text/Diff/Renderer/inline.php
wordpress/wp-includes/Text/Diff/Renderer.php
wordpress/wp-includes/Text/Diff/Engine/
wordpress/wp-includes/Text/Diff/Engine/native.php
wordpress/wp-includes/Text/Diff/Engine/string.php
wordpress/wp-includes/Text/Diff/Engine/xdiff.php
wordpress/wp-includes/Text/Diff/Engine/shell.php
wordpress/wp-includes/Text/Diff.php
wordpress/wp-includes/ms-deprecated.php
wordpress/wp-includes/class-phpass.php
wordpress/wp-includes/class-avif-info.php
wordpress/wp-includes/class-wp-editor.php
wordpress/wp-includes/class-wp-navigation-fallback.php
wordpress/wp-includes/meta.php
wordpress/wp-includes/assets/
wordpress/wp-includes/assets/script-loader-packages.php
wordpress/wp-includes/assets/script-loader-react-refresh-entry.php
wordpress/wp-includes/assets/script-loader-packages.min.php
wordpress/wp-includes/assets/script-loader-react-refresh-runtime.min.php
wordpress/wp-includes/assets/script-loader-react-refresh-entry.min.php
wordpress/wp-includes/assets/script-loader-react-refresh-runtime.php
wordpress/wp-includes/taxonomy.php
wordpress/wp-includes/rest-api.php
wordpress/wp-includes/class-wp-widget-factory.php
wordpress/wp-includes/nav-menu.php
wordpress/wp-includes/class-wp-http-response.php
wordpress/wp-includes/class-wp-tax-query.php
wordpress/wp-includes/class-wp-http-ixr-client.php
wordpress/wp-includes/shortcodes.php
wordpress/wp-includes/user.php
wordpress/wp-includes/class-wp-image-editor-gd.php
wordpress/wp-includes/class-walker-nav-menu.php
wordpress/wp-includes/class-wp-simplepie-sanitize-kses.php
wordpress/wp-includes/ms-functions.php
wordpress/wp-includes/class-wp-customize-section.php
wordpress/wp-includes/class-wp-block-bindings-registry.php
wordpress/wp-includes/class-wp-matchesmapregex.php
wordpress/wp-includes/class-wp-recovery-mode-cookie-service.php
wordpress/wp-includes/revision.php
wordpress/wp-includes/functions.wp-styles.php
wordpress/wp-includes/class-wp-customize-manager.php
wordpress/wp-includes/class-walker-category.php
wordpress/wp-includes/class-requests.php
wordpress/wp-includes/query.php
wordpress/wp-includes/ms-blogs.php
wordpress/wp-includes/class-wp-network.php
wordpress/wp-includes/cache-compat.php
wordpress/wp-includes/html-api/
wordpress/wp-includes/html-api/class-wp-html-text-replacement.php
wordpress/wp-includes/html-api/class-wp-html-processor-state.php
wordpress/wp-includes/html-api/class-wp-html-token.php
wordpress/wp-includes/html-api/class-wp-html-attribute-token.php
wordpress/wp-includes/html-api/class-wp-html-span.php
wordpress/wp-includes/html-api/class-wp-html-unsupported-exception.php
wordpress/wp-includes/html-api/class-wp-html-open-elements.php
wordpress/wp-includes/html-api/class-wp-html-processor.php
wordpress/wp-includes/html-api/class-wp-html-active-formatting-elements.php
wordpress/wp-includes/html-api/class-wp-html-tag-processor.php
wordpress/wp-includes/cron.php
wordpress/wp-includes/blocks.php
wordpress/wp-includes/class-wp-block-bindings-source.php
wordpress/wp-includes/ms-site.php
wordpress/wp-includes/feed-rss.php
wordpress/wp-includes/rewrite.php
wordpress/wp-includes/certificates/
wordpress/wp-includes/certificates/ca-bundle.crt
wordpress/wp-includes/class-wp-roles.php
wordpress/wp-includes/embed-template.php
wordpress/wp-includes/rss.php
wordpress/wp-includes/canonical.php
wordpress/wp-includes/class-wp-site.php
wordpress/wp-includes/class-wp-comment.php
wordpress/wp-includes/kses.php
wordpress/wp-includes/ms-settings.php
wordpress/wp-includes/class-wp-term-query.php
wordpress/wp-includes/category-template.php
wordpress/wp-includes/ID3/
wordpress/wp-includes/ID3/module.tag.lyrics3.php
wordpress/wp-includes/ID3/module.audio.dts.php
wordpress/wp-includes/ID3/module.tag.id3v1.php
wordpress/wp-includes/ID3/module.audio.flac.php
wordpress/wp-includes/ID3/module.audio.mp3.php
wordpress/wp-includes/ID3/module.audio-video.matroska.php
wordpress/wp-includes/ID3/module.audio-video.quicktime.php
wordpress/wp-includes/ID3/module.tag.apetag.php
wordpress/wp-includes/ID3/getid3.lib.php
wordpress/wp-includes/ID3/module.audio.ac3.php
wordpress/wp-includes/ID3/getid3.php
wordpress/wp-includes/ID3/license.txt
wordpress/wp-includes/ID3/module.audio-video.flv.php
wordpress/wp-includes/ID3/readme.txt
wordpress/wp-includes/ID3/module.tag.id3v2.php
wordpress/wp-includes/ID3/module.audio.ogg.php
wordpress/wp-includes/ID3/module.audio-video.riff.php
wordpress/wp-includes/ID3/module.audio-video.asf.php
wordpress/wp-includes/class-walker-page-dropdown.php
wordpress/wp-includes/class-wp-block-supports.php
wordpress/wp-includes/class-wp-customize-panel.php
wordpress/wp-includes/class-wp-locale.php
wordpress/wp-includes/class-wp-user-meta-session-tokens.php
wordpress/wp-includes/class-wp-block-parser-block.php
wordpress/wp-includes/class-wp-http-requests-response.php
wordpress/wp-includes/template-loader.php
wordpress/wp-includes/interactivity-api/
wordpress/wp-includes/interactivity-api/class-wp-interactivity-api.php
wordpress/wp-includes/interactivity-api/class-wp-interactivity-api-directives-processor.php
wordpress/wp-includes/interactivity-api/interactivity-api.php
wordpress/wp-includes/class-wp-site-query.php
wordpress/wp-includes/style-engine.php
wordpress/wp-includes/rss-functions.php
wordpress/wp-includes/vars.php
wordpress/wp-includes/feed-rss2.php
wordpress/wp-includes/sitemaps.php
wordpress/wp-includes/general-template.php
wordpress/wp-includes/class-wp-block-template.php
wordpress/wp-includes/script-modules.php
wordpress/wp-includes/default-constants.php
wordpress/wp-includes/blocks/
wordpress/wp-includes/blocks/latest-comments.php
wordpress/wp-includes/blocks/post-content.php
wordpress/wp-includes/blocks/navigation-link.php
wordpress/wp-includes/blocks/navigation/
wordpress/wp-includes/blocks/navigation/editor.min.css
wordpress/wp-includes/blocks/navigation/style-rtl.min.css
wordpress/wp-includes/blocks/navigation/block.json
wordpress/wp-includes/blocks/navigation/editor.css
wordpress/wp-includes/blocks/navigation/view-modal.min.asset.php
wordpress/wp-includes/blocks/navigation/view.min.asset.php
wordpress/wp-includes/blocks/navigation/view.js
wordpress/wp-includes/blocks/navigation/style.min.css
wordpress/wp-includes/blocks/navigation/editor-rtl.css
wordpress/wp-includes/blocks/navigation/view.asset.php
wordpress/wp-includes/blocks/navigation/view-modal.asset.php
wordpress/wp-includes/blocks/navigation/editor-rtl.min.css
wordpress/wp-includes/blocks/navigation/view.min.js
wordpress/wp-includes/blocks/navigation/style-rtl.css
wordpress/wp-includes/blocks/navigation/style.css
wordpress/wp-includes/blocks/post-content/
wordpress/wp-includes/blocks/post-content/editor.min.css
wordpress/wp-includes/blocks/post-content/block.json
wordpress/wp-includes/blocks/post-content/editor.css
wordpress/wp-includes/blocks/post-content/editor-rtl.css
wordpress/wp-includes/blocks/post-content/editor-rtl.min.css
wordpress/wp-includes/blocks/comments/
wordpress/wp-includes/blocks/comments/editor.min.css
wordpress/wp-includes/blocks/comments/style-rtl.min.css
wordpress/wp-includes/blocks/comments/block.json
wordpress/wp-includes/blocks/comments/editor.css
wordpress/wp-includes/blocks/comments/style.min.css
wordpress/wp-includes/blocks/comments/editor-rtl.css
wordpress/wp-includes/blocks/comments/editor-rtl.min.css
wordpress/wp-includes/blocks/comments/style-rtl.css
wordpress/wp-includes/blocks/comments/style.css
wordpress/wp-includes/blocks/calendar.php
wordpress/wp-includes/blocks/site-tagline.php
wordpress/wp-includes/blocks/loginout.php
wordpress/wp-includes/blocks/query-pagination-numbers/
wordpress/wp-includes/blocks/query-pagination-numbers/editor.min.css
wordpress/wp-includes/blocks/query-pagination-numbers/block.json
wordpress/wp-includes/blocks/query-pagination-numbers/editor.css
wordpress/wp-includes/blocks/query-pagination-numbers/editor-rtl.css
wordpress/wp-includes/blocks/query-pagination-numbers/editor-rtl.min.css
wordpress/wp-includes/blocks/gallery/
wordpress/wp-includes/blocks/gallery/editor.min.css
wordpress/wp-includes/blocks/gallery/style-rtl.min.css
wordpress/wp-includes/blocks/gallery/theme.css
wordpress/wp-includes/blocks/gallery/block.json
wordpress/wp-includes/blocks/gallery/theme-rtl.min.css
wordpress/wp-includes/blocks/gallery/editor.css
wordpress/wp-includes/blocks/gallery/theme.min.css
wordpress/wp-includes/blocks/gallery/style.min.css
wordpress/wp-includes/blocks/gallery/theme-rtl.css
wordpress/wp-includes/blocks/gallery/editor-rtl.css
wordpress/wp-includes/blocks/gallery/editor-rtl.min.css
wordpress/wp-includes/blocks/gallery/style-rtl.css
wordpress/wp-includes/blocks/gallery/style.css
wordpress/wp-includes/blocks/shortcode/
wordpress/wp-includes/blocks/shortcode/editor.min.css
wordpress/wp-includes/blocks/shortcode/block.json
wordpress/wp-includes/blocks/shortcode/editor.css
wordpress/wp-includes/blocks/shortcode/editor-rtl.css
wordpress/wp-includes/blocks/shortcode/editor-rtl.min.css
wordpress/wp-includes/blocks/calendar/
wordpress/wp-includes/blocks/calendar/style-rtl.min.css
wordpress/wp-includes/blocks/calendar/block.json
wordpress/wp-includes/blocks/calendar/style.min.css
wordpress/wp-includes/blocks/calendar/style-rtl.css
wordpress/wp-includes/blocks/calendar/style.css
wordpress/wp-includes/blocks/comment-content/
wordpress/wp-includes/blocks/comment-content/style-rtl.min.css
wordpress/wp-includes/blocks/comment-content/block.json
wordpress/wp-includes/blocks/comment-content/style.min.css
wordpress/wp-includes/blocks/comment-content/style-rtl.css
wordpress/wp-includes/blocks/comment-content/style.css
wordpress/wp-includes/blocks/details/
wordpress/wp-includes/blocks/details/editor.min.css
wordpress/wp-includes/blocks/details/style-rtl.min.css
wordpress/wp-includes/blocks/details/block.json
wordpress/wp-includes/blocks/details/editor.css
wordpress/wp-includes/blocks/details/style.min.css
wordpress/wp-includes/blocks/details/editor-rtl.css
wordpress/wp-includes/blocks/details/editor-rtl.min.css
wordpress/wp-includes/blocks/details/style-rtl.css
wordpress/wp-includes/blocks/details/style.css
wordpress/wp-includes/blocks/template-part/
wordpress/wp-includes/blocks/template-part/editor.min.css
wordpress/wp-includes/blocks/template-part/theme.css
wordpress/wp-includes/blocks/template-part/block.json
wordpress/wp-includes/blocks/template-part/theme-rtl.min.css
wordpress/wp-includes/blocks/template-part/editor.css
wordpress/wp-includes/blocks/template-part/theme.min.css
wordpress/wp-includes/blocks/template-part/theme-rtl.css
wordpress/wp-includes/blocks/template-part/editor-rtl.css
wordpress/wp-includes/blocks/template-part/editor-rtl.min.css
wordpress/wp-includes/blocks/comments-pagination-next.php
wordpress/wp-includes/blocks/post-title.php
wordpress/wp-includes/blocks/post-navigation-link/
wordpress/wp-includes/blocks/post-navigation-link/style-rtl.min.css
wordpress/wp-includes/blocks/post-navigation-link/block.json
wordpress/wp-includes/blocks/post-navigation-link/style.min.css
wordpress/wp-includes/blocks/post-navigation-link/style-rtl.css
wordpress/wp-includes/blocks/post-navigation-link/style.css
wordpress/wp-includes/blocks/require-dynamic-blocks.php
wordpress/wp-includes/blocks/columns/
wordpress/wp-includes/blocks/columns/editor.min.css
wordpress/wp-includes/blocks/columns/style-rtl.min.css
wordpress/wp-includes/blocks/columns/block.json
wordpress/wp-includes/blocks/columns/editor.css
wordpress/wp-includes/blocks/columns/style.min.css
wordpress/wp-includes/blocks/columns/editor-rtl.css
wordpress/wp-includes/blocks/columns/editor-rtl.min.css
wordpress/wp-includes/blocks/columns/style-rtl.css
wordpress/wp-includes/blocks/columns/style.css
wordpress/wp-includes/blocks/shortcode.php
wordpress/wp-includes/blocks/comment-reply-link/
wordpress/wp-includes/blocks/comment-reply-link/block.json
wordpress/wp-includes/blocks/post-author-name/
wordpress/wp-includes/blocks/post-author-name/block.json
wordpress/wp-includes/blocks/site-title.php
wordpress/wp-includes/blocks/term-description/
wordpress/wp-includes/blocks/term-description/style-rtl.min.css
wordpress/wp-includes/blocks/term-description/block.json
wordpress/wp-includes/blocks/term-description/style.min.css
wordpress/wp-includes/blocks/term-description/style-rtl.css
wordpress/wp-includes/blocks/term-description/style.css
wordpress/wp-includes/blocks/code/
wordpress/wp-includes/blocks/code/editor.min.css
wordpress/wp-includes/blocks/code/style-rtl.min.css
wordpress/wp-includes/blocks/code/theme.css
wordpress/wp-includes/blocks/code/block.json
wordpress/wp-includes/blocks/code/theme-rtl.min.css
wordpress/wp-includes/blocks/code/editor.css
wordpress/wp-includes/blocks/code/theme.min.css
wordpress/wp-includes/blocks/code/style.min.css
wordpress/wp-includes/blocks/code/theme-rtl.css
wordpress/wp-includes/blocks/code/editor-rtl.css
wordpress/wp-includes/blocks/code/editor-rtl.min.css
wordpress/wp-includes/blocks/code/style-rtl.css
wordpress/wp-includes/blocks/code/style.css
wordpress/wp-includes/blocks/comment-date.php
wordpress/wp-includes/blocks/footnotes/
wordpress/wp-includes/blocks/footnotes/style-rtl.min.css
wordpress/wp-includes/blocks/footnotes/block.json
wordpress/wp-includes/blocks/footnotes/style.min.css
wordpress/wp-includes/blocks/footnotes/style-rtl.css
wordpress/wp-includes/blocks/footnotes/style.css
wordpress/wp-includes/blocks/comments-pagination.php
wordpress/wp-includes/blocks/loginout/
wordpress/wp-includes/blocks/loginout/block.json
wordpress/wp-includes/blocks/avatar/
wordpress/wp-includes/blocks/avatar/editor.min.css
wordpress/wp-includes/blocks/avatar/style-rtl.min.css
wordpress/wp-includes/blocks/avatar/block.json
wordpress/wp-includes/blocks/avatar/editor.css
wordpress/wp-includes/blocks/avatar/style.min.css
wordpress/wp-includes/blocks/avatar/editor-rtl.css
wordpress/wp-includes/blocks/avatar/editor-rtl.min.css
wordpress/wp-includes/blocks/avatar/style-rtl.css
wordpress/wp-includes/blocks/avatar/style.css
wordpress/wp-includes/blocks/button/
wordpress/wp-includes/blocks/button/editor.min.css
wordpress/wp-includes/blocks/button/style-rtl.min.css
wordpress/wp-includes/blocks/button/block.json
wordpress/wp-includes/blocks/button/editor.css
wordpress/wp-includes/blocks/button/style.min.css
wordpress/wp-includes/blocks/button/editor-rtl.css
wordpress/wp-includes/blocks/button/editor-rtl.min.css
wordpress/wp-includes/blocks/button/style-rtl.css
wordpress/wp-includes/blocks/button/style.css
wordpress/wp-includes/blocks/comments-title/
wordpress/wp-includes/blocks/comments-title/editor.min.css
wordpress/wp-includes/blocks/comments-title/block.json
wordpress/wp-includes/blocks/comments-title/editor.css
wordpress/wp-includes/blocks/comments-title/editor-rtl.css
wordpress/wp-includes/blocks/comments-title/editor-rtl.min.css
wordpress/wp-includes/blocks/image.php
wordpress/wp-includes/blocks/widget-group.php
wordpress/wp-includes/blocks/comments-pagination-numbers.php
wordpress/wp-includes/blocks/post-author.php
wordpress/wp-includes/blocks/post-author-biography/
wordpress/wp-includes/blocks/post-author-biography/block.json
wordpress/wp-includes/blocks/categories/
wordpress/wp-includes/blocks/categories/editor.min.css
wordpress/wp-includes/blocks/categories/style-rtl.min.css
wordpress/wp-includes/blocks/categories/block.json
wordpress/wp-includes/blocks/categories/editor.css
wordpress/wp-includes/blocks/categories/style.min.css
wordpress/wp-includes/blocks/categories/editor-rtl.css
wordpress/wp-includes/blocks/categories/editor-rtl.min.css
wordpress/wp-includes/blocks/categories/style-rtl.css
wordpress/wp-includes/blocks/categories/style.css
wordpress/wp-includes/blocks/social-link/
wordpress/wp-includes/blocks/social-link/editor.min.css
wordpress/wp-includes/blocks/social-link/block.json
wordpress/wp-includes/blocks/social-link/editor.css
wordpress/wp-includes/blocks/social-link/editor-rtl.css
wordpress/wp-includes/blocks/social-link/editor-rtl.min.css
wordpress/wp-includes/blocks/separator/
wordpress/wp-includes/blocks/separator/editor.min.css
wordpress/wp-includes/blocks/separator/style-rtl.min.css
wordpress/wp-includes/blocks/separator/theme.css
wordpress/wp-includes/blocks/separator/block.json
wordpress/wp-includes/blocks/separator/theme-rtl.min.css
wordpress/wp-includes/blocks/separator/editor.css
wordpress/wp-includes/blocks/separator/theme.min.css
wordpress/wp-includes/blocks/separator/style.min.css
wordpress/wp-includes/blocks/separator/theme-rtl.css
wordpress/wp-includes/blocks/separator/editor-rtl.css
wordpress/wp-includes/blocks/separator/editor-rtl.min.css
wordpress/wp-includes/blocks/separator/style-rtl.css
wordpress/wp-includes/blocks/separator/style.css
wordpress/wp-includes/blocks/html/
wordpress/wp-includes/blocks/html/editor.min.css
wordpress/wp-includes/blocks/html/block.json
wordpress/wp-includes/blocks/html/editor.css
wordpress/wp-includes/blocks/html/editor-rtl.css
wordpress/wp-includes/blocks/html/editor-rtl.min.css
wordpress/wp-includes/blocks/page-list/
wordpress/wp-includes/blocks/page-list/editor.min.css
wordpress/wp-includes/blocks/page-list/style-rtl.min.css
wordpress/wp-includes/blocks/page-list/block.json
wordpress/wp-includes/blocks/page-list/editor.css
wordpress/wp-includes/blocks/page-list/style.min.css
wordpress/wp-includes/blocks/page-list/editor-rtl.css
wordpress/wp-includes/blocks/page-list/editor-rtl.min.css
wordpress/wp-includes/blocks/page-list/style-rtl.css
wordpress/wp-includes/blocks/page-list/style.css
wordpress/wp-includes/blocks/home-link/
wordpress/wp-includes/blocks/home-link/block.json
wordpress/wp-includes/blocks/pattern/
wordpress/wp-includes/blocks/pattern/block.json
wordpress/wp-includes/blocks/social-link.php
wordpress/wp-includes/blocks/require-static-blocks.php
wordpress/wp-includes/blocks/query-pagination-previous/
wordpress/wp-includes/blocks/query-pagination-previous/block.json
wordpress/wp-includes/blocks/widget-group/
wordpress/wp-includes/blocks/widget-group/block.json
wordpress/wp-includes/blocks/freeform/
wordpress/wp-includes/blocks/freeform/editor.min.css
wordpress/wp-includes/blocks/freeform/block.json
wordpress/wp-includes/blocks/freeform/editor.css
wordpress/wp-includes/blocks/freeform/editor-rtl.css
wordpress/wp-includes/blocks/freeform/editor-rtl.min.css
wordpress/wp-includes/blocks/audio/
wordpress/wp-includes/blocks/audio/editor.min.css
wordpress/wp-includes/blocks/audio/style-rtl.min.css
wordpress/wp-includes/blocks/audio/theme.css
wordpress/wp-includes/blocks/audio/block.json
wordpress/wp-includes/blocks/audio/theme-rtl.min.css
wordpress/wp-includes/blocks/audio/editor.css
wordpress/wp-includes/blocks/audio/theme.min.css
wordpress/wp-includes/blocks/audio/style.min.css
wordpress/wp-includes/blocks/audio/theme-rtl.css
wordpress/wp-includes/blocks/audio/editor-rtl.css
wordpress/wp-includes/blocks/audio/editor-rtl.min.css
wordpress/wp-includes/blocks/audio/style-rtl.css
wordpress/wp-includes/blocks/audio/style.css
wordpress/wp-includes/blocks/query/
wordpress/wp-includes/blocks/query/editor.min.css
wordpress/wp-includes/blocks/query/block.json
wordpress/wp-includes/blocks/query/editor.css
wordpress/wp-includes/blocks/query/view.min.asset.php
wordpress/wp-includes/blocks/query/view.js
wordpress/wp-includes/blocks/query/editor-rtl.css
wordpress/wp-includes/blocks/query/view.asset.php
wordpress/wp-includes/blocks/query/editor-rtl.min.css
wordpress/wp-includes/blocks/query/view.min.js
wordpress/wp-includes/blocks/blocks-json.php
wordpress/wp-includes/blocks/home-link.php
wordpress/wp-includes/blocks/post-excerpt.php
wordpress/wp-includes/blocks/post-author-name.php
wordpress/wp-includes/blocks/site-tagline/
wordpress/wp-includes/blocks/site-tagline/editor.min.css
wordpress/wp-includes/blocks/site-tagline/block.json
wordpress/wp-includes/blocks/site-tagline/editor.css
wordpress/wp-includes/blocks/site-tagline/editor-rtl.css
wordpress/wp-includes/blocks/site-tagline/editor-rtl.min.css
wordpress/wp-includes/blocks/verse/
wordpress/wp-includes/blocks/verse/style-rtl.min.css
wordpress/wp-includes/blocks/verse/block.json
wordpress/wp-includes/blocks/verse/style.min.css
wordpress/wp-includes/blocks/verse/style-rtl.css
wordpress/wp-includes/blocks/verse/style.css
wordpress/wp-includes/blocks/site-logo/
wordpress/wp-includes/blocks/site-logo/editor.min.css
wordpress/wp-includes/blocks/site-logo/style-rtl.min.css
wordpress/wp-includes/blocks/site-logo/block.json
wordpress/wp-includes/blocks/site-logo/editor.css
wordpress/wp-includes/blocks/site-logo/style.min.css
wordpress/wp-includes/blocks/site-logo/editor-rtl.css
wordpress/wp-includes/blocks/site-logo/editor-rtl.min.css
wordpress/wp-includes/blocks/site-logo/style-rtl.css
wordpress/wp-includes/blocks/site-logo/style.css
wordpress/wp-includes/blocks/query.php
wordpress/wp-includes/blocks/site-title/
wordpress/wp-includes/blocks/site-title/editor.min.css
wordpress/wp-includes/blocks/site-title/style-rtl.min.css
wordpress/wp-includes/blocks/site-title/block.json
wordpress/wp-includes/blocks/site-title/editor.css
wordpress/wp-includes/blocks/site-title/style.min.css
wordpress/wp-includes/blocks/site-title/editor-rtl.css
wordpress/wp-includes/blocks/site-title/editor-rtl.min.css
wordpress/wp-includes/blocks/site-title/style-rtl.css
wordpress/wp-includes/blocks/site-title/style.css
wordpress/wp-includes/blocks/page-list.php
wordpress/wp-includes/blocks/search.php
wordpress/wp-includes/blocks/query-no-results/
wordpress/wp-includes/blocks/query-no-results/block.json
wordpress/wp-includes/blocks/heading/
wordpress/wp-includes/blocks/heading/style-rtl.min.css
wordpress/wp-includes/blocks/heading/block.json
wordpress/wp-includes/blocks/heading/style.min.css
wordpress/wp-includes/blocks/heading/style-rtl.css
wordpress/wp-includes/blocks/heading/style.css
wordpress/wp-includes/blocks/block.php
wordpress/wp-includes/blocks/pullquote/
wordpress/wp-includes/blocks/pullquote/editor.min.css
wordpress/wp-includes/blocks/pullquote/style-rtl.min.css
wordpress/wp-includes/blocks/pullquote/theme.css
wordpress/wp-includes/blocks/pullquote/block.json
wordpress/wp-includes/blocks/pullquote/theme-rtl.min.css
wordpress/wp-includes/blocks/pullquote/editor.css
wordpress/wp-includes/blocks/pullquote/theme.min.css
wordpress/wp-includes/blocks/pullquote/style.min.css
wordpress/wp-includes/blocks/pullquote/theme-rtl.css
wordpress/wp-includes/blocks/pullquote/editor-rtl.css
wordpress/wp-includes/blocks/pullquote/editor-rtl.min.css
wordpress/wp-includes/blocks/pullquote/style-rtl.css
wordpress/wp-includes/blocks/pullquote/style.css
wordpress/wp-includes/blocks/buttons/
wordpress/wp-includes/blocks/buttons/editor.min.css
wordpress/wp-includes/blocks/buttons/style-rtl.min.css
wordpress/wp-includes/blocks/buttons/block.json
wordpress/wp-includes/blocks/buttons/editor.css
wordpress/wp-includes/blocks/buttons/style.min.css
wordpress/wp-includes/blocks/buttons/editor-rtl.css
wordpress/wp-includes/blocks/buttons/editor-rtl.min.css
wordpress/wp-includes/blocks/buttons/style-rtl.css
wordpress/wp-includes/blocks/buttons/style.css
wordpress/wp-includes/blocks/post-featured-image.php
wordpress/wp-includes/blocks/post-terms.php
wordpress/wp-includes/blocks/rss.php
wordpress/wp-includes/blocks/nextpage/
wordpress/wp-includes/blocks/nextpage/editor.min.css
wordpress/wp-includes/blocks/nextpage/block.json
wordpress/wp-includes/blocks/nextpage/editor.css
wordpress/wp-includes/blocks/nextpage/editor-rtl.css
wordpress/wp-includes/blocks/nextpage/editor-rtl.min.css
wordpress/wp-includes/blocks/paragraph/
wordpress/wp-includes/blocks/paragraph/editor.min.css
wordpress/wp-includes/blocks/paragraph/style-rtl.min.css
wordpress/wp-includes/blocks/paragraph/block.json
wordpress/wp-includes/blocks/paragraph/editor.css
wordpress/wp-includes/blocks/paragraph/style.min.css
wordpress/wp-includes/blocks/paragraph/editor-rtl.css
wordpress/wp-includes/blocks/paragraph/editor-rtl.min.css
wordpress/wp-includes/blocks/paragraph/style-rtl.css
wordpress/wp-includes/blocks/paragraph/style.css
wordpress/wp-includes/blocks/footnotes.php
wordpress/wp-includes/blocks/navigation-submenu/
wordpress/wp-includes/blocks/navigation-submenu/editor.min.css
wordpress/wp-includes/blocks/navigation-submenu/block.json
wordpress/wp-includes/blocks/navigation-submenu/editor.css
wordpress/wp-includes/blocks/navigation-submenu/editor-rtl.css
wordpress/wp-includes/blocks/navigation-submenu/editor-rtl.min.css
wordpress/wp-includes/blocks/archives/
wordpress/wp-includes/blocks/archives/editor.min.css
wordpress/wp-includes/blocks/archives/style-rtl.min.css
wordpress/wp-includes/blocks/archives/block.json
wordpress/wp-includes/blocks/archives/editor.css
wordpress/wp-includes/blocks/archives/style.min.css
wordpress/wp-includes/blocks/archives/editor-rtl.css
wordpress/wp-includes/blocks/archives/editor-rtl.min.css
wordpress/wp-includes/blocks/archives/style-rtl.css
wordpress/wp-includes/blocks/archives/style.css
wordpress/wp-includes/blocks/post-template/
wordpress/wp-includes/blocks/post-template/editor.min.css
wordpress/wp-includes/blocks/post-template/style-rtl.min.css
wordpress/wp-includes/blocks/post-template/block.json
wordpress/wp-includes/blocks/post-template/editor.css
wordpress/wp-includes/blocks/post-template/style.min.css
wordpress/wp-includes/blocks/post-template/editor-rtl.css
wordpress/wp-includes/blocks/post-template/editor-rtl.min.css
wordpress/wp-includes/blocks/post-template/style-rtl.css
wordpress/wp-includes/blocks/post-template/style.css
wordpress/wp-includes/blocks/query-pagination/
wordpress/wp-includes/blocks/query-pagination/editor.min.css
wordpress/wp-includes/blocks/query-pagination/style-rtl.min.css
wordpress/wp-includes/blocks/query-pagination/block.json
wordpress/wp-includes/blocks/query-pagination/editor.css
wordpress/wp-includes/blocks/query-pagination/style.min.css
wordpress/wp-includes/blocks/query-pagination/editor-rtl.css
wordpress/wp-includes/blocks/query-pagination/editor-rtl.min.css
wordpress/wp-includes/blocks/query-pagination/style-rtl.css
wordpress/wp-includes/blocks/query-pagination/style.css
wordpress/wp-includes/blocks/post-comments-form.php
wordpress/wp-includes/blocks/embed/
wordpress/wp-includes/blocks/embed/editor.min.css
wordpress/wp-includes/blocks/embed/style-rtl.min.css
wordpress/wp-includes/blocks/embed/theme.css
wordpress/wp-includes/blocks/embed/block.json
wordpress/wp-includes/blocks/embed/theme-rtl.min.css
wordpress/wp-includes/blocks/embed/editor.css
wordpress/wp-includes/blocks/embed/theme.min.css
wordpress/wp-includes/blocks/embed/style.min.css
wordpress/wp-includes/blocks/embed/theme-rtl.css
wordpress/wp-includes/blocks/embed/editor-rtl.css
wordpress/wp-includes/blocks/embed/editor-rtl.min.css
wordpress/wp-includes/blocks/embed/style-rtl.css
wordpress/wp-includes/blocks/embed/style.css
wordpress/wp-includes/blocks/archives.php
wordpress/wp-includes/blocks/navigation-submenu.php
wordpress/wp-includes/blocks/block/
wordpress/wp-includes/blocks/block/editor.min.css
wordpress/wp-includes/blocks/block/block.json
wordpress/wp-includes/blocks/block/editor.css
wordpress/wp-includes/blocks/block/editor-rtl.css
wordpress/wp-includes/blocks/block/editor-rtl.min.css
wordpress/wp-includes/blocks/post-featured-image/
wordpress/wp-includes/blocks/post-featured-image/editor.min.css
wordpress/wp-includes/blocks/post-featured-image/style-rtl.min.css
wordpress/wp-includes/blocks/post-featured-image/block.json
wordpress/wp-includes/blocks/post-featured-image/editor.css
wordpress/wp-includes/blocks/post-featured-image/style.min.css
wordpress/wp-includes/blocks/post-featured-image/editor-rtl.css
wordpress/wp-includes/blocks/post-featured-image/editor-rtl.min.css
wordpress/wp-includes/blocks/post-featured-image/style-rtl.css
wordpress/wp-includes/blocks/post-featured-image/style.css
wordpress/wp-includes/blocks/comment-content.php
wordpress/wp-includes/blocks/media-text/
wordpress/wp-includes/blocks/media-text/editor.min.css
wordpress/wp-includes/blocks/media-text/style-rtl.min.css
wordpress/wp-includes/blocks/media-text/block.json
wordpress/wp-includes/blocks/media-text/editor.css
wordpress/wp-includes/blocks/media-text/style.min.css
wordpress/wp-includes/blocks/media-text/editor-rtl.css
wordpress/wp-includes/blocks/media-text/editor-rtl.min.css
wordpress/wp-includes/blocks/media-text/style-rtl.css
wordpress/wp-includes/blocks/media-text/style.css
wordpress/wp-includes/blocks/page-list-item/
wordpress/wp-includes/blocks/page-list-item/block.json
wordpress/wp-includes/blocks/column/
wordpress/wp-includes/blocks/column/block.json
wordpress/wp-includes/blocks/categories.php
wordpress/wp-includes/blocks/comment-edit-link/
wordpress/wp-includes/blocks/comment-edit-link/block.json
wordpress/wp-includes/blocks/rss/
wordpress/wp-includes/blocks/rss/editor.min.css
wordpress/wp-includes/blocks/rss/style-rtl.min.css
wordpress/wp-includes/blocks/rss/block.json
wordpress/wp-includes/blocks/rss/editor.css
wordpress/wp-includes/blocks/rss/style.min.css
wordpress/wp-includes/blocks/rss/editor-rtl.css
wordpress/wp-includes/blocks/rss/editor-rtl.min.css
wordpress/wp-includes/blocks/rss/style-rtl.css
wordpress/wp-includes/blocks/rss/style.css
wordpress/wp-includes/blocks/comments-pagination/
wordpress/wp-includes/blocks/comments-pagination/editor.min.css
wordpress/wp-includes/blocks/comments-pagination/style-rtl.min.css
wordpress/wp-includes/blocks/comments-pagination/block.json
wordpress/wp-includes/blocks/comments-pagination/editor.css
wordpress/wp-includes/blocks/comments-pagination/style.min.css
wordpress/wp-includes/blocks/comments-pagination/editor-rtl.css
wordpress/wp-includes/blocks/comments-pagination/editor-rtl.min.css
wordpress/wp-includes/blocks/comments-pagination/style-rtl.css
wordpress/wp-includes/blocks/comments-pagination/style.css
wordpress/wp-includes/blocks/comment-date/
wordpress/wp-includes/blocks/comment-date/block.json
wordpress/wp-includes/blocks/query-title/
wordpress/wp-includes/blocks/query-title/style-rtl.min.css
wordpress/wp-includes/blocks/query-title/block.json
wordpress/wp-includes/blocks/query-title/style.min.css
wordpress/wp-includes/blocks/query-title/style-rtl.css
wordpress/wp-includes/blocks/query-title/style.css
wordpress/wp-includes/blocks/comments-pagination-previous.php
wordpress/wp-includes/blocks/post-navigation-link.php
wordpress/wp-includes/blocks/quote/
wordpress/wp-includes/blocks/quote/style-rtl.min.css
wordpress/wp-includes/blocks/quote/theme.css
wordpress/wp-includes/blocks/quote/block.json
wordpress/wp-includes/blocks/quote/theme-rtl.min.css
wordpress/wp-includes/blocks/quote/theme.min.css
wordpress/wp-includes/blocks/quote/style.min.css
wordpress/wp-includes/blocks/quote/theme-rtl.css
wordpress/wp-includes/blocks/quote/style-rtl.css
wordpress/wp-includes/blocks/quote/style.css
wordpress/wp-includes/blocks/more/
wordpress/wp-includes/blocks/more/editor.min.css
wordpress/wp-includes/blocks/more/block.json
wordpress/wp-includes/blocks/more/editor.css
wordpress/wp-includes/blocks/more/editor-rtl.css
wordpress/wp-includes/blocks/more/editor-rtl.min.css
wordpress/wp-includes/blocks/read-more/
wordpress/wp-includes/blocks/read-more/style-rtl.min.css
wordpress/wp-includes/blocks/read-more/block.json
wordpress/wp-includes/blocks/read-more/style.min.css
wordpress/wp-includes/blocks/read-more/style-rtl.css
wordpress/wp-includes/blocks/read-more/style.css
wordpress/wp-includes/blocks/query-no-results.php
wordpress/wp-includes/blocks/post-date/
wordpress/wp-includes/blocks/post-date/style-rtl.min.css
wordpress/wp-includes/blocks/post-date/block.json
wordpress/wp-includes/blocks/post-date/style.min.css
wordpress/wp-includes/blocks/post-date/style-rtl.css
wordpress/wp-includes/blocks/post-date/style.css
wordpress/wp-includes/blocks/query-pagination-numbers.php
wordpress/wp-includes/blocks/tag-cloud/
wordpress/wp-includes/blocks/tag-cloud/style-rtl.min.css
wordpress/wp-includes/blocks/tag-cloud/block.json
wordpress/wp-includes/blocks/tag-cloud/style.min.css
wordpress/wp-includes/blocks/tag-cloud/style-rtl.css
wordpress/wp-includes/blocks/tag-cloud/style.css
wordpress/wp-includes/blocks/legacy-widget.php
wordpress/wp-includes/blocks/comment-edit-link.php
wordpress/wp-includes/blocks/missing/
wordpress/wp-includes/blocks/missing/block.json
wordpress/wp-includes/blocks/post-title/
wordpress/wp-includes/blocks/post-title/style-rtl.min.css
wordpress/wp-includes/blocks/post-title/block.json
wordpress/wp-includes/blocks/post-title/style.min.css
wordpress/wp-includes/blocks/post-title/style-rtl.css
wordpress/wp-includes/blocks/post-title/style.css
wordpress/wp-includes/blocks/index.php
wordpress/wp-includes/blocks/social-links/
wordpress/wp-includes/blocks/social-links/editor.min.css
wordpress/wp-includes/blocks/social-links/style-rtl.min.css
wordpress/wp-includes/blocks/social-links/block.json
wordpress/wp-includes/blocks/social-links/editor.css
wordpress/wp-includes/blocks/social-links/style.min.css
wordpress/wp-includes/blocks/social-links/editor-rtl.css
wordpress/wp-includes/blocks/social-links/editor-rtl.min.css
wordpress/wp-includes/blocks/social-links/style-rtl.css
wordpress/wp-includes/blocks/social-links/style.css
wordpress/wp-includes/blocks/page-list-item.php
wordpress/wp-includes/blocks/gallery.php
wordpress/wp-includes/blocks/query-pagination-next.php
wordpress/wp-includes/blocks/post-excerpt/
wordpress/wp-includes/blocks/post-excerpt/editor.min.css
wordpress/wp-includes/blocks/post-excerpt/style-rtl.min.css
wordpress/wp-includes/blocks/post-excerpt/block.json
wordpress/wp-includes/blocks/post-excerpt/editor.css
wordpress/wp-includes/blocks/post-excerpt/style.min.css
wordpress/wp-includes/blocks/post-excerpt/editor-rtl.css
wordpress/wp-includes/blocks/post-excerpt/editor-rtl.min.css
wordpress/wp-includes/blocks/post-excerpt/style-rtl.css
wordpress/wp-includes/blocks/post-excerpt/style.css
wordpress/wp-includes/blocks/preformatted/
wordpress/wp-includes/blocks/preformatted/style-rtl.min.css
wordpress/wp-includes/blocks/preformatted/block.json
wordpress/wp-includes/blocks/preformatted/style.min.css
wordpress/wp-includes/blocks/preformatted/style-rtl.css
wordpress/wp-includes/blocks/preformatted/style.css
wordpress/wp-includes/blocks/latest-posts.php
wordpress/wp-includes/blocks/latest-comments/
wordpress/wp-includes/blocks/latest-comments/style-rtl.min.css
wordpress/wp-includes/blocks/latest-comments/block.json
wordpress/wp-includes/blocks/latest-comments/style.min.css
wordpress/wp-includes/blocks/latest-comments/style-rtl.css
wordpress/wp-includes/blocks/latest-comments/style.css
wordpress/wp-includes/blocks/comments-pagination-previous/
wordpress/wp-includes/blocks/comments-pagination-previous/block.json
wordpress/wp-includes/blocks/comment-author-name.php
wordpress/wp-includes/blocks/comments-pagination-next/
wordpress/wp-includes/blocks/comments-pagination-next/block.json
wordpress/wp-includes/blocks/read-more.php
wordpress/wp-includes/blocks/cover.php
wordpress/wp-includes/blocks/search/
wordpress/wp-includes/blocks/search/editor.min.css
wordpress/wp-includes/blocks/search/style-rtl.min.css
wordpress/wp-includes/blocks/search/theme.css
wordpress/wp-includes/blocks/search/block.json
wordpress/wp-includes/blocks/search/theme-rtl.min.css
wordpress/wp-includes/blocks/search/editor.css
wordpress/wp-includes/blocks/search/view.min.asset.php
wordpress/wp-includes/blocks/search/view.js
wordpress/wp-includes/blocks/search/theme.min.css
wordpress/wp-includes/blocks/search/style.min.css
wordpress/wp-includes/blocks/search/theme-rtl.css
wordpress/wp-includes/blocks/search/editor-rtl.css
wordpress/wp-includes/blocks/search/view.asset.php
wordpress/wp-includes/blocks/search/editor-rtl.min.css
wordpress/wp-includes/blocks/search/view.min.js
wordpress/wp-includes/blocks/search/style-rtl.css
wordpress/wp-includes/blocks/search/style.css
wordpress/wp-includes/blocks/query-pagination-previous.php
wordpress/wp-includes/blocks/list-item/
wordpress/wp-includes/blocks/list-item/block.json
wordpress/wp-includes/blocks/navigation-link/
wordpress/wp-includes/blocks/navigation-link/editor.min.css
wordpress/wp-includes/blocks/navigation-link/style-rtl.min.css
wordpress/wp-includes/blocks/navigation-link/block.json
wordpress/wp-includes/blocks/navigation-link/editor.css
wordpress/wp-includes/blocks/navigation-link/style.min.css
wordpress/wp-includes/blocks/navigation-link/editor-rtl.css
wordpress/wp-includes/blocks/navigation-link/editor-rtl.min.css
wordpress/wp-includes/blocks/navigation-link/style-rtl.css
wordpress/wp-includes/blocks/navigation-link/style.css
wordpress/wp-includes/blocks/latest-posts/
wordpress/wp-includes/blocks/latest-posts/editor.min.css
wordpress/wp-includes/blocks/latest-posts/style-rtl.min.css
wordpress/wp-includes/blocks/latest-posts/block.json
wordpress/wp-includes/blocks/latest-posts/editor.css
wordpress/wp-includes/blocks/latest-posts/style.min.css
wordpress/wp-includes/blocks/latest-posts/editor-rtl.css
wordpress/wp-includes/blocks/latest-posts/editor-rtl.min.css
wordpress/wp-includes/blocks/latest-posts/style-rtl.css
wordpress/wp-includes/blocks/latest-posts/style.css
wordpress/wp-includes/blocks/template-part.php
wordpress/wp-includes/blocks/post-comments-form/
wordpress/wp-includes/blocks/post-comments-form/editor.min.css
wordpress/wp-includes/blocks/post-comments-form/style-rtl.min.css
wordpress/wp-includes/blocks/post-comments-form/block.json
wordpress/wp-includes/blocks/post-comments-form/editor.css
wordpress/wp-includes/blocks/post-comments-form/style.min.css
wordpress/wp-includes/blocks/post-comments-form/editor-rtl.css
wordpress/wp-includes/blocks/post-comments-form/editor-rtl.min.css
wordpress/wp-includes/blocks/post-comments-form/style-rtl.css
wordpress/wp-includes/blocks/post-comments-form/style.css
wordpress/wp-includes/blocks/post-author-biography.php
wordpress/wp-includes/blocks/term-description.php
wordpress/wp-includes/blocks/group/
wordpress/wp-includes/blocks/group/editor.min.css
wordpress/wp-includes/blocks/group/style-rtl.min.css
wordpress/wp-includes/blocks/group/theme.css
wordpress/wp-includes/blocks/group/block.json
wordpress/wp-includes/blocks/group/theme-rtl.min.css
wordpress/wp-includes/blocks/group/editor.css
wordpress/wp-includes/blocks/group/theme.min.css
wordpress/wp-includes/blocks/group/style.min.css
wordpress/wp-includes/blocks/group/theme-rtl.css
wordpress/wp-includes/blocks/group/editor-rtl.css
wordpress/wp-includes/blocks/group/editor-rtl.min.css
wordpress/wp-includes/blocks/group/style-rtl.css
wordpress/wp-includes/blocks/group/style.css
wordpress/wp-includes/blocks/post-terms/
wordpress/wp-includes/blocks/post-terms/style-rtl.min.css
wordpress/wp-includes/blocks/post-terms/block.json
wordpress/wp-includes/blocks/post-terms/style.min.css
wordpress/wp-includes/blocks/post-terms/style-rtl.css
wordpress/wp-includes/blocks/post-terms/style.css
wordpress/wp-includes/blocks/pattern.php
wordpress/wp-includes/blocks/heading.php
wordpress/wp-includes/blocks/avatar.php
wordpress/wp-includes/blocks/table/
wordpress/wp-includes/blocks/table/editor.min.css
wordpress/wp-includes/blocks/table/style-rtl.min.css
wordpress/wp-includes/blocks/table/theme.css
wordpress/wp-includes/blocks/table/block.json
wordpress/wp-includes/blocks/table/theme-rtl.min.css
wordpress/wp-includes/blocks/table/editor.css
wordpress/wp-includes/blocks/table/theme.min.css
wordpress/wp-includes/blocks/table/style.min.css
wordpress/wp-includes/blocks/table/theme-rtl.css
wordpress/wp-includes/blocks/table/editor-rtl.css
wordpress/wp-includes/blocks/table/editor-rtl.min.css
wordpress/wp-includes/blocks/table/style-rtl.css
wordpress/wp-includes/blocks/table/style.css
wordpress/wp-includes/blocks/comments.php
wordpress/wp-includes/blocks/list/
wordpress/wp-includes/blocks/list/style-rtl.min.css
wordpress/wp-includes/blocks/list/block.json
wordpress/wp-includes/blocks/list/style.min.css
wordpress/wp-includes/blocks/list/style-rtl.css
wordpress/wp-includes/blocks/list/style.css
wordpress/wp-includes/blocks/comment-author-name/
wordpress/wp-includes/blocks/comment-author-name/block.json
wordpress/wp-includes/blocks/file.php
wordpress/wp-includes/blocks/comments-pagination-numbers/
wordpress/wp-includes/blocks/comments-pagination-numbers/editor.min.css
wordpress/wp-includes/blocks/comments-pagination-numbers/block.json
wordpress/wp-includes/blocks/comments-pagination-numbers/editor.css
wordpress/wp-includes/blocks/comments-pagination-numbers/editor-rtl.css
wordpress/wp-includes/blocks/comments-pagination-numbers/editor-rtl.min.css
wordpress/wp-includes/blocks/comment-template.php
wordpress/wp-includes/blocks/query-pagination-next/
wordpress/wp-includes/blocks/query-pagination-next/block.json
wordpress/wp-includes/blocks/file/
wordpress/wp-includes/blocks/file/editor.min.css
wordpress/wp-includes/blocks/file/style-rtl.min.css
wordpress/wp-includes/blocks/file/block.json
wordpress/wp-includes/blocks/file/editor.css
wordpress/wp-includes/blocks/file/view.min.asset.php
wordpress/wp-includes/blocks/file/view.js
wordpress/wp-includes/blocks/file/style.min.css
wordpress/wp-includes/blocks/file/editor-rtl.css
wordpress/wp-includes/blocks/file/view.asset.php
wordpress/wp-includes/blocks/file/editor-rtl.min.css
wordpress/wp-includes/blocks/file/view.min.js
wordpress/wp-includes/blocks/file/style-rtl.css
wordpress/wp-includes/blocks/file/style.css
wordpress/wp-includes/blocks/image/
wordpress/wp-includes/blocks/image/editor.min.css
wordpress/wp-includes/blocks/image/style-rtl.min.css
wordpress/wp-includes/blocks/image/theme.css
wordpress/wp-includes/blocks/image/block.json
wordpress/wp-includes/blocks/image/theme-rtl.min.css
wordpress/wp-includes/blocks/image/editor.css
wordpress/wp-includes/blocks/image/view.min.asset.php
wordpress/wp-includes/blocks/image/view.js
wordpress/wp-includes/blocks/image/theme.min.css
wordpress/wp-includes/blocks/image/style.min.css
wordpress/wp-includes/blocks/image/theme-rtl.css
wordpress/wp-includes/blocks/image/editor-rtl.css
wordpress/wp-includes/blocks/image/view.asset.php
wordpress/wp-includes/blocks/image/editor-rtl.min.css
wordpress/wp-includes/blocks/image/view.min.js
wordpress/wp-includes/blocks/image/style-rtl.css
wordpress/wp-includes/blocks/image/style.css
wordpress/wp-includes/blocks/navigation.php
wordpress/wp-includes/blocks/query-title.php
wordpress/wp-includes/blocks/comments-title.php
wordpress/wp-includes/blocks/query-pagination.php
wordpress/wp-includes/blocks/text-columns/
wordpress/wp-includes/blocks/text-columns/editor.min.css
wordpress/wp-includes/blocks/text-columns/style-rtl.min.css
wordpress/wp-includes/blocks/text-columns/block.json
wordpress/wp-includes/blocks/text-columns/editor.css
wordpress/wp-includes/blocks/text-columns/style.min.css
wordpress/wp-includes/blocks/text-columns/editor-rtl.css
wordpress/wp-includes/blocks/text-columns/editor-rtl.min.css
wordpress/wp-includes/blocks/text-columns/style-rtl.css
wordpress/wp-includes/blocks/text-columns/style.css
wordpress/wp-includes/blocks/comment-template/
wordpress/wp-includes/blocks/comment-template/style-rtl.min.css
wordpress/wp-includes/blocks/comment-template/block.json
wordpress/wp-includes/blocks/comment-template/style.min.css
wordpress/wp-includes/blocks/comment-template/style-rtl.css
wordpress/wp-includes/blocks/comment-template/style.css
wordpress/wp-includes/blocks/video/
wordpress/wp-includes/blocks/video/editor.min.css
wordpress/wp-includes/blocks/video/style-rtl.min.css
wordpress/wp-includes/blocks/video/theme.css
wordpress/wp-includes/blocks/video/block.json
wordpress/wp-includes/blocks/video/theme-rtl.min.css
wordpress/wp-includes/blocks/video/editor.css
wordpress/wp-includes/blocks/video/theme.min.css
wordpress/wp-includes/blocks/video/style.min.css
wordpress/wp-includes/blocks/video/theme-rtl.css
wordpress/wp-includes/blocks/video/editor-rtl.css
wordpress/wp-includes/blocks/video/editor-rtl.min.css
wordpress/wp-includes/blocks/video/style-rtl.css
wordpress/wp-includes/blocks/video/style.css
wordpress/wp-includes/blocks/cover/
wordpress/wp-includes/blocks/cover/editor.min.css
wordpress/wp-includes/blocks/cover/style-rtl.min.css
wordpress/wp-includes/blocks/cover/block.json
wordpress/wp-includes/blocks/cover/editor.css
wordpress/wp-includes/blocks/cover/style.min.css
wordpress/wp-includes/blocks/cover/editor-rtl.css
wordpress/wp-includes/blocks/cover/editor-rtl.min.css
wordpress/wp-includes/blocks/cover/style-rtl.css
wordpress/wp-includes/blocks/cover/style.css
wordpress/wp-includes/blocks/post-template.php
wordpress/wp-includes/blocks/post-date.php
wordpress/wp-includes/blocks/post-author/
wordpress/wp-includes/blocks/post-author/style-rtl.min.css
wordpress/wp-includes/blocks/post-author/block.json
wordpress/wp-includes/blocks/post-author/style.min.css
wordpress/wp-includes/blocks/post-author/style-rtl.css
wordpress/wp-includes/blocks/post-author/style.css
wordpress/wp-includes/blocks/comment-reply-link.php
wordpress/wp-includes/blocks/tag-cloud.php
wordpress/wp-includes/blocks/site-logo.php
wordpress/wp-includes/blocks/legacy-widget/
wordpress/wp-includes/blocks/legacy-widget/block.json
wordpress/wp-includes/blocks/spacer/
wordpress/wp-includes/blocks/spacer/editor.min.css
wordpress/wp-includes/blocks/spacer/style-rtl.min.css
wordpress/wp-includes/blocks/spacer/block.json
wordpress/wp-includes/blocks/spacer/editor.css
wordpress/wp-includes/blocks/spacer/style.min.css
wordpress/wp-includes/blocks/spacer/editor-rtl.css
wordpress/wp-includes/blocks/spacer/editor-rtl.min.css
wordpress/wp-includes/blocks/spacer/style-rtl.css
wordpress/wp-includes/blocks/spacer/style.css
wordpress/wp-includes/class-wp-block-parser.php
wordpress/wp-includes/class-wp-plugin-dependencies.php
wordpress/wp-includes/pomo/
wordpress/wp-includes/pomo/translations.php
wordpress/wp-includes/pomo/plural-forms.php
wordpress/wp-includes/pomo/streams.php
wordpress/wp-includes/pomo/entry.php
wordpress/wp-includes/pomo/mo.php
wordpress/wp-includes/pomo/po.php
wordpress/wp-includes/customize/
wordpress/wp-includes/customize/class-wp-customize-nav-menu-section.php
wordpress/wp-includes/customize/class-wp-customize-upload-control.php
wordpress/wp-includes/customize/class-wp-customize-nav-menu-item-setting.php
wordpress/wp-includes/customize/class-wp-customize-nav-menu-item-control.php
wordpress/wp-includes/customize/class-wp-customize-selective-refresh.php
wordpress/wp-includes/customize/class-wp-customize-code-editor-control.php
wordpress/wp-includes/customize/class-wp-customize-header-image-control.php
wordpress/wp-includes/customize/class-wp-customize-themes-panel.php
wordpress/wp-includes/customize/class-wp-customize-filter-setting.php
wordpress/wp-includes/customize/class-wp-widget-area-customize-control.php
wordpress/wp-includes/customize/class-wp-customize-background-position-control.php
wordpress/wp-includes/customize/class-wp-customize-new-menu-control.php
wordpress/wp-includes/customize/class-wp-customize-nav-menus-panel.php
wordpress/wp-includes/customize/class-wp-customize-themes-section.php
wordpress/wp-includes/customize/class-wp-customize-background-image-control.php
wordpress/wp-includes/customize/class-wp-customize-site-icon-control.php
wordpress/wp-includes/customize/class-wp-customize-nav-menu-auto-add-control.php
wordpress/wp-includes/customize/class-wp-customize-background-image-setting.php
wordpress/wp-includes/customize/class-wp-customize-image-control.php
wordpress/wp-includes/customize/class-wp-customize-nav-menu-location-control.php
wordpress/wp-includes/customize/class-wp-customize-date-time-control.php
wordpress/wp-includes/customize/class-wp-customize-color-control.php
wordpress/wp-includes/customize/class-wp-sidebar-block-editor-control.php
wordpress/wp-includes/customize/class-wp-customize-custom-css-setting.php
wordpress/wp-includes/customize/class-wp-customize-header-image-setting.php
wordpress/wp-includes/customize/class-wp-customize-partial.php
wordpress/wp-includes/customize/class-wp-customize-nav-menu-locations-control.php
wordpress/wp-includes/customize/class-wp-customize-theme-control.php
wordpress/wp-includes/customize/class-wp-customize-new-menu-section.php
wordpress/wp-includes/customize/class-wp-customize-nav-menu-name-control.php
wordpress/wp-includes/customize/class-wp-customize-sidebar-section.php
wordpress/wp-includes/customize/class-wp-customize-nav-menu-control.php
wordpress/wp-includes/customize/class-wp-customize-cropped-image-control.php
wordpress/wp-includes/customize/class-wp-customize-media-control.php
wordpress/wp-includes/customize/class-wp-customize-nav-menu-setting.php
wordpress/wp-includes/customize/class-wp-widget-form-customize-control.php
wordpress/wp-includes/widgets/
wordpress/wp-includes/widgets/class-wp-widget-pages.php
wordpress/wp-includes/widgets/class-wp-widget-media-image.php
wordpress/wp-includes/widgets/class-wp-widget-search.php
wordpress/wp-includes/widgets/class-wp-widget-tag-cloud.php
wordpress/wp-includes/widgets/class-wp-widget-recent-comments.php
wordpress/wp-includes/widgets/class-wp-widget-media.php
wordpress/wp-includes/widgets/class-wp-widget-archives.php
wordpress/wp-includes/widgets/class-wp-widget-block.php
wordpress/wp-includes/widgets/class-wp-widget-custom-html.php
wordpress/wp-includes/widgets/class-wp-nav-menu-widget.php
wordpress/wp-includes/widgets/class-wp-widget-recent-posts.php
wordpress/wp-includes/widgets/class-wp-widget-links.php
wordpress/wp-includes/widgets/class-wp-widget-media-audio.php
wordpress/wp-includes/widgets/class-wp-widget-rss.php
wordpress/wp-includes/widgets/class-wp-widget-meta.php
wordpress/wp-includes/widgets/class-wp-widget-categories.php
wordpress/wp-includes/widgets/class-wp-widget-media-video.php
wordpress/wp-includes/widgets/class-wp-widget-calendar.php
wordpress/wp-includes/widgets/class-wp-widget-media-gallery.php
wordpress/wp-includes/widgets/class-wp-widget-text.php
wordpress/wp-includes/class-wp-image-editor.php
wordpress/wp-includes/deprecated.php
wordpress/wp-includes/class-wp-http-cookie.php
wordpress/wp-includes/class-wp-block-list.php
wordpress/wp-includes/feed-rss2-comments.php
wordpress/wp-includes/class-wp-date-query.php
wordpress/wp-includes/class-wp-oembed-controller.php
wordpress/wp-includes/class-walker-comment.php
wordpress/wp-includes/class-feed.php
wordpress/wp-includes/block-patterns.php
wordpress/wp-includes/class-wp-list-util.php
wordpress/wp-includes/class-wp-recovery-mode.php
wordpress/wp-includes/plugin.php
wordpress/wp-includes/class-walker-category-dropdown.php
wordpress/wp-includes/link-template.php
wordpress/wp-includes/comment.php
wordpress/wp-includes/class-wp-network-query.php
wordpress/wp-includes/https-detection.php
wordpress/wp-includes/registration.php
wordpress/wp-includes/PHPMailer/
wordpress/wp-includes/PHPMailer/SMTP.php
wordpress/wp-includes/PHPMailer/PHPMailer.php
wordpress/wp-includes/PHPMailer/Exception.php
wordpress/wp-includes/default-widgets.php
wordpress/wp-includes/class-oembed.php
wordpress/wp-includes/post-thumbnail-template.php
wordpress/wp-includes/capabilities.php
wordpress/wp-includes/class-wp-text-diff-renderer-inline.php
wordpress/wp-includes/l10n/
wordpress/wp-includes/l10n/class-wp-translations.php
wordpress/wp-includes/l10n/class-wp-translation-file.php
wordpress/wp-includes/l10n/class-wp-translation-controller.php
wordpress/wp-includes/l10n/class-wp-translation-file-php.php
wordpress/wp-includes/l10n/class-wp-translation-file-mo.php
wordpress/wp-includes/version.php
wordpress/wp-includes/update.php
wordpress/wp-includes/class.wp-styles.php
wordpress/wp-includes/class-wp-error.php
wordpress/wp-includes/block-bindings.php
wordpress/wp-includes/class-wp-post-type.php
wordpress/wp-includes/class-wp-meta-query.php
wordpress/wp-includes/class-wp-feed-cache.php
wordpress/wp-includes/fonts/
wordpress/wp-includes/fonts/dashicons.eot
wordpress/wp-includes/fonts/dashicons.woff
wordpress/wp-includes/fonts/class-wp-font-library.php
wordpress/wp-includes/fonts/class-wp-font-face-resolver.php
wordpress/wp-includes/fonts/class-wp-font-face.php
wordpress/wp-includes/fonts/class-wp-font-collection.php
wordpress/wp-includes/fonts/dashicons.woff2
wordpress/wp-includes/fonts/dashicons.ttf
wordpress/wp-includes/fonts/class-wp-font-utils.php
wordpress/wp-includes/fonts/dashicons.svg
wordpress/wp-includes/class-wp-taxonomy.php
wordpress/wp-includes/http.php
wordpress/wp-includes/atomlib.php
wordpress/wp-includes/class-wp-customize-nav-menus.php
wordpress/wp-includes/functions.php
wordpress/wp-includes/class-wp-term.php
wordpress/wp-includes/author-template.php
wordpress/wp-includes/pluggable.php
wordpress/wp-includes/class-wp-recovery-mode-key-service.php
wordpress/wp-includes/class.wp-scripts.php
wordpress/wp-includes/class-wp-user-request.php
wordpress/wp-includes/class-wp-theme-json-resolver.php
wordpress/wp-includes/error-protection.php
wordpress/wp-includes/class-json.php
wordpress/wp-includes/class-wp-user.php
wordpress/wp-includes/class-wp-block-type.php
wordpress/wp-includes/class-wp-ajax-response.php
wordpress/wp-includes/class-wp-textdomain-registry.php
wordpress/wp-includes/sodium_compat/
wordpress/wp-includes/sodium_compat/src/
wordpress/wp-includes/sodium_compat/src/Core32/
wordpress/wp-includes/sodium_compat/src/Core32/Int32.php
wordpress/wp-includes/sodium_compat/src/Core32/SipHash.php
wordpress/wp-includes/sodium_compat/src/Core32/Ed25519.php
wordpress/wp-includes/sodium_compat/src/Core32/Curve25519.php
wordpress/wp-includes/sodium_compat/src/Core32/SecretStream/
wordpress/wp-includes/sodium_compat/src/Core32/SecretStream/State.php
wordpress/wp-includes/sodium_compat/src/Core32/XChaCha20.php
wordpress/wp-includes/sodium_compat/src/Core32/Poly1305/
wordpress/wp-includes/sodium_compat/src/Core32/Poly1305/State.php
wordpress/wp-includes/sodium_compat/src/Core32/ChaCha20.php
wordpress/wp-includes/sodium_compat/src/Core32/Salsa20.php
wordpress/wp-includes/sodium_compat/src/Core32/Int64.php
wordpress/wp-includes/sodium_compat/src/Core32/ChaCha20/
wordpress/wp-includes/sodium_compat/src/Core32/ChaCha20/IetfCtx.php
wordpress/wp-includes/sodium_compat/src/Core32/ChaCha20/Ctx.php
wordpress/wp-includes/sodium_compat/src/Core32/XSalsa20.php
wordpress/wp-includes/sodium_compat/src/Core32/Poly1305.php
wordpress/wp-includes/sodium_compat/src/Core32/HChaCha20.php
wordpress/wp-includes/sodium_compat/src/Core32/Util.php
wordpress/wp-includes/sodium_compat/src/Core32/HSalsa20.php
wordpress/wp-includes/sodium_compat/src/Core32/Curve25519/
wordpress/wp-includes/sodium_compat/src/Core32/Curve25519/H.php
wordpress/wp-includes/sodium_compat/src/Core32/Curve25519/Fe.php
wordpress/wp-includes/sodium_compat/src/Core32/Curve25519/README.md
wordpress/wp-includes/sodium_compat/src/Core32/Curve25519/Ge/
wordpress/wp-includes/sodium_compat/src/Core32/Curve25519/Ge/P2.php
wordpress/wp-includes/sodium_compat/src/Core32/Curve25519/Ge/Cached.php
wordpress/wp-includes/sodium_compat/src/Core32/Curve25519/Ge/P3.php
wordpress/wp-includes/sodium_compat/src/Core32/Curve25519/Ge/Precomp.php
wordpress/wp-includes/sodium_compat/src/Core32/Curve25519/Ge/P1p1.php
wordpress/wp-includes/sodium_compat/src/Core32/X25519.php
wordpress/wp-includes/sodium_compat/src/Core32/BLAKE2b.php
wordpress/wp-includes/sodium_compat/src/PHP52/
wordpress/wp-includes/sodium_compat/src/PHP52/SplFixedArray.php
wordpress/wp-includes/sodium_compat/src/Crypto.php
wordpress/wp-includes/sodium_compat/src/Compat.php
wordpress/wp-includes/sodium_compat/src/Core/
wordpress/wp-includes/sodium_compat/src/Core/Base64/
wordpress/wp-includes/sodium_compat/src/Core/Base64/Original.php
wordpress/wp-includes/sodium_compat/src/Core/Base64/Common.php
wordpress/wp-includes/sodium_compat/src/Core/Base64/UrlSafe.php
wordpress/wp-includes/sodium_compat/src/Core/SipHash.php
wordpress/wp-includes/sodium_compat/src/Core/Ed25519.php
wordpress/wp-includes/sodium_compat/src/Core/Ristretto255.php
wordpress/wp-includes/sodium_compat/src/Core/Curve25519.php
wordpress/wp-includes/sodium_compat/src/Core/SecretStream/
wordpress/wp-includes/sodium_compat/src/Core/SecretStream/State.php
wordpress/wp-includes/sodium_compat/src/Core/XChaCha20.php
wordpress/wp-includes/sodium_compat/src/Core/Poly1305/
wordpress/wp-includes/sodium_compat/src/Core/Poly1305/State.php
wordpress/wp-includes/sodium_compat/src/Core/ChaCha20.php
wordpress/wp-includes/sodium_compat/src/Core/Salsa20.php
wordpress/wp-includes/sodium_compat/src/Core/ChaCha20/
wordpress/wp-includes/sodium_compat/src/Core/ChaCha20/IetfCtx.php
wordpress/wp-includes/sodium_compat/src/Core/ChaCha20/Ctx.php
wordpress/wp-includes/sodium_compat/src/Core/XSalsa20.php
wordpress/wp-includes/sodium_compat/src/Core/Poly1305.php
wordpress/wp-includes/sodium_compat/src/Core/HChaCha20.php
wordpress/wp-includes/sodium_compat/src/Core/Util.php
wordpress/wp-includes/sodium_compat/src/Core/HSalsa20.php
wordpress/wp-includes/sodium_compat/src/Core/Curve25519/
wordpress/wp-includes/sodium_compat/src/Core/Curve25519/H.php
wordpress/wp-includes/sodium_compat/src/Core/Curve25519/Fe.php
wordpress/wp-includes/sodium_compat/src/Core/Curve25519/README.md
wordpress/wp-includes/sodium_compat/src/Core/Curve25519/Ge/
wordpress/wp-includes/sodium_compat/src/Core/Curve25519/Ge/P2.php
wordpress/wp-includes/sodium_compat/src/Core/Curve25519/Ge/Cached.php
wordpress/wp-includes/sodium_compat/src/Core/Curve25519/Ge/P3.php
wordpress/wp-includes/sodium_compat/src/Core/Curve25519/Ge/Precomp.php
wordpress/wp-includes/sodium_compat/src/Core/Curve25519/Ge/P1p1.php
wordpress/wp-includes/sodium_compat/src/Core/X25519.php
wordpress/wp-includes/sodium_compat/src/Core/BLAKE2b.php
wordpress/wp-includes/sodium_compat/src/SodiumException.php
wordpress/wp-includes/sodium_compat/src/Crypto32.php
wordpress/wp-includes/sodium_compat/src/File.php
wordpress/wp-includes/sodium_compat/autoload-php7.php
wordpress/wp-includes/sodium_compat/composer.json
wordpress/wp-includes/sodium_compat/namespaced/
wordpress/wp-includes/sodium_compat/namespaced/Crypto.php
wordpress/wp-includes/sodium_compat/namespaced/Compat.php
wordpress/wp-includes/sodium_compat/namespaced/Core/
wordpress/wp-includes/sodium_compat/namespaced/Core/SipHash.php
wordpress/wp-includes/sodium_compat/namespaced/Core/Xsalsa20.php
wordpress/wp-includes/sodium_compat/namespaced/Core/Ed25519.php
wordpress/wp-includes/sodium_compat/namespaced/Core/Curve25519.php
wordpress/wp-includes/sodium_compat/namespaced/Core/XChaCha20.php
wordpress/wp-includes/sodium_compat/namespaced/Core/Poly1305/
wordpress/wp-includes/sodium_compat/namespaced/Core/Poly1305/State.php
wordpress/wp-includes/sodium_compat/namespaced/Core/ChaCha20.php
wordpress/wp-includes/sodium_compat/namespaced/Core/Salsa20.php
wordpress/wp-includes/sodium_compat/namespaced/Core/ChaCha20/
wordpress/wp-includes/sodium_compat/namespaced/Core/ChaCha20/IetfCtx.php
wordpress/wp-includes/sodium_compat/namespaced/Core/ChaCha20/Ctx.php
wordpress/wp-includes/sodium_compat/namespaced/Core/Poly1305.php
wordpress/wp-includes/sodium_compat/namespaced/Core/HChaCha20.php
wordpress/wp-includes/sodium_compat/namespaced/Core/Util.php
wordpress/wp-includes/sodium_compat/namespaced/Core/HSalsa20.php
wordpress/wp-includes/sodium_compat/namespaced/Core/Curve25519/
wordpress/wp-includes/sodium_compat/namespaced/Core/Curve25519/H.php
wordpress/wp-includes/sodium_compat/namespaced/Core/Curve25519/Fe.php
wordpress/wp-includes/sodium_compat/namespaced/Core/Curve25519/Ge/
wordpress/wp-includes/sodium_compat/namespaced/Core/Curve25519/Ge/P2.php
wordpress/wp-includes/sodium_compat/namespaced/Core/Curve25519/Ge/Cached.php
wordpress/wp-includes/sodium_compat/namespaced/Core/Curve25519/Ge/P3.php
wordpress/wp-includes/sodium_compat/namespaced/Core/Curve25519/Ge/Precomp.php
wordpress/wp-includes/sodium_compat/namespaced/Core/Curve25519/Ge/P1p1.php
wordpress/wp-includes/sodium_compat/namespaced/Core/X25519.php
wordpress/wp-includes/sodium_compat/namespaced/Core/BLAKE2b.php
wordpress/wp-includes/sodium_compat/namespaced/File.php
wordpress/wp-includes/sodium_compat/lib/
wordpress/wp-includes/sodium_compat/lib/namespaced.php
wordpress/wp-includes/sodium_compat/lib/constants.php
wordpress/wp-includes/sodium_compat/lib/php72compat.php
wordpress/wp-includes/sodium_compat/lib/php72compat_const.php
wordpress/wp-includes/sodium_compat/lib/sodium_compat.php
wordpress/wp-includes/sodium_compat/lib/ristretto255.php
wordpress/wp-includes/sodium_compat/lib/stream-xchacha20.php
wordpress/wp-includes/sodium_compat/LICENSE
wordpress/wp-includes/sodium_compat/autoload.php
wordpress/wp-includes/robots-template.php
wordpress/wp-includes/sitemaps/
wordpress/wp-includes/sitemaps/class-wp-sitemaps-renderer.php
wordpress/wp-includes/sitemaps/class-wp-sitemaps-provider.php
wordpress/wp-includes/sitemaps/class-wp-sitemaps-index.php
wordpress/wp-includes/sitemaps/class-wp-sitemaps-stylesheet.php
wordpress/wp-includes/sitemaps/providers/
wordpress/wp-includes/sitemaps/providers/class-wp-sitemaps-taxonomies.php
wordpress/wp-includes/sitemaps/providers/class-wp-sitemaps-posts.php
wordpress/wp-includes/sitemaps/providers/class-wp-sitemaps-users.php
wordpress/wp-includes/sitemaps/class-wp-sitemaps-registry.php
wordpress/wp-includes/sitemaps/class-wp-sitemaps.php
wordpress/wp-includes/session.php
wordpress/wp-includes/rest-api/
wordpress/wp-includes/rest-api/endpoints/
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-font-faces-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-block-patterns-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-application-passwords-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-widget-types-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-pattern-directory-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-navigation-fallback-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-block-types-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-themes-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-font-families-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-widgets-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-templates-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-search-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-url-details-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-attachments-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-users-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-template-autosaves-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-global-styles-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-sidebars-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-block-pattern-categories-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-block-renderer-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-posts-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-comments-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-menu-items-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-menu-locations-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-post-types-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-autosaves-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-edit-site-export-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-terms-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-plugins-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-font-collections-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-site-health-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-blocks-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-post-statuses-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-settings-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-revisions-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-block-directory-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-menus-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-taxonomies-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-template-revisions-controller.php
wordpress/wp-includes/rest-api/endpoints/class-wp-rest-global-styles-revisions-controller.php
wordpress/wp-includes/rest-api/class-wp-rest-response.php
wordpress/wp-includes/rest-api/class-wp-rest-server.php
wordpress/wp-includes/rest-api/search/
wordpress/wp-includes/rest-api/search/class-wp-rest-post-search-handler.php
wordpress/wp-includes/rest-api/search/class-wp-rest-post-format-search-handler.php
wordpress/wp-includes/rest-api/search/class-wp-rest-term-search-handler.php
wordpress/wp-includes/rest-api/search/class-wp-rest-search-handler.php
wordpress/wp-includes/rest-api/fields/
wordpress/wp-includes/rest-api/fields/class-wp-rest-post-meta-fields.php
wordpress/wp-includes/rest-api/fields/class-wp-rest-meta-fields.php
wordpress/wp-includes/rest-api/fields/class-wp-rest-term-meta-fields.php
wordpress/wp-includes/rest-api/fields/class-wp-rest-user-meta-fields.php
wordpress/wp-includes/rest-api/fields/class-wp-rest-comment-meta-fields.php
wordpress/wp-includes/rest-api/class-wp-rest-request.php
wordpress/wp-includes/class-wp-classic-to-block-menu-converter.php
wordpress/wp-includes/class-wp-admin-bar.php
wordpress/wp-includes/class-wp-script-modules.php
wordpress/wp-includes/block-editor.php
wordpress/wp-includes/class-wp-hook.php
wordpress/wp-includes/class-wp-block-patterns-registry.php
wordpress/wp-includes/https-migration.php
wordpress/wp-includes/post.php
wordpress/wp-includes/class-wp-object-cache.php
wordpress/wp-includes/class-wp-walker.php
wordpress/wp-includes/class-wp-rewrite.php
wordpress/wp-includes/ms-network.php
wordpress/wp-includes/class-wp-customize-setting.php
wordpress/wp-includes/IXR/
wordpress/wp-includes/IXR/class-IXR-server.php
wordpress/wp-includes/IXR/class-IXR-message.php
wordpress/wp-includes/IXR/class-IXR-value.php
wordpress/wp-includes/IXR/class-IXR-introspectionserver.php
wordpress/wp-includes/IXR/class-IXR-client.php
wordpress/wp-includes/IXR/class-IXR-clientmulticall.php
wordpress/wp-includes/IXR/class-IXR-base64.php
wordpress/wp-includes/IXR/class-IXR-date.php
wordpress/wp-includes/IXR/class-IXR-request.php
wordpress/wp-includes/IXR/class-IXR-error.php
wordpress/wp-includes/class-wp-block.php
wordpress/wp-includes/class-wp-query.php
wordpress/wp-includes/class-wp-scripts.php
wordpress/wp-includes/block-patterns/
wordpress/wp-includes/block-patterns/query-offset-posts.php
wordpress/wp-includes/block-patterns/query-large-title-posts.php
wordpress/wp-includes/block-patterns/social-links-shared-background-color.php
wordpress/wp-includes/block-patterns/query-grid-posts.php
wordpress/wp-includes/block-patterns/query-medium-posts.php
wordpress/wp-includes/block-patterns/query-standard-posts.php
wordpress/wp-includes/block-patterns/query-small-posts.php
wordpress/wp-includes/class-wp.php
wordpress/wp-includes/spl-autoload-compat.php
wordpress/wp-includes/registration-functions.php
wordpress/wp-includes/class-wp-application-passwords.php
wordpress/wp-includes/global-styles-and-settings.php
wordpress/wp-includes/feed-rdf.php
wordpress/wp-includes/load.php
wordpress/wp-includes/bookmark.php
wordpress/wp-includes/comment-template.php
wordpress/wp-includes/class-wp-block-styles-registry.php
wordpress/wp-includes/theme-i18n.json
wordpress/wp-includes/fonts.php
wordpress/wp-includes/admin-bar.php
wordpress/wp-includes/ms-load.php
wordpress/wp-includes/class.wp-dependencies.php
wordpress/wp-includes/class-wp-theme-json-schema.php
wordpress/wp-includes/class-wp-http-proxy.php
wordpress/wp-includes/class-wp-block-parser-frame.php
wordpress/wp-includes/block-bindings/
wordpress/wp-includes/block-bindings/post-meta.php
wordpress/wp-includes/block-bindings/pattern-overrides.php
wordpress/wp-includes/class-wp-dependency.php
wordpress/wp-includes/js/
wordpress/wp-includes/js/utils.js
wordpress/wp-includes/js/customize-preview.min.js
wordpress/wp-includes/js/customize-preview-nav-menus.js
wordpress/wp-includes/js/autosave.js
wordpress/wp-includes/js/customize-selective-refresh.js
wordpress/wp-includes/js/customize-preview-widgets.js
wordpress/wp-includes/js/hoverIntent.min.js
wordpress/wp-includes/js/media-grid.js
wordpress/wp-includes/js/comment-reply.js
wordpress/wp-includes/js/customize-loader.js
wordpress/wp-includes/js/wp-sanitize.js
wordpress/wp-includes/js/wp-emoji.js
wordpress/wp-includes/js/swfobject.js
wordpress/wp-includes/js/wp-sanitize.min.js
wordpress/wp-includes/js/wp-emoji-loader.js
wordpress/wp-includes/js/media-audiovideo.js
wordpress/wp-includes/js/zxcvbn-async.js
wordpress/wp-includes/js/colorpicker.min.js
wordpress/wp-includes/js/hoverintent-js.min.js
wordpress/wp-includes/js/backbone.js
wordpress/wp-includes/js/json2.js
wordpress/wp-includes/js/media-editor.js
wordpress/wp-includes/js/customize-preview-widgets.min.js
wordpress/wp-includes/js/media-grid.min.js
wordpress/wp-includes/js/wplink.js
wordpress/wp-includes/js/customize-views.min.js
wordpress/wp-includes/js/underscore.min.js
wordpress/wp-includes/js/wp-embed-template.js
wordpress/wp-includes/js/wp-backbone.js
wordpress/wp-includes/js/utils.min.js
wordpress/wp-includes/js/customize-preview-nav-menus.min.js
wordpress/wp-includes/js/media-models.js
wordpress/wp-includes/js/wp-embed.js
wordpress/wp-includes/js/comment-reply.min.js
wordpress/wp-includes/js/wp-util.min.js
wordpress/wp-includes/js/tw-sack.min.js
wordpress/wp-includes/js/heartbeat.js
wordpress/wp-includes/js/media-views.js
wordpress/wp-includes/js/dist/
wordpress/wp-includes/js/dist/annotations.min.js
wordpress/wp-includes/js/dist/core-data.js
wordpress/wp-includes/js/dist/list-reusable-blocks.js
wordpress/wp-includes/js/dist/dom.min.js
wordpress/wp-includes/js/dist/block-directory.js
wordpress/wp-includes/js/dist/edit-widgets.js
wordpress/wp-includes/js/dist/core-commands.js
wordpress/wp-includes/js/dist/keyboard-shortcuts.js
wordpress/wp-includes/js/dist/patterns.js
wordpress/wp-includes/js/dist/url.min.js
wordpress/wp-includes/js/dist/components.js
wordpress/wp-includes/js/dist/edit-site.min.js
wordpress/wp-includes/js/dist/wordcount.js
wordpress/wp-includes/js/dist/vendor/
wordpress/wp-includes/js/dist/vendor/wp-polyfill-object-fit.min.js
wordpress/wp-includes/js/dist/vendor/regenerator-runtime.min.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill-object-fit.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill-inert.js
wordpress/wp-includes/js/dist/vendor/lodash.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill-fetch.min.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill-dom-rect.min.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill-element-closest.min.js
wordpress/wp-includes/js/dist/vendor/moment.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill-formdata.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill-node-contains.min.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill.min.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill-importmap.js
wordpress/wp-includes/js/dist/vendor/react.js
wordpress/wp-includes/js/dist/vendor/react-dom.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill-url.min.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill-element-closest.js
wordpress/wp-includes/js/dist/vendor/react.min.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill-node-contains.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill-url.js
wordpress/wp-includes/js/dist/vendor/regenerator-runtime.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill-inert.min.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill-dom-rect.js
wordpress/wp-includes/js/dist/vendor/lodash.min.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill-importmap.min.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill-fetch.js
wordpress/wp-includes/js/dist/vendor/wp-polyfill-formdata.min.js
wordpress/wp-includes/js/dist/vendor/react-dom.min.js
wordpress/wp-includes/js/dist/vendor/moment.min.js
wordpress/wp-includes/js/dist/blocks.js
wordpress/wp-includes/js/dist/private-apis.js
wordpress/wp-includes/js/dist/keyboard-shortcuts.min.js
wordpress/wp-includes/js/dist/router.js
wordpress/wp-includes/js/dist/block-library.min.js
wordpress/wp-includes/js/dist/token-list.js
wordpress/wp-includes/js/dist/widgets.min.js
wordpress/wp-includes/js/dist/reusable-blocks.js
wordpress/wp-includes/js/dist/style-engine.js
wordpress/wp-includes/js/dist/core-commands.min.js
wordpress/wp-includes/js/dist/commands.min.js
wordpress/wp-includes/js/dist/data.min.js
wordpress/wp-includes/js/dist/api-fetch.js
wordpress/wp-includes/js/dist/list-reusable-blocks.min.js
wordpress/wp-includes/js/dist/style-engine.min.js
wordpress/wp-includes/js/dist/interactivity.min.js
wordpress/wp-includes/js/dist/deprecated.min.js
wordpress/wp-includes/js/dist/edit-widgets.min.js
wordpress/wp-includes/js/dist/date.min.js
wordpress/wp-includes/js/dist/escape-html.min.js
wordpress/wp-includes/js/dist/edit-post.min.js
wordpress/wp-includes/js/dist/block-editor.min.js
wordpress/wp-includes/js/dist/dom-ready.js
wordpress/wp-includes/js/dist/blob.js
wordpress/wp-includes/js/dist/priority-queue.js
wordpress/wp-includes/js/dist/core-data.min.js
wordpress/wp-includes/js/dist/preferences.js
wordpress/wp-includes/js/dist/block-serialization-default-parser.js
wordpress/wp-includes/js/dist/compose.min.js
wordpress/wp-includes/js/dist/components.min.js
wordpress/wp-includes/js/dist/editor.min.js
wordpress/wp-includes/js/dist/is-shallow-equal.min.js
wordpress/wp-includes/js/dist/format-library.js
wordpress/wp-includes/js/dist/media-utils.min.js
wordpress/wp-includes/js/dist/date.js
wordpress/wp-includes/js/dist/i18n.js
wordpress/wp-includes/js/dist/edit-site.js
wordpress/wp-includes/js/dist/keycodes.min.js
wordpress/wp-includes/js/dist/preferences-persistence.min.js
wordpress/wp-includes/js/dist/block-serialization-default-parser.min.js
wordpress/wp-includes/js/dist/customize-widgets.js
wordpress/wp-includes/js/dist/notices.js
wordpress/wp-includes/js/dist/html-entities.min.js
wordpress/wp-includes/js/dist/preferences-persistence.js
wordpress/wp-includes/js/dist/hooks.js
wordpress/wp-includes/js/dist/media-utils.js
wordpress/wp-includes/js/dist/a11y.js
wordpress/wp-includes/js/dist/hooks.min.js
wordpress/wp-includes/js/dist/router.min.js
wordpress/wp-includes/js/dist/preferences.min.js
wordpress/wp-includes/js/dist/redux-routine.js
wordpress/wp-includes/js/dist/reusable-blocks.min.js
wordpress/wp-includes/js/dist/primitives.js
wordpress/wp-includes/js/dist/block-editor.js
wordpress/wp-includes/js/dist/warning.min.js
wordpress/wp-includes/js/dist/escape-html.js
wordpress/wp-includes/js/dist/i18n.min.js
wordpress/wp-includes/js/dist/rich-text.min.js
wordpress/wp-includes/js/dist/api-fetch.min.js
wordpress/wp-includes/js/dist/viewport.min.js
wordpress/wp-includes/js/dist/redux-routine.min.js
wordpress/wp-includes/js/dist/warning.js
wordpress/wp-includes/js/dist/notices.min.js
wordpress/wp-includes/js/dist/autop.min.js
wordpress/wp-includes/js/dist/undo-manager.js
wordpress/wp-includes/js/dist/data-controls.js
wordpress/wp-includes/js/dist/blob.min.js
wordpress/wp-includes/js/dist/block-library.js
wordpress/wp-includes/js/dist/token-list.min.js
wordpress/wp-includes/js/dist/plugins.js
wordpress/wp-includes/js/dist/editor.js
wordpress/wp-includes/js/dist/html-entities.js
wordpress/wp-includes/js/dist/deprecated.js
wordpress/wp-includes/js/dist/undo-manager.min.js
wordpress/wp-includes/js/dist/data.js
wordpress/wp-includes/js/dist/viewport.js
wordpress/wp-includes/js/dist/compose.js
wordpress/wp-includes/js/dist/nux.min.js
wordpress/wp-includes/js/dist/block-directory.min.js
wordpress/wp-includes/js/dist/commands.js
wordpress/wp-includes/js/dist/shortcode.min.js
wordpress/wp-includes/js/dist/widgets.js
wordpress/wp-includes/js/dist/interactivity-router.js
wordpress/wp-includes/js/dist/interactivity-router.min.js
wordpress/wp-includes/js/dist/blocks.min.js
wordpress/wp-includes/js/dist/customize-widgets.min.js
wordpress/wp-includes/js/dist/development/
wordpress/wp-includes/js/dist/development/react-refresh-runtime.js
wordpress/wp-includes/js/dist/development/react-refresh-entry.min.js
wordpress/wp-includes/js/dist/development/react-refresh-runtime.min.js
wordpress/wp-includes/js/dist/development/react-refresh-entry.js
wordpress/wp-includes/js/dist/patterns.min.js
wordpress/wp-includes/js/dist/nux.js
wordpress/wp-includes/js/dist/interactivity.js
wordpress/wp-includes/js/dist/priority-queue.min.js
wordpress/wp-includes/js/dist/format-library.min.js
wordpress/wp-includes/js/dist/plugins.min.js
wordpress/wp-includes/js/dist/shortcode.js
wordpress/wp-includes/js/dist/dom.js
wordpress/wp-includes/js/dist/data-controls.min.js
wordpress/wp-includes/js/dist/dom-ready.min.js
wordpress/wp-includes/js/dist/keycodes.js
wordpress/wp-includes/js/dist/element.js
wordpress/wp-includes/js/dist/private-apis.min.js
wordpress/wp-includes/js/dist/is-shallow-equal.js
wordpress/wp-includes/js/dist/rich-text.js
wordpress/wp-includes/js/dist/primitives.min.js
wordpress/wp-includes/js/dist/edit-post.js
wordpress/wp-includes/js/dist/autop.js
wordpress/wp-includes/js/dist/annotations.js
wordpress/wp-includes/js/dist/element.min.js
wordpress/wp-includes/js/dist/interactivity-router.min.asset.php
wordpress/wp-includes/js/dist/wordcount.min.js
wordpress/wp-includes/js/dist/a11y.min.js
wordpress/wp-includes/js/dist/server-side-render.min.js
wordpress/wp-includes/js/dist/interactivity-router.asset.php
wordpress/wp-includes/js/dist/url.js
wordpress/wp-includes/js/dist/server-side-render.js
wordpress/wp-includes/js/wp-embed-template.min.js
wordpress/wp-includes/js/customize-models.js
wordpress/wp-includes/js/wp-api.js
wordpress/wp-includes/js/tinymce/
wordpress/wp-includes/js/tinymce/tiny_mce_popup.js
wordpress/wp-includes/js/tinymce/langs/
wordpress/wp-includes/js/tinymce/langs/wp-langs-en.js
wordpress/wp-includes/js/tinymce/wp-tinymce.js
wordpress/wp-includes/js/tinymce/skins/
wordpress/wp-includes/js/tinymce/skins/wordpress/
wordpress/wp-includes/js/tinymce/skins/wordpress/images/
wordpress/wp-includes/js/tinymce/skins/wordpress/images/gallery.png
wordpress/wp-includes/js/tinymce/skins/wordpress/images/pagebreak.png
wordpress/wp-includes/js/tinymce/skins/wordpress/images/audio.png
wordpress/wp-includes/js/tinymce/skins/wordpress/images/more.png
wordpress/wp-includes/js/tinymce/skins/wordpress/images/pagebreak-2x.png
wordpress/wp-includes/js/tinymce/skins/wordpress/images/more-2x.png
wordpress/wp-includes/js/tinymce/skins/wordpress/images/dashicon-edit.png
wordpress/wp-includes/js/tinymce/skins/wordpress/images/playlist-audio.png
wordpress/wp-includes/js/tinymce/skins/wordpress/images/video.png
wordpress/wp-includes/js/tinymce/skins/wordpress/images/embedded.png
wordpress/wp-includes/js/tinymce/skins/wordpress/images/gallery-2x.png
wordpress/wp-includes/js/tinymce/skins/wordpress/images/dashicon-no.png
wordpress/wp-includes/js/tinymce/skins/wordpress/images/playlist-video.png
wordpress/wp-includes/js/tinymce/skins/wordpress/wp-content.css
wordpress/wp-includes/js/tinymce/skins/lightgray/
wordpress/wp-includes/js/tinymce/skins/lightgray/content.min.css
wordpress/wp-includes/js/tinymce/skins/lightgray/content.inline.min.css
wordpress/wp-includes/js/tinymce/skins/lightgray/img/
wordpress/wp-includes/js/tinymce/skins/lightgray/img/object.gif
wordpress/wp-includes/js/tinymce/skins/lightgray/img/loader.gif
wordpress/wp-includes/js/tinymce/skins/lightgray/img/trans.gif
wordpress/wp-includes/js/tinymce/skins/lightgray/img/anchor.gif
wordpress/wp-includes/js/tinymce/skins/lightgray/fonts/
wordpress/wp-includes/js/tinymce/skins/lightgray/fonts/tinymce.ttf
wordpress/wp-includes/js/tinymce/skins/lightgray/fonts/tinymce-small.ttf
wordpress/wp-includes/js/tinymce/skins/lightgray/fonts/tinymce.eot
wordpress/wp-includes/js/tinymce/skins/lightgray/fonts/tinymce-small.svg
wordpress/wp-includes/js/tinymce/skins/lightgray/fonts/tinymce.svg
wordpress/wp-includes/js/tinymce/skins/lightgray/fonts/tinymce-small.eot
wordpress/wp-includes/js/tinymce/skins/lightgray/fonts/tinymce-small.woff
wordpress/wp-includes/js/tinymce/skins/lightgray/fonts/tinymce.woff
wordpress/wp-includes/js/tinymce/skins/lightgray/skin.min.css
wordpress/wp-includes/js/tinymce/themes/
wordpress/wp-includes/js/tinymce/themes/inlite/
wordpress/wp-includes/js/tinymce/themes/inlite/theme.min.js
wordpress/wp-includes/js/tinymce/themes/inlite/theme.js
wordpress/wp-includes/js/tinymce/themes/modern/
wordpress/wp-includes/js/tinymce/themes/modern/theme.min.js
wordpress/wp-includes/js/tinymce/themes/modern/theme.js
wordpress/wp-includes/js/tinymce/plugins/
wordpress/wp-includes/js/tinymce/plugins/fullscreen/
wordpress/wp-includes/js/tinymce/plugins/fullscreen/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/fullscreen/plugin.js
wordpress/wp-includes/js/tinymce/plugins/wplink/
wordpress/wp-includes/js/tinymce/plugins/wplink/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/wplink/plugin.js
wordpress/wp-includes/js/tinymce/plugins/wpemoji/
wordpress/wp-includes/js/tinymce/plugins/wpemoji/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/wpemoji/plugin.js
wordpress/wp-includes/js/tinymce/plugins/media/
wordpress/wp-includes/js/tinymce/plugins/media/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/media/plugin.js
wordpress/wp-includes/js/tinymce/plugins/wptextpattern/
wordpress/wp-includes/js/tinymce/plugins/wptextpattern/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/wptextpattern/plugin.js
wordpress/wp-includes/js/tinymce/plugins/wpdialogs/
wordpress/wp-includes/js/tinymce/plugins/wpdialogs/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/wpdialogs/plugin.js
wordpress/wp-includes/js/tinymce/plugins/wpview/
wordpress/wp-includes/js/tinymce/plugins/wpview/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/wpview/plugin.js
wordpress/wp-includes/js/tinymce/plugins/wordpress/
wordpress/wp-includes/js/tinymce/plugins/wordpress/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/wordpress/plugin.js
wordpress/wp-includes/js/tinymce/plugins/wpautoresize/
wordpress/wp-includes/js/tinymce/plugins/wpautoresize/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/wpautoresize/plugin.js
wordpress/wp-includes/js/tinymce/plugins/paste/
wordpress/wp-includes/js/tinymce/plugins/paste/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/paste/plugin.js
wordpress/wp-includes/js/tinymce/plugins/colorpicker/
wordpress/wp-includes/js/tinymce/plugins/colorpicker/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/colorpicker/plugin.js
wordpress/wp-includes/js/tinymce/plugins/hr/
wordpress/wp-includes/js/tinymce/plugins/hr/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/hr/plugin.js
wordpress/wp-includes/js/tinymce/plugins/lists/
wordpress/wp-includes/js/tinymce/plugins/lists/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/lists/plugin.js
wordpress/wp-includes/js/tinymce/plugins/tabfocus/
wordpress/wp-includes/js/tinymce/plugins/tabfocus/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/tabfocus/plugin.js
wordpress/wp-includes/js/tinymce/plugins/compat3x/
wordpress/wp-includes/js/tinymce/plugins/compat3x/css/
wordpress/wp-includes/js/tinymce/plugins/compat3x/css/dialog.css
wordpress/wp-includes/js/tinymce/plugins/compat3x/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/compat3x/plugin.js
wordpress/wp-includes/js/tinymce/plugins/wpeditimage/
wordpress/wp-includes/js/tinymce/plugins/wpeditimage/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/wpeditimage/plugin.js
wordpress/wp-includes/js/tinymce/plugins/link/
wordpress/wp-includes/js/tinymce/plugins/link/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/link/plugin.js
wordpress/wp-includes/js/tinymce/plugins/directionality/
wordpress/wp-includes/js/tinymce/plugins/directionality/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/directionality/plugin.js
wordpress/wp-includes/js/tinymce/plugins/textcolor/
wordpress/wp-includes/js/tinymce/plugins/textcolor/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/textcolor/plugin.js
wordpress/wp-includes/js/tinymce/plugins/wpgallery/
wordpress/wp-includes/js/tinymce/plugins/wpgallery/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/wpgallery/plugin.js
wordpress/wp-includes/js/tinymce/plugins/image/
wordpress/wp-includes/js/tinymce/plugins/image/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/image/plugin.js
wordpress/wp-includes/js/tinymce/plugins/charmap/
wordpress/wp-includes/js/tinymce/plugins/charmap/plugin.min.js
wordpress/wp-includes/js/tinymce/plugins/charmap/plugin.js
wordpress/wp-includes/js/tinymce/tinymce.min.js
wordpress/wp-includes/js/tinymce/license.txt
wordpress/wp-includes/js/tinymce/utils/
wordpress/wp-includes/js/tinymce/utils/mctabs.js
wordpress/wp-includes/js/tinymce/utils/validate.js
wordpress/wp-includes/js/tinymce/utils/form_utils.js
wordpress/wp-includes/js/tinymce/utils/editable_selects.js
wordpress/wp-includes/js/tinymce/wp-tinymce.php
wordpress/wp-includes/js/wp-list-revisions.min.js
wordpress/wp-includes/js/imgareaselect/
wordpress/wp-includes/js/imgareaselect/border-anim-v.gif
wordpress/wp-includes/js/imgareaselect/imgareaselect.css
wordpress/wp-includes/js/imgareaselect/jquery.imgareaselect.js
wordpress/wp-includes/js/imgareaselect/jquery.imgareaselect.min.js
wordpress/wp-includes/js/imgareaselect/border-anim-h.gif
wordpress/wp-includes/js/autosave.min.js
wordpress/wp-includes/js/wp-lists.min.js
wordpress/wp-includes/js/wpdialog.js
wordpress/wp-includes/js/wpdialog.min.js
wordpress/wp-includes/js/admin-bar.js
wordpress/wp-includes/js/wp-embed.min.js
wordpress/wp-includes/js/media-models.min.js
wordpress/wp-includes/js/hoverIntent.js
wordpress/wp-includes/js/wp-auth-check.min.js
wordpress/wp-includes/js/colorpicker.js
wordpress/wp-includes/js/heartbeat.min.js
wordpress/wp-includes/js/underscore.js
wordpress/wp-includes/js/customize-selective-refresh.min.js
wordpress/wp-includes/js/api-request.js
wordpress/wp-includes/js/wp-emoji.min.js
wordpress/wp-includes/js/quicktags.min.js
wordpress/wp-includes/js/customize-preview.js
wordpress/wp-includes/js/customize-loader.min.js
wordpress/wp-includes/js/customize-views.js
wordpress/wp-includes/js/wp-util.js
wordpress/wp-includes/js/wp-backbone.min.js
wordpress/wp-includes/js/admin-bar.min.js
wordpress/wp-includes/js/tw-sack.js
wordpress/wp-includes/js/clipboard.min.js
wordpress/wp-includes/js/mce-view.min.js
wordpress/wp-includes/js/json2.min.js
wordpress/wp-includes/js/wp-ajax-response.min.js
wordpress/wp-includes/js/jquery/
wordpress/wp-includes/js/jquery/jquery.ui.touch-punch.js
wordpress/wp-includes/js/jquery/jquery.form.min.js
wordpress/wp-includes/js/jquery/jquery.color.min.js
wordpress/wp-includes/js/jquery/jquery-migrate.js
wordpress/wp-includes/js/jquery/jquery.hotkeys.js
wordpress/wp-includes/js/jquery/jquery-migrate.min.js
wordpress/wp-includes/js/jquery/jquery.masonry.min.js
wordpress/wp-includes/js/jquery/jquery.hotkeys.min.js
wordpress/wp-includes/js/jquery/jquery.table-hotkeys.min.js
wordpress/wp-includes/js/jquery/jquery.serialize-object.js
wordpress/wp-includes/js/jquery/suggest.min.js
wordpress/wp-includes/js/jquery/jquery.js
wordpress/wp-includes/js/jquery/suggest.js
wordpress/wp-includes/js/jquery/jquery.min.js
wordpress/wp-includes/js/jquery/ui/
wordpress/wp-includes/js/jquery/ui/effect-fade.js
wordpress/wp-includes/js/jquery/ui/controlgroup.min.js
wordpress/wp-includes/js/jquery/ui/effect-highlight.min.js
wordpress/wp-includes/js/jquery/ui/effect-clip.js
wordpress/wp-includes/js/jquery/ui/effect-highlight.js
wordpress/wp-includes/js/jquery/ui/button.js
wordpress/wp-includes/js/jquery/ui/tabs.min.js
wordpress/wp-includes/js/jquery/ui/selectable.min.js
wordpress/wp-includes/js/jquery/ui/draggable.js
wordpress/wp-includes/js/jquery/ui/droppable.min.js
wordpress/wp-includes/js/jquery/ui/effect.js
wordpress/wp-includes/js/jquery/ui/spinner.js
wordpress/wp-includes/js/jquery/ui/effect-explode.min.js
wordpress/wp-includes/js/jquery/ui/accordion.js
wordpress/wp-includes/js/jquery/ui/core.min.js
wordpress/wp-includes/js/jquery/ui/effect-pulsate.min.js
wordpress/wp-includes/js/jquery/ui/mouse.min.js
wordpress/wp-includes/js/jquery/ui/effect-puff.min.js
wordpress/wp-includes/js/jquery/ui/effect-shake.js
wordpress/wp-includes/js/jquery/ui/menu.js
wordpress/wp-includes/js/jquery/ui/draggable.min.js
wordpress/wp-includes/js/jquery/ui/selectmenu.js
wordpress/wp-includes/js/jquery/ui/sortable.js
wordpress/wp-includes/js/jquery/ui/core.js
wordpress/wp-includes/js/jquery/ui/effect-slide.min.js
wordpress/wp-includes/js/jquery/ui/effect-drop.js
wordpress/wp-includes/js/jquery/ui/effect-size.js
wordpress/wp-includes/js/jquery/ui/autocomplete.js
wordpress/wp-includes/js/jquery/ui/menu.min.js
wordpress/wp-includes/js/jquery/ui/tabs.js
wordpress/wp-includes/js/jquery/ui/effect-bounce.min.js
wordpress/wp-includes/js/jquery/ui/effect-drop.min.js
wordpress/wp-includes/js/jquery/ui/selectable.js
wordpress/wp-includes/js/jquery/ui/dialog.js
wordpress/wp-includes/js/jquery/ui/effect-fold.min.js
wordpress/wp-includes/js/jquery/ui/checkboxradio.min.js
wordpress/wp-includes/js/jquery/ui/effect-puff.js
wordpress/wp-includes/js/jquery/ui/autocomplete.min.js
wordpress/wp-includes/js/jquery/ui/slider.js
wordpress/wp-includes/js/jquery/ui/tooltip.min.js
wordpress/wp-includes/js/jquery/ui/sortable.min.js
wordpress/wp-includes/js/jquery/ui/droppable.js
wordpress/wp-includes/js/jquery/ui/effect-blind.min.js
wordpress/wp-includes/js/jquery/ui/effect-pulsate.js
wordpress/wp-includes/js/jquery/ui/resizable.min.js
wordpress/wp-includes/js/jquery/ui/datepicker.js
wordpress/wp-includes/js/jquery/ui/effect-scale.js
wordpress/wp-includes/js/jquery/ui/checkboxradio.js
wordpress/wp-includes/js/jquery/ui/spinner.min.js
wordpress/wp-includes/js/jquery/ui/button.min.js
wordpress/wp-includes/js/jquery/ui/progressbar.min.js
wordpress/wp-includes/js/jquery/ui/tooltip.js
wordpress/wp-includes/js/jquery/ui/effect.min.js
wordpress/wp-includes/js/jquery/ui/effect-transfer.js
wordpress/wp-includes/js/jquery/ui/accordion.min.js
wordpress/wp-includes/js/jquery/ui/effect-fade.min.js
wordpress/wp-includes/js/jquery/ui/effect-fold.js
wordpress/wp-includes/js/jquery/ui/effect-transfer.min.js
wordpress/wp-includes/js/jquery/ui/mouse.js
wordpress/wp-includes/js/jquery/ui/controlgroup.js
wordpress/wp-includes/js/jquery/ui/progressbar.js
wordpress/wp-includes/js/jquery/ui/effect-clip.min.js
wordpress/wp-includes/js/jquery/ui/effect-scale.min.js
wordpress/wp-includes/js/jquery/ui/selectmenu.min.js
wordpress/wp-includes/js/jquery/ui/effect-blind.js
wordpress/wp-includes/js/jquery/ui/effect-explode.js
wordpress/wp-includes/js/jquery/ui/resizable.js
wordpress/wp-includes/js/jquery/ui/dialog.min.js
wordpress/wp-includes/js/jquery/ui/effect-slide.js
wordpress/wp-includes/js/jquery/ui/slider.min.js
wordpress/wp-includes/js/jquery/ui/effect-bounce.js
wordpress/wp-includes/js/jquery/ui/effect-size.min.js
wordpress/wp-includes/js/jquery/ui/datepicker.min.js
wordpress/wp-includes/js/jquery/ui/effect-shake.min.js
wordpress/wp-includes/js/jquery/jquery.table-hotkeys.js
wordpress/wp-includes/js/jquery/jquery.form.js
wordpress/wp-includes/js/jquery/jquery.schedule.js
wordpress/wp-includes/js/jquery/jquery.query.js
wordpress/wp-includes/js/crop/
wordpress/wp-includes/js/crop/cropper.js
wordpress/wp-includes/js/crop/cropper.css
wordpress/wp-includes/js/crop/marqueeVert.gif
wordpress/wp-includes/js/crop/marqueeHoriz.gif
wordpress/wp-includes/js/twemoji.min.js
wordpress/wp-includes/js/twemoji.js
wordpress/wp-includes/js/thickbox/
wordpress/wp-includes/js/thickbox/thickbox.js
wordpress/wp-includes/js/thickbox/macFFBgHack.png
wordpress/wp-includes/js/thickbox/thickbox.css
wordpress/wp-includes/js/thickbox/loadingAnimation.gif
wordpress/wp-includes/js/zxcvbn.min.js
wordpress/wp-includes/js/shortcode.min.js
wordpress/wp-includes/js/zxcvbn-async.min.js
wordpress/wp-includes/js/imagesloaded.min.js
wordpress/wp-includes/js/customize-base.js
wordpress/wp-includes/js/clipboard.js
wordpress/wp-includes/js/mce-view.js
wordpress/wp-includes/js/customize-models.min.js
wordpress/wp-includes/js/mediaelement/
wordpress/wp-includes/js/mediaelement/mediaelementplayer-legacy.css
wordpress/wp-includes/js/mediaelement/mediaelement-migrate.min.js
wordpress/wp-includes/js/mediaelement/mediaelementplayer-legacy.min.css
wordpress/wp-includes/js/mediaelement/wp-mediaelement.css
wordpress/wp-includes/js/mediaelement/mediaelement-migrate.js
wordpress/wp-includes/js/mediaelement/mediaelement-and-player.js
wordpress/wp-includes/js/mediaelement/wp-playlist.min.js
wordpress/wp-includes/js/mediaelement/mediaelement.min.js
wordpress/wp-includes/js/mediaelement/wp-mediaelement.js
wordpress/wp-includes/js/mediaelement/wp-mediaelement.min.css
wordpress/wp-includes/js/mediaelement/mediaelementplayer.css
wordpress/wp-includes/js/mediaelement/wp-mediaelement.min.js
wordpress/wp-includes/js/mediaelement/mediaelement.js
wordpress/wp-includes/js/mediaelement/renderers/
wordpress/wp-includes/js/mediaelement/renderers/vimeo.min.js
wordpress/wp-includes/js/mediaelement/renderers/vimeo.js
wordpress/wp-includes/js/mediaelement/mejs-controls.png
wordpress/wp-includes/js/mediaelement/mediaelementplayer.min.css
wordpress/wp-includes/js/mediaelement/wp-playlist.js
wordpress/wp-includes/js/mediaelement/mediaelement-and-player.min.js
wordpress/wp-includes/js/mediaelement/mejs-controls.svg
wordpress/wp-includes/js/wp-list-revisions.js
wordpress/wp-includes/js/wp-pointer.min.js
wordpress/wp-includes/js/shortcode.js
wordpress/wp-includes/js/customize-base.min.js
wordpress/wp-includes/js/quicktags.js
wordpress/wp-includes/js/wp-custom-header.js
wordpress/wp-includes/js/media-audiovideo.min.js
wordpress/wp-includes/js/wp-lists.js
wordpress/wp-includes/js/wp-emoji-loader.min.js
wordpress/wp-includes/js/media-views.min.js
wordpress/wp-includes/js/wp-ajax-response.js
wordpress/wp-includes/js/wp-api.min.js
wordpress/wp-includes/js/wplink.min.js
wordpress/wp-includes/js/swfupload/
wordpress/wp-includes/js/swfupload/handlers.js
wordpress/wp-includes/js/swfupload/handlers.min.js
wordpress/wp-includes/js/swfupload/license.txt
wordpress/wp-includes/js/swfupload/swfupload.js
wordpress/wp-includes/js/wp-pointer.js
wordpress/wp-includes/js/backbone.min.js
wordpress/wp-includes/js/codemirror/
wordpress/wp-includes/js/codemirror/fakejshint.js
wordpress/wp-includes/js/codemirror/htmlhint-kses.js
wordpress/wp-includes/js/codemirror/csslint.js
wordpress/wp-includes/js/codemirror/esprima.js
wordpress/wp-includes/js/codemirror/htmlhint.js
wordpress/wp-includes/js/codemirror/jsonlint.js
wordpress/wp-includes/js/codemirror/codemirror.min.js
wordpress/wp-includes/js/codemirror/codemirror.min.css
wordpress/wp-includes/js/masonry.min.js
wordpress/wp-includes/js/wp-emoji-release.min.js
wordpress/wp-includes/js/media-editor.min.js
wordpress/wp-includes/js/wp-auth-check.js
wordpress/wp-includes/js/plupload/
wordpress/wp-includes/js/plupload/plupload.min.js
wordpress/wp-includes/js/plupload/handlers.js
wordpress/wp-includes/js/plupload/moxie.js
wordpress/wp-includes/js/plupload/plupload.js
wordpress/wp-includes/js/plupload/handlers.min.js
wordpress/wp-includes/js/plupload/wp-plupload.js
wordpress/wp-includes/js/plupload/license.txt
wordpress/wp-includes/js/plupload/wp-plupload.min.js
wordpress/wp-includes/js/plupload/moxie.min.js
wordpress/wp-includes/js/jcrop/
wordpress/wp-includes/js/jcrop/jquery.Jcrop.min.css
wordpress/wp-includes/js/jcrop/jquery.Jcrop.min.js
wordpress/wp-includes/js/jcrop/Jcrop.gif
wordpress/wp-includes/js/wp-custom-header.min.js
wordpress/wp-includes/js/api-request.min.js
wordpress/wp-includes/bookmark-template.php
wordpress/wp-includes/widgets.php
wordpress/wp-includes/class-wp-embed.php
wordpress/wp-includes/feed-atom.php
wordpress/wp-includes/class-wp-image-editor-imagick.php
wordpress/wp-includes/class-IXR.php
wordpress/wp-includes/style-engine/
wordpress/wp-includes/style-engine/class-wp-style-engine-processor.php
wordpress/wp-includes/style-engine/class-wp-style-engine.php
wordpress/wp-includes/style-engine/class-wp-style-engine-css-declarations.php
wordpress/wp-includes/style-engine/class-wp-style-engine-css-rules-store.php
wordpress/wp-includes/style-engine/class-wp-style-engine-css-rule.php
wordpress/wp-includes/wp-db.php
wordpress/wp-includes/nav-menu-template.php
wordpress/wp-includes/formatting.php
wordpress/wp-includes/class-wp-http-streams.php
wordpress/wp-includes/class-wp-metadata-lazyloader.php
wordpress/wp-includes/post-template.php
wordpress/wp-includes/feed-atom-comments.php
wordpress/wp-includes/pluggable-deprecated.php
wordpress/wp-includes/theme.php
wordpress/wp-includes/template.php
wordpress/wp-includes/class-wp-text-diff-renderer-table.php
wordpress/wp-includes/class-wp-session-tokens.php
wordpress/wp-includes/class-wp-block-type-registry.php
wordpress/wp-includes/wp-diff.php
wordpress/wp-includes/option.php
wordpress/wp-includes/class-wp-locale-switcher.php
wordpress/wp-includes/post-formats.php
wordpress/wp-includes/template-canvas.php
wordpress/wp-includes/functions.wp-scripts.php
wordpress/wp-includes/class-phpmailer.php
wordpress/wp-includes/class-wp-xmlrpc-server.php
wordpress/wp-includes/class-wp-http-encoding.php
wordpress/wp-includes/theme-previews.php
wordpress/wp-includes/ms-default-constants.php
wordpress/wp-activate.php
wordpress/wp-admin/
wordpress/wp-admin/edit-tag-form.php
wordpress/wp-admin/images/
wordpress/wp-admin/images/post-formats32-vs.png
wordpress/wp-admin/images/arrows-2x.png
wordpress/wp-admin/images/resize-rtl.gif
wordpress/wp-admin/images/contribute-no-code.svg
wordpress/wp-admin/images/date-button.gif
wordpress/wp-admin/images/browser-rtl.png
wordpress/wp-admin/images/wordpress-logo.png
wordpress/wp-admin/images/stars-2x.png
wordpress/wp-admin/images/browser.png
wordpress/wp-admin/images/post-formats32.png
wordpress/wp-admin/images/post-formats.png
wordpress/wp-admin/images/bubble_bg.gif
wordpress/wp-admin/images/icons32-vs-2x.png
wordpress/wp-admin/images/dashboard-background.svg
wordpress/wp-admin/images/spinner.gif
wordpress/wp-admin/images/comment-grey-bubble.png
wordpress/wp-admin/images/contribute-main.svg
wordpress/wp-admin/images/about-texture.png
wordpress/wp-admin/images/imgedit-icons-2x.png
wordpress/wp-admin/images/arrows.png
wordpress/wp-admin/images/align-center-2x.png
wordpress/wp-admin/images/list-2x.png
wordpress/wp-admin/images/wpspin_light-2x.gif
wordpress/wp-admin/images/xit-2x.gif
wordpress/wp-admin/images/generic.png
wordpress/wp-admin/images/menu-vs-2x.png
wordpress/wp-admin/images/icons32.png
wordpress/wp-admin/images/w-logo-blue.png
wordpress/wp-admin/images/media-button.png
wordpress/wp-admin/images/spinner-2x.gif
wordpress/wp-admin/images/about-release-badge.svg
wordpress/wp-admin/images/menu.png
wordpress/wp-admin/images/align-none.png
wordpress/wp-admin/images/icons32-2x.png
wordpress/wp-admin/images/marker.png
wordpress/wp-admin/images/resize.gif
wordpress/wp-admin/images/sort-2x.gif
wordpress/wp-admin/images/align-center.png
wordpress/wp-admin/images/freedom-3.svg
wordpress/wp-admin/images/loading.gif
wordpress/wp-admin/images/comment-grey-bubble-2x.png
wordpress/wp-admin/images/wordpress-logo.svg
wordpress/wp-admin/images/bubble_bg-2x.gif
wordpress/wp-admin/images/align-right.png
wordpress/wp-admin/images/freedom-4.svg
wordpress/wp-admin/images/wordpress-logo-white.svg
wordpress/wp-admin/images/media-button-image.gif
wordpress/wp-admin/images/imgedit-icons.png
wordpress/wp-admin/images/stars.png
wordpress/wp-admin/images/wpspin_light.gif
wordpress/wp-admin/images/align-none-2x.png
wordpress/wp-admin/images/xit.gif
wordpress/wp-admin/images/sort.gif
wordpress/wp-admin/images/list.png
wordpress/wp-admin/images/align-left.png
wordpress/wp-admin/images/w-logo-white.png
wordpress/wp-admin/images/contribute-code.svg
wordpress/wp-admin/images/freedom-2.svg
wordpress/wp-admin/images/align-right-2x.png
wordpress/wp-admin/images/media-button-other.gif
wordpress/wp-admin/images/se.png
wordpress/wp-admin/images/date-button-2x.gif
wordpress/wp-admin/images/resize-2x.gif
wordpress/wp-admin/images/icons32-vs.png
wordpress/wp-admin/images/align-left-2x.png
wordpress/wp-admin/images/media-button-2x.png
wordpress/wp-admin/images/yes.png
wordpress/wp-admin/images/freedom-1.svg
wordpress/wp-admin/images/resize-rtl-2x.gif
wordpress/wp-admin/images/media-button-video.gif
wordpress/wp-admin/images/privacy.svg
wordpress/wp-admin/images/mask.png
wordpress/wp-admin/images/post-formats-vs.png
wordpress/wp-admin/images/no.png
wordpress/wp-admin/images/wheel.png
wordpress/wp-admin/images/menu-vs.png
wordpress/wp-admin/images/menu-2x.png
wordpress/wp-admin/images/media-button-music.gif
wordpress/wp-admin/edit-form-comment.php
wordpress/wp-admin/ms-sites.php
wordpress/wp-admin/custom-header.php
wordpress/wp-admin/contribute.php
wordpress/wp-admin/custom-background.php
wordpress/wp-admin/customize.php
wordpress/wp-admin/admin-functions.php
wordpress/wp-admin/css/
wordpress/wp-admin/css/code-editor.css
wordpress/wp-admin/css/about.css
wordpress/wp-admin/css/customize-nav-menus-rtl.css
wordpress/wp-admin/css/wp-admin-rtl.min.css
wordpress/wp-admin/css/media-rtl.css
wordpress/wp-admin/css/list-tables-rtl.css
wordpress/wp-admin/css/site-health-rtl.css
wordpress/wp-admin/css/color-picker.css
wordpress/wp-admin/css/list-tables-rtl.min.css
wordpress/wp-admin/css/nav-menus.min.css
wordpress/wp-admin/css/themes.css
wordpress/wp-admin/css/login-rtl.min.css
wordpress/wp-admin/css/login-rtl.css
wordpress/wp-admin/css/revisions.css
wordpress/wp-admin/css/forms.css
wordpress/wp-admin/css/common-rtl.min.css
wordpress/wp-admin/css/deprecated-media.css
wordpress/wp-admin/css/site-health.css
wordpress/wp-admin/css/customize-widgets.css
wordpress/wp-admin/css/media-rtl.min.css
wordpress/wp-admin/css/list-tables.css
wordpress/wp-admin/css/edit.css
wordpress/wp-admin/css/nav-menus-rtl.min.css
wordpress/wp-admin/css/edit-rtl.min.css
wordpress/wp-admin/css/site-icon-rtl.min.css
wordpress/wp-admin/css/revisions-rtl.css
wordpress/wp-admin/css/wp-admin.min.css
wordpress/wp-admin/css/site-health-rtl.min.css
wordpress/wp-admin/css/media.min.css
wordpress/wp-admin/css/install.css
wordpress/wp-admin/css/widgets.min.css
wordpress/wp-admin/css/customize-controls.css
wordpress/wp-admin/css/customize-controls-rtl.css
wordpress/wp-admin/css/themes-rtl.css
wordpress/wp-admin/css/about-rtl.min.css
wordpress/wp-admin/css/common.css
wordpress/wp-admin/css/edit.min.css
wordpress/wp-admin/css/color-picker-rtl.min.css
wordpress/wp-admin/css/code-editor-rtl.min.css
wordpress/wp-admin/css/site-icon.css
wordpress/wp-admin/css/site-health.min.css
wordpress/wp-admin/css/nav-menus.css
wordpress/wp-admin/css/code-editor-rtl.css
wordpress/wp-admin/css/customize-nav-menus.min.css
wordpress/wp-admin/css/login.css
wordpress/wp-admin/css/deprecated-media-rtl.css
wordpress/wp-admin/css/admin-menu.css
wordpress/wp-admin/css/dashboard-rtl.css
wordpress/wp-admin/css/deprecated-media-rtl.min.css
wordpress/wp-admin/css/dashboard-rtl.min.css
wordpress/wp-admin/css/farbtastic-rtl.css
wordpress/wp-admin/css/revisions-rtl.min.css
wordpress/wp-admin/css/customize-nav-menus-rtl.min.css
wordpress/wp-admin/css/colors/
wordpress/wp-admin/css/colors/sunrise/
wordpress/wp-admin/css/colors/sunrise/colors.scss
wordpress/wp-admin/css/colors/sunrise/colors-rtl.min.css
wordpress/wp-admin/css/colors/sunrise/colors-rtl.css
wordpress/wp-admin/css/colors/sunrise/colors.min.css
wordpress/wp-admin/css/colors/sunrise/colors.css
wordpress/wp-admin/css/colors/ocean/
wordpress/wp-admin/css/colors/ocean/colors.scss
wordpress/wp-admin/css/colors/ocean/colors-rtl.min.css
wordpress/wp-admin/css/colors/ocean/colors-rtl.css
wordpress/wp-admin/css/colors/ocean/colors.min.css
wordpress/wp-admin/css/colors/ocean/colors.css
wordpress/wp-admin/css/colors/ectoplasm/
wordpress/wp-admin/css/colors/ectoplasm/colors.scss
wordpress/wp-admin/css/colors/ectoplasm/colors-rtl.min.css
wordpress/wp-admin/css/colors/ectoplasm/colors-rtl.css
wordpress/wp-admin/css/colors/ectoplasm/colors.min.css
wordpress/wp-admin/css/colors/ectoplasm/colors.css
wordpress/wp-admin/css/colors/_admin.scss
wordpress/wp-admin/css/colors/blue/
wordpress/wp-admin/css/colors/blue/colors.scss
wordpress/wp-admin/css/colors/blue/colors-rtl.min.css
wordpress/wp-admin/css/colors/blue/colors-rtl.css
wordpress/wp-admin/css/colors/blue/colors.min.css
wordpress/wp-admin/css/colors/blue/colors.css
wordpress/wp-admin/css/colors/light/
wordpress/wp-admin/css/colors/light/colors.scss
wordpress/wp-admin/css/colors/light/colors-rtl.min.css
wordpress/wp-admin/css/colors/light/colors-rtl.css
wordpress/wp-admin/css/colors/light/colors.min.css
wordpress/wp-admin/css/colors/light/colors.css
wordpress/wp-admin/css/colors/_mixins.scss
wordpress/wp-admin/css/colors/_variables.scss
wordpress/wp-admin/css/colors/midnight/
wordpress/wp-admin/css/colors/midnight/colors.scss
wordpress/wp-admin/css/colors/midnight/colors-rtl.min.css
wordpress/wp-admin/css/colors/midnight/colors-rtl.css
wordpress/wp-admin/css/colors/midnight/colors.min.css
wordpress/wp-admin/css/colors/midnight/colors.css
wordpress/wp-admin/css/colors/coffee/
wordpress/wp-admin/css/colors/coffee/colors.scss
wordpress/wp-admin/css/colors/coffee/colors-rtl.min.css
wordpress/wp-admin/css/colors/coffee/colors-rtl.css
wordpress/wp-admin/css/colors/coffee/colors.min.css
wordpress/wp-admin/css/colors/coffee/colors.css
wordpress/wp-admin/css/colors/modern/
wordpress/wp-admin/css/colors/modern/colors.scss
wordpress/wp-admin/css/colors/modern/colors-rtl.min.css
wordpress/wp-admin/css/colors/modern/colors-rtl.css
wordpress/wp-admin/css/colors/modern/colors.min.css
wordpress/wp-admin/css/colors/modern/colors.css
wordpress/wp-admin/css/install-rtl.css
wordpress/wp-admin/css/deprecated-media.min.css
wordpress/wp-admin/css/install.min.css
wordpress/wp-admin/css/themes.min.css
wordpress/wp-admin/css/site-icon.min.css
wordpress/wp-admin/css/edit-rtl.css
wordpress/wp-admin/css/color-picker-rtl.css
wordpress/wp-admin/css/customize-controls.min.css
wordpress/wp-admin/css/l10n.css
wordpress/wp-admin/css/farbtastic.min.css
wordpress/wp-admin/css/admin-menu-rtl.min.css
wordpress/wp-admin/css/dashboard.min.css
wordpress/wp-admin/css/media.css
wordpress/wp-admin/css/widgets-rtl.css
wordpress/wp-admin/css/customize-nav-menus.css
wordpress/wp-admin/css/install-rtl.min.css
wordpress/wp-admin/css/l10n.min.css
wordpress/wp-admin/css/nav-menus-rtl.css
wordpress/wp-admin/css/farbtastic-rtl.min.css
wordpress/wp-admin/css/admin-menu-rtl.css
wordpress/wp-admin/css/farbtastic.css
wordpress/wp-admin/css/code-editor.min.css
wordpress/wp-admin/css/customize-widgets.min.css
wordpress/wp-admin/css/wp-admin.css
wordpress/wp-admin/css/admin-menu.min.css
wordpress/wp-admin/css/common.min.css
wordpress/wp-admin/css/customize-widgets-rtl.css
wordpress/wp-admin/css/site-icon-rtl.css
wordpress/wp-admin/css/about-rtl.css
wordpress/wp-admin/css/about.min.css
wordpress/wp-admin/css/list-tables.min.css
wordpress/wp-admin/css/widgets.css
wordpress/wp-admin/css/revisions.min.css
wordpress/wp-admin/css/l10n-rtl.min.css
wordpress/wp-admin/css/dashboard.css
wordpress/wp-admin/css/widgets-rtl.min.css
wordpress/wp-admin/css/forms.min.css
wordpress/wp-admin/css/common-rtl.css
wordpress/wp-admin/css/forms-rtl.min.css
wordpress/wp-admin/css/login.min.css
wordpress/wp-admin/css/themes-rtl.min.css
wordpress/wp-admin/css/l10n-rtl.css
wordpress/wp-admin/css/customize-controls-rtl.min.css
wordpress/wp-admin/css/forms-rtl.css
wordpress/wp-admin/css/wp-admin-rtl.css
wordpress/wp-admin/css/customize-widgets-rtl.min.css
wordpress/wp-admin/css/color-picker.min.css
wordpress/wp-admin/term.php
wordpress/wp-admin/async-upload.php
wordpress/wp-admin/about.php
wordpress/wp-admin/upgrade-functions.php
wordpress/wp-admin/edit-tags.php
wordpress/wp-admin/my-sites.php
wordpress/wp-admin/export-personal-data.php
wordpress/wp-admin/profile.php
wordpress/wp-admin/site-health-info.php
wordpress/wp-admin/upload.php
wordpress/wp-admin/options-media.php
wordpress/wp-admin/upgrade.php
wordpress/wp-admin/media.php
wordpress/wp-admin/update-core.php
wordpress/wp-admin/edit.php
wordpress/wp-admin/widgets-form-blocks.php
wordpress/wp-admin/media-new.php
wordpress/wp-admin/menu.php
wordpress/wp-admin/ms-edit.php
wordpress/wp-admin/users.php
wordpress/wp-admin/edit-comments.php
wordpress/wp-admin/theme-editor.php
wordpress/wp-admin/options-general.php
wordpress/wp-admin/site-editor.php
wordpress/wp-admin/media-upload.php
wordpress/wp-admin/widgets-form.php
wordpress/wp-admin/link-parse-opml.php
wordpress/wp-admin/privacy-policy-guide.php
wordpress/wp-admin/revision.php
wordpress/wp-admin/options-privacy.php
wordpress/wp-admin/ms-options.php
wordpress/wp-admin/moderation.php
wordpress/wp-admin/admin-post.php
wordpress/wp-admin/nav-menus.php
wordpress/wp-admin/ms-users.php
wordpress/wp-admin/plugin-install.php
wordpress/wp-admin/tools.php
wordpress/wp-admin/network.php
wordpress/wp-admin/load-styles.php
wordpress/wp-admin/privacy.php
wordpress/wp-admin/erase-personal-data.php
wordpress/wp-admin/export.php
wordpress/wp-admin/link-manager.php
wordpress/wp-admin/ms-delete-site.php
wordpress/wp-admin/load-scripts.php
wordpress/wp-admin/comment.php
wordpress/wp-admin/user/
wordpress/wp-admin/user/about.php
wordpress/wp-admin/user/profile.php
wordpress/wp-admin/user/menu.php
wordpress/wp-admin/user/privacy.php
wordpress/wp-admin/user/index.php
wordpress/wp-admin/user/credits.php
wordpress/wp-admin/user/user-edit.php
wordpress/wp-admin/user/admin.php
wordpress/wp-admin/user/freedoms.php
wordpress/wp-admin/plugin-editor.php
wordpress/wp-admin/options-writing.php
wordpress/wp-admin/site-health.php
wordpress/wp-admin/options.php
wordpress/wp-admin/index.php
wordpress/wp-admin/update.php
wordpress/wp-admin/includes/
wordpress/wp-admin/includes/admin-filters.php
wordpress/wp-admin/includes/class-wp-community-events.php
wordpress/wp-admin/includes/class-theme-installer-skin.php
wordpress/wp-admin/includes/class-pclzip.php
wordpress/wp-admin/includes/class-ftp-pure.php
wordpress/wp-admin/includes/edit-tag-messages.php
wordpress/wp-admin/includes/class-custom-image-header.php
wordpress/wp-admin/includes/class-plugin-upgrader.php
wordpress/wp-admin/includes/class-wp-themes-list-table.php
wordpress/wp-admin/includes/class-wp-posts-list-table.php
wordpress/wp-admin/includes/class-walker-nav-menu-edit.php
wordpress/wp-admin/includes/class-wp-upgrader-skins.php
wordpress/wp-admin/includes/class-wp-plugins-list-table.php
wordpress/wp-admin/includes/ms-admin-filters.php
wordpress/wp-admin/includes/class-wp-internal-pointers.php
wordpress/wp-admin/includes/upgrade.php
wordpress/wp-admin/includes/media.php
wordpress/wp-admin/includes/class-wp-screen.php
wordpress/wp-admin/includes/class-walker-nav-menu-checklist.php
wordpress/wp-admin/includes/class-wp-site-health-auto-updates.php
wordpress/wp-admin/includes/meta-boxes.php
wordpress/wp-admin/includes/image.php
wordpress/wp-admin/includes/noop.php
wordpress/wp-admin/includes/update-core.php
wordpress/wp-admin/includes/class-wp-ajax-upgrader-skin.php
wordpress/wp-admin/includes/menu.php
wordpress/wp-admin/includes/class-wp-plugin-install-list-table.php
wordpress/wp-admin/includes/image-edit.php
wordpress/wp-admin/includes/ms-deprecated.php
wordpress/wp-admin/includes/class-language-pack-upgrader-skin.php
wordpress/wp-admin/includes/class-custom-background.php
wordpress/wp-admin/includes/taxonomy.php
wordpress/wp-admin/includes/class-wp-privacy-requests-table.php
wordpress/wp-admin/includes/nav-menu.php
wordpress/wp-admin/includes/class-ftp.php
wordpress/wp-admin/includes/class-file-upload-upgrader.php
wordpress/wp-admin/includes/user.php
wordpress/wp-admin/includes/ajax-actions.php
wordpress/wp-admin/includes/class-wp-post-comments-list-table.php
wordpress/wp-admin/includes/class-plugin-installer-skin.php
wordpress/wp-admin/includes/class-wp-filesystem-direct.php
wordpress/wp-admin/includes/misc.php
wordpress/wp-admin/includes/class-wp-ms-sites-list-table.php
wordpress/wp-admin/includes/revision.php
wordpress/wp-admin/includes/class-wp-privacy-data-removal-requests-list-table.php
wordpress/wp-admin/includes/continents-cities.php
wordpress/wp-admin/includes/class-theme-upgrader.php
wordpress/wp-admin/includes/class-wp-site-health.php
wordpress/wp-admin/includes/class-wp-list-table.php
wordpress/wp-admin/includes/plugin-install.php
wordpress/wp-admin/includes/class-wp-site-icon.php
wordpress/wp-admin/includes/network.php
wordpress/wp-admin/includes/class-wp-importer.php
wordpress/wp-admin/includes/class-wp-application-passwords-list-table.php
wordpress/wp-admin/includes/deprecated.php
wordpress/wp-admin/includes/class-wp-terms-list-table.php
wordpress/wp-admin/includes/export.php
wordpress/wp-admin/includes/class-wp-filesystem-base.php
wordpress/wp-admin/includes/class-wp-ms-users-list-table.php
wordpress/wp-admin/includes/plugin.php
wordpress/wp-admin/includes/comment.php
wordpress/wp-admin/includes/class-wp-media-list-table.php
wordpress/wp-admin/includes/class-wp-debug-data.php
wordpress/wp-admin/includes/class-wp-comments-list-table.php
wordpress/wp-admin/includes/options.php
wordpress/wp-admin/includes/class-bulk-upgrader-skin.php
wordpress/wp-admin/includes/class-bulk-plugin-upgrader-skin.php
wordpress/wp-admin/includes/update.php
wordpress/wp-admin/includes/class-language-pack-upgrader.php
wordpress/wp-admin/includes/dashboard.php
wordpress/wp-admin/includes/class-theme-upgrader-skin.php
wordpress/wp-admin/includes/theme-install.php
wordpress/wp-admin/includes/schema.php
wordpress/wp-admin/includes/privacy-tools.php
wordpress/wp-admin/includes/class-wp-upgrader-skin.php
wordpress/wp-admin/includes/credits.php
wordpress/wp-admin/includes/class-wp-links-list-table.php
wordpress/wp-admin/includes/class-wp-filesystem-ftpext.php
wordpress/wp-admin/includes/class-bulk-theme-upgrader-skin.php
wordpress/wp-admin/includes/class-wp-list-table-compat.php
wordpress/wp-admin/includes/post.php
wordpress/wp-admin/includes/class-core-upgrader.php
wordpress/wp-admin/includes/import.php
wordpress/wp-admin/includes/class-ftp-sockets.php
wordpress/wp-admin/includes/class-wp-automatic-updater.php
wordpress/wp-admin/includes/list-table.php
wordpress/wp-admin/includes/class-wp-filesystem-ssh2.php
wordpress/wp-admin/includes/class-automatic-upgrader-skin.php
wordpress/wp-admin/includes/screen.php
wordpress/wp-admin/includes/class-wp-privacy-data-export-requests-list-table.php
wordpress/wp-admin/includes/translation-install.php
wordpress/wp-admin/includes/admin.php
wordpress/wp-admin/includes/class-wp-privacy-policy-content.php
wordpress/wp-admin/includes/file.php
wordpress/wp-admin/includes/bookmark.php
wordpress/wp-admin/includes/class-wp-upgrader.php
wordpress/wp-admin/includes/class-plugin-upgrader-skin.php
wordpress/wp-admin/includes/ms.php
wordpress/wp-admin/includes/widgets.php
wordpress/wp-admin/includes/class-wp-ms-themes-list-table.php
wordpress/wp-admin/includes/class-wp-filesystem-ftpsockets.php
wordpress/wp-admin/includes/class-wp-theme-install-list-table.php
wordpress/wp-admin/includes/theme.php
wordpress/wp-admin/includes/template.php
wordpress/wp-admin/includes/class-walker-category-checklist.php
wordpress/wp-admin/includes/class-wp-users-list-table.php
wordpress/wp-admin/menu-header.php
wordpress/wp-admin/options-head.php
wordpress/wp-admin/theme-install.php
wordpress/wp-admin/install-helper.php
wordpress/wp-admin/credits.php
wordpress/wp-admin/maint/
wordpress/wp-admin/maint/repair.php
wordpress/wp-admin/ms-upgrade-network.php
wordpress/wp-admin/ms-themes.php
wordpress/wp-admin/post.php
wordpress/wp-admin/edit-link-form.php
wordpress/wp-admin/edit-form-blocks.php
wordpress/wp-admin/user-edit.php
wordpress/wp-admin/press-this.php
wordpress/wp-admin/authorize-application.php
wordpress/wp-admin/admin-footer.php
wordpress/wp-admin/edit-form-advanced.php
wordpress/wp-admin/import.php
wordpress/wp-admin/ms-admin.php
wordpress/wp-admin/link.php
wordpress/wp-admin/network/
wordpress/wp-admin/network/contribute.php
wordpress/wp-admin/network/site-info.php
wordpress/wp-admin/network/about.php
wordpress/wp-admin/network/profile.php
wordpress/wp-admin/network/upgrade.php
wordpress/wp-admin/network/update-core.php
wordpress/wp-admin/network/edit.php
wordpress/wp-admin/network/sites.php
wordpress/wp-admin/network/menu.php
wordpress/wp-admin/network/users.php
wordpress/wp-admin/network/theme-editor.php
wordpress/wp-admin/network/site-users.php
wordpress/wp-admin/network/plugin-install.php
wordpress/wp-admin/network/privacy.php
wordpress/wp-admin/network/settings.php
wordpress/wp-admin/network/plugin-editor.php
wordpress/wp-admin/network/index.php
wordpress/wp-admin/network/update.php
wordpress/wp-admin/network/theme-install.php
wordpress/wp-admin/network/setup.php
wordpress/wp-admin/network/credits.php
wordpress/wp-admin/network/user-edit.php
wordpress/wp-admin/network/plugins.php
wordpress/wp-admin/network/admin.php
wordpress/wp-admin/network/freedoms.php
wordpress/wp-admin/network/user-new.php
wordpress/wp-admin/network/site-settings.php
wordpress/wp-admin/network/site-themes.php
wordpress/wp-admin/network/site-new.php
wordpress/wp-admin/network/themes.php
wordpress/wp-admin/plugins.php
wordpress/wp-admin/admin.php
wordpress/wp-admin/freedoms.php
wordpress/wp-admin/link-add.php
wordpress/wp-admin/user-new.php
wordpress/wp-admin/options-discussion.php
wordpress/wp-admin/admin-ajax.php
wordpress/wp-admin/js/
wordpress/wp-admin/js/auth-app.min.js
wordpress/wp-admin/js/revisions.min.js
wordpress/wp-admin/js/nav-menu.js
wordpress/wp-admin/js/plugin-install.min.js
wordpress/wp-admin/js/plugin-install.js
wordpress/wp-admin/js/inline-edit-tax.min.js
wordpress/wp-admin/js/inline-edit-tax.js
wordpress/wp-admin/js/widgets.min.js
wordpress/wp-admin/js/gallery.min.js
wordpress/wp-admin/js/tags-suggest.js
wordpress/wp-admin/js/accordion.js
wordpress/wp-admin/js/user-profile.js
wordpress/wp-admin/js/application-passwords.min.js
wordpress/wp-admin/js/media-gallery.js
wordpress/wp-admin/js/password-strength-meter.min.js
wordpress/wp-admin/js/dashboard.js
wordpress/wp-admin/js/iris.min.js
wordpress/wp-admin/js/revisions.js
wordpress/wp-admin/js/comment.min.js
wordpress/wp-admin/js/editor.min.js
wordpress/wp-admin/js/xfn.min.js
wordpress/wp-admin/js/farbtastic.js
wordpress/wp-admin/js/postbox.js
wordpress/wp-admin/js/tags-suggest.min.js
wordpress/wp-admin/js/word-count.min.js
wordpress/wp-admin/js/comment.js
wordpress/wp-admin/js/theme.min.js
wordpress/wp-admin/js/site-health.js
wordpress/wp-admin/js/edit-comments.min.js
wordpress/wp-admin/js/user-suggest.js
wordpress/wp-admin/js/user-suggest.min.js
wordpress/wp-admin/js/tags-box.js
wordpress/wp-admin/js/site-icon.js
wordpress/wp-admin/js/customize-widgets.js
wordpress/wp-admin/js/post.js
wordpress/wp-admin/js/image-edit.min.js
wordpress/wp-admin/js/xfn.js
wordpress/wp-admin/js/updates.min.js
wordpress/wp-admin/js/language-chooser.min.js
wordpress/wp-admin/js/widgets/
wordpress/wp-admin/js/widgets/media-image-widget.js
wordpress/wp-admin/js/widgets/media-video-widget.min.js
wordpress/wp-admin/js/widgets/media-widgets.min.js
wordpress/wp-admin/js/widgets/custom-html-widgets.min.js
wordpress/wp-admin/js/widgets/custom-html-widgets.js
wordpress/wp-admin/js/widgets/media-audio-widget.js
wordpress/wp-admin/js/widgets/media-image-widget.min.js
wordpress/wp-admin/js/widgets/media-gallery-widget.min.js
wordpress/wp-admin/js/widgets/media-audio-widget.min.js
wordpress/wp-admin/js/widgets/media-video-widget.js
wordpress/wp-admin/js/widgets/text-widgets.min.js
wordpress/wp-admin/js/widgets/media-gallery-widget.js
wordpress/wp-admin/js/widgets/media-widgets.js
wordpress/wp-admin/js/widgets/text-widgets.js
wordpress/wp-admin/js/language-chooser.js
wordpress/wp-admin/js/theme-plugin-editor.min.js
wordpress/wp-admin/js/post.min.js
wordpress/wp-admin/js/tags.js
wordpress/wp-admin/js/privacy-tools.min.js
wordpress/wp-admin/js/media-upload.min.js
wordpress/wp-admin/js/privacy-tools.js
wordpress/wp-admin/js/site-health.min.js
wordpress/wp-admin/js/media.min.js
wordpress/wp-admin/js/theme.js
wordpress/wp-admin/js/customize-controls.js
wordpress/wp-admin/js/edit-comments.js
wordpress/wp-admin/js/customize-nav-menus.min.js
wordpress/wp-admin/js/tags.min.js
wordpress/wp-admin/js/dashboard.min.js
wordpress/wp-admin/js/link.min.js
wordpress/wp-admin/js/editor.js
wordpress/wp-admin/js/customize-controls.min.js
wordpress/wp-admin/js/theme-plugin-editor.js
wordpress/wp-admin/js/nav-menu.min.js
wordpress/wp-admin/js/image-edit.js
wordpress/wp-admin/js/custom-background.min.js
wordpress/wp-admin/js/application-passwords.js
wordpress/wp-admin/js/password-toggle.js
wordpress/wp-admin/js/user-profile.min.js
wordpress/wp-admin/js/password-toggle.min.js
wordpress/wp-admin/js/svg-painter.js
wordpress/wp-admin/js/link.js
wordpress/wp-admin/js/custom-header.js
wordpress/wp-admin/js/widgets.js
wordpress/wp-admin/js/gallery.js
wordpress/wp-admin/js/word-count.js
wordpress/wp-admin/js/accordion.min.js
wordpress/wp-admin/js/inline-edit-post.min.js
wordpress/wp-admin/js/customize-widgets.min.js
wordpress/wp-admin/js/inline-edit-post.js
wordpress/wp-admin/js/updates.js
wordpress/wp-admin/js/media-upload.js
wordpress/wp-admin/js/media.js
wordpress/wp-admin/js/editor-expand.min.js
wordpress/wp-admin/js/media-gallery.min.js
wordpress/wp-admin/js/common.min.js
wordpress/wp-admin/js/tags-box.min.js
wordpress/wp-admin/js/svg-painter.min.js
wordpress/wp-admin/js/custom-background.js
wordpress/wp-admin/js/color-picker.min.js
wordpress/wp-admin/js/site-icon.min.js
wordpress/wp-admin/js/auth-app.js
wordpress/wp-admin/js/code-editor.js
wordpress/wp-admin/js/common.js
wordpress/wp-admin/js/set-post-thumbnail.min.js
wordpress/wp-admin/js/postbox.min.js
wordpress/wp-admin/js/color-picker.js
wordpress/wp-admin/js/password-strength-meter.js
wordpress/wp-admin/js/customize-nav-menus.js
wordpress/wp-admin/js/editor-expand.js
wordpress/wp-admin/js/code-editor.min.js
wordpress/wp-admin/js/set-post-thumbnail.js
wordpress/wp-admin/options-permalink.php
wordpress/wp-admin/widgets.php
wordpress/wp-admin/setup-config.php
wordpress/wp-admin/install.php
wordpress/wp-admin/admin-header.php
wordpress/wp-admin/post-new.php
wordpress/wp-admin/themes.php
wordpress/wp-admin/options-reading.php
wordpress/wp-trackback.php
wordpress/wp-comments-post.php
[ec2-user@ip-14-0-1-75 wordpress]$

cd into /var/www/html directory

[ec2-user@ip-14-0-1-75 wordpress]$ sudo wget http://wordpress.org/latest.tar.gz
--2024-05-09 01:26:45--  http://wordpress.org/latest.tar.gz
Resolving wordpress.org (wordpress.org)... 198.143.164.252
Connecting to wordpress.org (wordpress.org)|198.143.164.252|:80... connected.
HTTP request sent, awaiting response... 301 Moved Permanently
Location: https://wordpress.org/latest.tar.gz [following]
--2024-05-09 01:26:45--  https://wordpress.org/latest.tar.gz
Connecting to wordpress.org (wordpress.org)|198.143.164.252|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 24696379 (24M) [application/octet-stream]
Saving to: ‘latest.tar.gz’

latest.tar.gz       100%[===================>]  23.55M  49.0MB/s    in 0.5s    

2024-05-09 01:26:46 (49.0 MB/s) - ‘latest.tar.gz’ saved [24696379/24696379]






