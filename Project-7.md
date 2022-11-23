# Project 7: Devops Tooling Website Solution

In this project one will implement a solution that consists of following components:

Infrastructure: AWS
Webserver Linux: Red Hat Enterprise Linux 8
Database Server: Ubuntu 20.04 + MySQL
Storage Server: Red Hat Enterprise Linux 8 + NFS Server
Programming Language: PHP
Code Repository: GitHub

Step 0:  Preparing prerequisites

- Sign in to AWS free tier account and create four new EC2 Instances of t2.nano family with RedHat OS, and one new EC2 Instance of Ubuntu 20.04
- Name the new instances thus;
- Server A name - "NFS server" (Red Hat Enterprise Linux 8 + NFS Server)
- Server B name - "webserver 1" (Red Hat Enterprise Linux 8)
- Server C name - "webserver 2" (Red Hat Enterprise Linux 8)
- Server D name - "webserver 3" (Red Hat Enterprise Linux 8)
- Server C name - "database server" (Ubuntu 20.04 + MySQL)
  
Step 1:  Creating and adding EBS Volumes to NFS server EC2 instances

- Create 3 volumes , each of 10 GiB NFS1, NFS2 and NFS3) and attach them to the EC2 instances, "NFS server".
- Connect to these instances via SSH on five different windows terminals renamed as;
- NFS server
- webserver-1
- webserver-2
- webserver-3
- database server

Configure LVM on the Server

`lsblk`

![EBS volumes status](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/lsblk_NFS.PNG)

Start partitioning the EBS volumes

`sudo gdisk /dev/xvdf`

![partitioning xvdf](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/xvdf_NFS.PNG)

![partitioning xvdf status](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/xvdf_NFS_confirmed.PNG)

`sudo gdisk /dev/xvdg`

![partitioning xvdg](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/xvdg_NFS.PNG)

![partitioning xvdg status](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/xvdg_NFS_confirmed.PNG)

`sudo gdisk /dev/xvdh`

![partitioning xvdh](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/xvdh_NFS.PNG)

![partitioning xvdh status](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/xvdh_NFS_confirmed.PNG)

Install the lvm2 package

`sudo yum install lvm2`

![lvm2 installation status](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/lvm2_NFS.PNG)

Check for available partitions

`sudo lvmdiskscan`

![search status](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/lvmdiskscan_NFS.PNG)

Create physical volume

`sudo pvcreate`

![pvcreate status ](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/pvcreate_NFS.PNG)

Check the physical volumes

`sudo pvs`

![physical volume search status](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/pvs_NFS.PNG)

Create a volume group

`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

![volume group creation status](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/vgcreate_NFS.PNG)

Check the volume group status

![vg check status](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/vgs_NFS.PNG)

Create the logical volumes

`sudo lvcreate -n apps-lv -L 9G webdata-vg`

`sudo lvcreate -n logs-lv -L 9G webdata-vg`

`sudo lvcreate -n opt-lv -L 9G webdata-vg`

![Logical volumes created](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/lvcreate_NFS.PNG)

Check the logical volumes

`sudo lvs`

![Alt text](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/lvs_nfs.PNG)

`lsblk`

![lsblk status](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/lvs_lsblk_nfs.PNG)

`sudo vgdisplay -v #view complete setup -VG, PV, and LV`

![Alt text](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/gdisplay_nfs.PNG)

Format the disks as as xfs

`sudo mkfs -t xfs /dev/webdata-vg/apps-lv`

`sudo mkfs -t xfs /dev/webdata-vg/logs-lv`

`sudo mkfs -t xfs /dev/webdata-vg/opt-lv`

![disks formatting status](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/xfs_format_nfs.PNG)

Create the mount points

`sudo mkdir /mnt/apps`

`sudo mkdir /mnt/logs`

`sudo mkdir /mnt/opt`

![mount points created](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/mountPoints_nfs.PNG)

Mount the logical volumes

`sudo mount /dev/webdata-vg/apps-lv /mnt/apps`

`sudo mount /dev/webdata-vg/logs-lv /mnt/logs`

`sudo mount /dev/webdata-vg/opt-lv /mnt/opt`

![mount status](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/mountCompleted_nfs.PNG)

Install NFS server, configure it to start on reboot and make sure it is up and running

Update the NFS server

`sudo yum -y update`

![Update NFS server](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/update_NFS_instance.PNG)

`sudo yum install nfs-utils -y`

![install utils](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/install_NFS_utils.PNG)

`sudo systemctl start nfs-server.service`

`sudo systemctl enable nfs-server.service`

`sudo systemctl status nfs-server.service`

![NFS server status](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/NFS_Server_is_running.PNG)

Configure the database server and install mysql-server

`sudo apt update`

![update db server](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/update_db.PNG)

`sudo apt install mysql-server -y`

![install mysql-server](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/install_mysql_db.PNG)

configure a MySQL DBMS

`sudo mysql`

`create database tooling`

`Create user 'webaccess`@'172.31.32.0/20' identified by 'password';`

`grant all privileges on tooling.* to 'webaccess`@'172.31.32.0/20';`

`flush privileges`

![configure a MySQL DBMS](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/create%20db_and_user_db.PNG)

set up permission that will allow our Web servers to read, write and execute files on NFS:

`sudo chown -R nobody: /mnt/apps`

`sudo chown -R nobody: /mnt/logs`

`sudo chown -R nobody: /mnt/opt`

`sudo chmod -R 777 /mnt/apps`

`sudo chmod -R 777 /mnt/logs`

`sudo chmod -R 777 /mnt/opt`

`sudo systemctl restart nfs-server.service`

![permissions changed successfully](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/permissions_changed_server_restarted.PNG)

Configure access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.32.0/20 ):

`sudo vi /etc/exports`

![updating subnet CIDR](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/update_subnet_CIDR_NFS.PNG)

`sudo exportfs -arv`

![exported mount points for webservers accessibility](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/mountPoint_exported_NFS.PNG)

`rpcinfo -p | grep nfs`

![Check the listening port](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/Check_port_used_by_NFS.PNG)

Important note: In order for NFS server to be accessible from your client,  open the following ports: TCP 111, UDP 111, UDP 2049:

![updating inbound rules status](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/updating_inbound_rules_NFS.PNG)

Prepare the Web Servers

Install NFS client on all three web servers

`sudo yum install nfs-utils nfs4-acl-tools -y`

![webserver-1 NFS client installation](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/install_NFS_client_wbs1.PNG)

![webserver-2 NFS client installation](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/install_NFS_client_wbs2.PNG)

![webserver-3 NFS client installation](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/install_NFS_client_wbs3.PNG)

Mount /var/www/ and target the NFS server’s export for apps:

`sudo mkdir /var/www`

`sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid 172.31.32.0/20:/mnt/apps /var/www`

![Mount /var/www/ targeting export for apps on webserver-1](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/mount_for_apps_wbs1.PNG)

![Mount /var/www/ targeting export for logs on webserver-1](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/mount_forr_logs_wbs1.PNG)

Repeat process for web servers 2 and 3:

![Mount /var/www/ targeting export for apps on webserver-2](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/mountPoint_nfs_wbs2.PNG)

![Mount /var/www/ targeting export for apps on webserver-3](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/mountPoint_nfs_wbs3.PNG)

Verify that NFS was mounted successfully by running

`df -h`

![mount confirmed fpr webserver-1](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/mount_confirmed_wbs1.PNG)

![mount confirmed fpr webserver-2](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/df-h_wbd2.PNG)

![mount confirmed fpr webserver-3](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/df-h_wbs3.PNG)

Make sure that the changes will persist on Web Server after reboot:

`sudo vi /etc/fstab`

![open fstab in webserver-1](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/open_and_edit_fstab__logs_wbs1.PNG)

![open fstab in webserver-2](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/open_and_edit_fstab_wbs2.PNG)

![open fstab in webserver-3](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/open_and_edit_fstab__logs_wbs3.PNG)

update fstab in all web servers with the following code:

`<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0`

![updating fstab for webserver-1](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/update_fstab_wbs1.PNG)

![updating fstab for webserver-2](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/update_fstab_wbs2.PNG)

![updating fstab for webserver-3](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/update_fstab_wbs3.PNG)

Install Remi’s repository, Apache and PHP on all web servers

`sudo yum install httpd -y`

![installing Apache on webserver-1](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/installing_Apache_wbs1.PNG)

![installing Apache on webserver-2](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/installing_Apache_wbs2.PNG)

![installing Apache on webserver-3](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/installing_Apache_wbs3.PNG)

`sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

![installation status on wb1](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/install_epel_wbs1.PNG)

![installation status on wb2](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/install_epel_wbs2.PNG)

![installation status on wb3](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/install_epel_wbs3.PNG)

`sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

![install remirepo_wbs1](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/remirepo_wbs1.PNG)

![install remirepo_wbs2](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/remirepo_wbs2.PNG)

![install remirepo_wbs3](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/remirepo_wbs3.PNG)

`sudo dnf module reset php`

![reset php web1](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/php_module_reset_wbs1.PNG)

![reset php web2](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/php_module_reset_wbs2.PNG)

![reset php web3](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/php_module_reset_wbs3.PNG)

`sudo dnf module enable php:remi-7.4`

`sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`

![installation status on wb1](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/php_opache_wbs1.PNG)

![installation status on wb2](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/php_opache_wbs2.PNG)

![installation status on wb3](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/php_opache_wbs3.PNG)

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

`setsebool -P httpd_execmem 1`

![restart and enable php on webserver-1](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/start_and_enable_php.PNG)

![setsebool for webserver-1](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/setsbool_wbs1.PNG)

![restart and enable php and setsebool for webserver-2](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/start_and_enable_php_setsbool_wbs2.PNG)

![restart and enable php and setsebool for webserver-3](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/start_and_enable_php_setsbool_wbs3.PNG)

Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs

`sudo vi /etc/fstab`

![update fstab](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/updating_fstab_for_log_httpd_wbs1.PNG)

`sudo mount -t nfs -o rw, nosuid 172.31.47.64:/mnt/logs /var/log/httpd`

![Locate and mount the log folder for Apache](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/mount_forr_logs_wbs1.PNG)

Fork the tooling source code from Darey.io Github Account to your Github account

`sudo yum install git`

![install git](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/install_git_wbs1.PNG)

`git init`

`git clone https://github.com/darey-io/tooling.git`

![Fork the tooling source code](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/git_init_clone.PNG)

Deploy the tooling website’s code to the Webserver

`cd tooling`

`cp -R html/. /var/www/html`

![deployment status](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/html_folder_deployed_to_target_dir_wbs1.PNG)

disable SELinux

`sudo setenforce 0`

![setenforce](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/restarting_Apache_wbs1.PNG)

`sudo vi /etc/sysconfig/selinux`

![disable selinux](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/Selinux_disabled_wbs1.PNG)

Update the website’s configuration to connect to the database (in /var/www/html/functions.php file):
Apply tooling-db.sql script to your database using this command

`mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql`

![update configuration](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/updating_functions_php_file_wbs1.PNG)

Open Mysql db configuration file

`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`

![Open Mysql config file](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/Open_mysql_cnf_db.PNG)

Edit the binding address in the Mysql db configuration file

![binding address updated](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/update_binding_address_mysql_cnf_db.PNG)

Restart mysql

`sudo systemctl restart mysql`

`sudo systemctl enable mysql`

![Restarting mysql](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/restart_mysql_db.PNG)

Back to the database

![Confirming db connection to webserver](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/webserver_mysql_connection_confirmation1.PNG)

![Confirming db connection to webserver](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/webserver_mysql_connection_confirmation2.PNG)

Open the website in your browser using WebServer Public IP Address

![Login page](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/Login_page.PNG)

![Login Successful](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/tooling_welcome_page.PNG)

![Propitix tooling page](../../../../../../C:/Users/user/Desktop/Tunde-Workspace/Project-7/images/tooling_welcome_page2.PNG)cd
