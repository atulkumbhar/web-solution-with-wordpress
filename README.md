# Web Solution With WordPress
You are progressing in practicing to implement web solutions using different technologies. As a DevOps engineer you will most probably encounter [PHP](https://www.php.net/)-based solutions since, even in 2021, it is the dominant web programming language used by more websites than any other programming language.

In this project you will be tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. [WordPress](https://en.wikipedia.org/wiki/WordPress) is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).

This project consists of two parts:
- Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give you practical experience of working with disks, partitions and volumes in Linux.
- Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify your skills of deploying Web and DB tiers of Web solution.

As a DevOps engineer, your deep understanding of core components of web solutions and ability to troubleshoot them will play essential role in your further progress and development.

Generally, web, or mobile solutions are implemented based on what is called the **Three-tier Architecture**.

**Three-tier Architecture** is a client-server software architecture pattern that comprise of 3 separate layers.

![](./images/six.jpeg)

- **Presentation Layer (PL)**: This is the user interface such as the client server or browser on your laptop.
- **Business Layer** (BL): This is the backend program that implements business logic. Application or Webserver
- **Data Access or Management Layer** (DAL): This is the layer for computer data storage and data access. [Database Server](https://www.computerhope.com/jargon/d/database-server.htm) or File System Server such as [FTP server](https://titanftp.com/2018/09/11/what-is-an-ftp-server/), or [NFS Server](https://searchenterprisedesktop.techtarget.com/definition/Network-File-System)

In this project, you will have the hands-on experience that showcases **Three-tier Architecture** while also ensuring that the disks used to store files on the Linux servers are adequately partitioned and managed through programs such as **gdisk** and **LVM** respectively.

You will be working working with several storage and disk management concepts, to have a better understanding, watch following video: [Disk management in Linux](https://www.youtube.com/playlist?list=PL6IQ3nFZzWfqZ18ip2tnHbbDp6JyMZtSG)

**Note**: We are gradually introducing new AWS elements into our solutions, but do not be worried if you do not fully understand AWS Cloud Services yet, there are Cloud focused projects ahead where we will get into deep details of various Cloud concepts and technologies - not only AWS, but other Cloud Service Providers as well.

### Your 3-Tier Setup
- A Laptop or PC to serve as a client
- An EC2 Linux Server as a web server (This is where you will install Wordpress)
- An EC2 Linux server as a database (DB) server

***Use RedHat OS for this project***

By now you should know how to spin up an EC2 instanse on AWS, but if you forgot - refer to [WEB STACK IMPLEMENTATION](https://github.com/samuelbartels20/web-stack-implementation) Step 0. In previous projects we used ???Ubuntu???, but it is better to be well-versed with various Linux distributions, thus, for this projects we will use very popular distribution called ???RedHat??? (it also has a fully compatible derivative - CentOS)

**Note**: for Ubuntu server, when connecting to it via SSH/Putty or any other tool, we used ubuntu user, but for RedHat you will need to use ec2-user user. Connection string will look like ec2-user@<Public-IP>

Let us get started!

###  Prepare a Web Server
- Launch an EC2 instance that will serve as ???Web Server???. Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.

Learn How to Add EBS Volume to an EC2 instance [here](https://www.youtube.com/watch?v=HPXnXkBzIHw)

![](./images/volume_creation.png)

![](./images/redhat.png)

![](./images/volume.png)

- Attach all three volumes one by one to your Web Server EC2 instance

![](./images/attach2.png)
![](./images/attach.png)

![](./images/volume.png)

- Open up the Linux terminal to begin configuration
![](./images/screen.png)

- Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there - their names will likely be xvdf, xvdh, xvdg.
![](./images/lsblk.png)

- Use **df -h** command to see all mounts and free space on your server
![](./images/df.png)

- Use **gdisk** utility to create a single partition on each of the 3 disks
```
sudo gdisk /dev/xvdf
```

```
 GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help branch segun-edits: p
Disk /dev/xvdf: 20971520 sectors, 10.0 GiB
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): D936A35E-CE80-41A1-B87E-54D2044D160B
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048        20971486   10.0 GiB    8E00  Linux LVM

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): yes
OK; writing new GUID partition table (GPT) to /dev/xvdf.
The operation has completed successfully.
Now,  your changes has been configured succesfuly, exit out of the gdisk console and do the same for the remaining disks.
```

![](./images/gdisk.png)

![](./images/gdisk2.png)

![](./images/gdisk3.png)

- Use lsblk utility to view the newly configured partition on each of the 3 disks.

![](./images/lsblk2.png)

- Install lvm2 package using sudo yum install lvm2. Run sudo lvmdiskscan command to check for available partitions.

![](./images/lvm.png)

**Note**:** Previously, in Ubuntu we used apt command to install packages, in RedHat/CentOS a different package manager is used, so we shall use yum command instead.

- Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```

![](./images/lvm2.png)

Verify that your Physical volume has been created successfully by running **sudo pvs**

![](./images/lvm3.png)

Use **vgcreate** utility to add all 3 PVs to a volume group (VG). Name the VG **webdata-vg**
```
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
```

![](./images/vg.png)

Verify that your VG has been created successfully by running **sudo vgs**

![](./images/vg2.png)

Use **lvcreate** utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.
```
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```

![](./images/lvm4.png)

Verify that your Logical Volume has been created successfully by running **sudo lvs**

![](./images/lvm5.png)

Verify the entire setup
```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk 
```

![](./images/disk.png)

![](./images/disk2.png)

Use mkfs.ext4 to format the logical volumes with [ext4](https://en.wikipedia.org/wiki/Ext4) filesystem
```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```

![](./images/lsl.png)

Create /var/www/html directory to store website files
```
sudo mkdir -p /var/www/html
```

Create /home/recovery/logs to store backup of log data
```
sudo mkdir -p /home/recovery/logs
```

Mount /var/www/html on apps-lv logical volume
```
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
```

Use **rsync** utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
```
sudo rsync -av /var/log/. /home/recovery/logs/
```

![](./images/disk6.png)

Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very important)
```
sudo mount /dev/webdata-vg/logs-lv /var/log
```

Restore log files back into /var/log directory
```
sudo rsync -av /home/recovery/logs/log/. /var/log
```
![](./images/fdg.png)

Update /etc/fstab file so that the mount configuration will persist after restart of the server

The UUID of the device will be used to update the /etc/fstab file;
```
sudo blkid
```
![](./images/blkid.png)

![](./images/blkid2.png)

```
sudo vi /etc/fstab
```

Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.

![](./images/fstab.png)

![](./images/fst.png)

Test the configuration and reload the daemon
```
sudo mount -a
sudo systemctl daemon-reload
```

Verify your setup by running df -h, output must look like this
![](./images/gka.png)

### Prepare the Database Server
Launch a second RedHat EC2 instance that will have a role - ???DB Server??? Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

![](./images/wordpress2.png)

- Create 3 volumes in the same AZ as your DB Server EC2, each of 10 GiB
- Attach all three volumes one by one to your DB Server EC2 instance
![](./images/db-database.png)
- Open up the Linux terminal to begin configuration
- Use **lsblk** command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there - their names will likely be xvdf, xvdh, xvdg.

![](./images/ls3.png)

- Use df -h command to see all mounts and free space on your db server

![](./images/df3.png)

- Use gdisk utility to create a single partition on each of the 3 disks
```
sudo gdisk /dev/xvdf
sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdh
```

```
 GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help branch segun-edits: p
Disk /dev/xvdf: 20971520 sectors, 10.0 GiB
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): D936A35E-CE80-41A1-B87E-54D2044D160B
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048        20971486   10.0 GiB    8E00  Linux LVM

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): yes
OK; writing new GUID partition table (GPT) to /dev/xvdf.
The operation has completed successfully.
Now,  your changes has been configured succesfuly, exit out of the gdisk console and do the same for the remaining disks.
```

![](./images/g20.png)

![](./images/g21.png)

![](./images/g22.png)

- Use lsblk utility to view the newly configured partition on each of the 3 disks.

![](./images/g24.png)

- Install lvm2 package using **sudo yum install lvm2**. Run sudo lvmdiskscan command to check for available partitions.

![](./images/g25.png)

**Note**: Previously, in Ubuntu we used apt command to install packages, in RedHat/CentOS a different package manager is used, so we shall use yum command instead.

- Use **pvcreate** utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```

![](./images/g26.png)

Verify that your Physical volume has been created successfully by running **sudo pvs**

![](./images/g27.png)

- Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
```
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
```

![](./images/g28.png)

Verify that your VG has been created successfully by running **sudo vgs**

![](./images/g29.png)

- Use lvcreate utility to create 2 logical volumes. db-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: db-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.
```
sudo lvcreate -n db-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```

![](./images/g30.png)


Verify that your Logical Volume has been created successfully by running **sudo lvs**

![](./images/g34.png)

Verify the entire setup
```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk 
```

![](./images/g35.png)

Use mkfs.ext4 to format the logical volumes with [ext4](https://en.wikipedia.org/wiki/Ext4) filesystem

```
sudo mkfs -t ext4 /dev/webdata-vg/db-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```

![](./images/g36.png)

Create /var/www/db directory to store database files
```
sudo mkdir -p /var/www/db
```

Create /home/recovery/logs to store backup of log data
```
sudo mkdir -p /home/recovery/logs
```

Mount /var/www/html on db-lv logical volume
```
sudo mount /dev/webdata-vg/db-lv /var/www/db/
```

Use [rsync](https://linux.die.net/man/1/rsync) utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

```
sudo rsync -av /var/log/. /home/recovery/logs/
```

![](./images/g37.png)

Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very important)
```
sudo mount /dev/webdata-vg/logs-lv /var/log
```

Restore log files back into /var/log directory
```
sudo rsync -av /home/recovery/logs/log/. /var/log
```


![](./images/g38.png)

Update /etc/fstab file so that the mount configuration will persist after restart of the server.

The UUID of the device will be used to update the /etc/fstab file;

```
sudo blkid
```

![](./images/bl2.png)


![](./images/g39.png)

sudo vi /etc/fstab

Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.

![](./images/fstab1.png)

![](./images/g40.png)

Test the configuration and reload the daemon
```
sudo mount -a
sudo systemctl daemon-reload
```

Verify your setup by running df -h, output must look like this:

![](./images/g41.png)


###  Install Wordpress on your Web Server EC2
Update the repository
```
sudo yum -y update
```
![](./images/g43.png)

Install wget, Apache and it???s dependencies
```
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
```

![](./images/g42.png)

Start Apache
```
sudo systemctl enable httpd
sudo systemctl start httpd
```

To install PHP and it???s depemdencies
```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```

Restart Apache
```
sudo systemctl restart httpd
```

Download wordpress and copy wordpress to var/www/html
```
mkdir wordpress
cd   wordpress
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz
sudo rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
cp -R wordpress /var/www/html/
```

Configure SELinux Policies
```
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
 ```

### Install MySQL on your DB Server EC2
```
sudo yum update
sudo yum install mysql-server
```

![](./images/g44.png)

Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:

```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```

###  Configure DB to work with WordPress
```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```

![](./images/g46.png)

### Configure WordPress to connect to remote database
Hint: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server???s IP address, so in the Inbound Rule configuration specify source as /32

![](./images/DB_inbound_rule.png)

Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client
```
sudo yum install mysql
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
```

![](./images/g47.png)

![](./images/g48.png)

Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.

![](./images/g49.png)

Change permissions and configuration so Apache could use WordPress:

Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation???s IP)

Try to access from your browser the link to your WordPress
```
http://<Web-Server-Public-IP-Address>/wordpress/
```

![](./images/wp_config.png)

Fill out your DB credentials:
![](./images/wp_config1.png)

If you see this message - it means your WordPress has successfully connected to your remote MySQL database

![](./images/wp_config_success.png)

Important: Do not forget to STOP your EC2 instances after completion of the project to avoid extra costs.