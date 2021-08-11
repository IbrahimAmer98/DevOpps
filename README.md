# Trainees TASK [linux + shell]


## Part 1 :LVM

pvcreate /dev/sdc #to create a new phiscal disk
vgcreate -s 16M vg01 /dev/sdc# create a new volume groupe "vg01"
lvcreate -l 50 -n lv01 vg01 # craete a new lV "lv01"and put it in Vg "vg01"
mkfs.ext4 /dev/vg01/lv01# formate LV "lv01" as ext4
blkid /dev/vg01/lv01# display LV "lv01" UUID    /dev/vg01/lv01: UUID="8e560d27-b8b0-412c-82f5-e1cd75fd72a2" TYPE="ext4"
mkdir /mnt/data# create data Diroctry for mounting
vim /etc/fstab# add new line UUID=8e560d27-b8b0-412c-82f5-e1cd75fd72a2 /mnt/data ext4 defaults 0 0
mount -a# to reflect the change
df -h# to display the work /dev/mapper/vg01-lv01  772M  1.6M  714M   1% /mnt/data

## ******************************************************************************************

## Part 2 : Users,Groups and Permissions

### 1

useradd -u 601 user1# create a new user "user1" and uid=601
passwd user1 thene redhat # make user1 password :"redhat"
vi /etc/ssh/sshd_config# add new line DenyUsers user1 to make it non-interactive
systemctl restart sshd# to make change works 
### 2
groupadd TrainingGroup# create a new groupe "TrainingGroup"
usermod -G TrainingGroup user1# add user1 to TrainingGroup
### 3
useradd user2
useradd user3
passwd user2 thene redhat
passwd user3 thene redhat# craete two user "user2/user3" password:"redhat"
groupadd admingroup# create a new groupe "admingroup"
usermod -G admingroup user2
usermod -G admingroup user3
vi /etc/passwd# change user3 to user3:x:0:0::/home/user3:/bin/bash

## ******************************************************************************************

## Part 3:SSH

ssh-keygen#Generating the key
ssh-copy-id me@172.20.51.124#copy public key to a remote system
******************************************************************************************

## Part 4: Permissions

cp /etc/fstab /var/tmp/admin#Copy /etc/fstab to /var/tmp name it admin
setfacl -m u:user1:rw admin#user1 could read, write and modify it
setfacl -m u:user2:--- admin#user2 can’t do any permission
getfacl admin#to check if it works
 file: admin
 owner: root
 group: root
user::rw-
user:user1:rw-
user:user2:---
group::r--
mask::rw-
other::r--

## ******************************************************************************************

## Part 5: Permissions

sestatus# display SELinux status
vi /etc/selinux/config then edit it to SELINUX=enforcing

## ******************************************************************************************

## Part 6:Bash Bcript and Processes

vi t2.sh#create shell script
#!/bin/bash

runtime="10 minute"
endtime=$(date -ud "$runtime" +%s)

while [[ $(date -u +%s) -le $endtime ]]
do
    echo "Time Now: `date +%H:%M:%S`"
    echo "Sleeping for 60 seconds"
    sleep 60
done

chmod +x t2.sh#change mode to run it
./t2.sh then ctrl +z then bg
jobs # to check process
ps # to display process 
kill -10 PID # to kill it

## ******************************************************************************************

## Part 7:Yum Repo

### 1
yum install tmux#Install tmux on your machine
### 2
yum install httpd
yum install mysql#Install apache server on your machine(httpd) and Install mysql
### 3
mkdir -p /var/www/html/repos/#Create a Directory to Store  Repo
rpm -Uvh https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm#nstall Zabbix repository

createrepo /var/www/html#Create the New Repo
cp /etc/yum.repos.d/*.repo /tmp/ #for protection 
vi /etc/yum.repos.d/remote.repo
[remote]

name=RHEL Apache

baseurl=http://172.20.51.94

enabled=1

gpgcheck=0

### 4
yum install --downloadonly --downloaddir=/var/www/html/repos fping 
yum install zabbix-agent --disablerepo="*" --enablerepo="remote"  # install  from my local repo
### 5
yum install zabbix-agent

yum install zabbix-server

yum install php

yum install zabbix-web
******************************************************************************************

## Part 8: Network management

### 1
firewall-cmd --zone=public --add-port=443/tcp 

firewall-cmd --zone=public --add-port=80/tcp #Open port 443 , 80
### 2
firewall-cmd --runtime-to-permanent #Make the changes permanent
### 3
firewall-cmd --add-rich-rule='rule family=ipv4 source address=172.20.51.94 service name=ssh reject'#Block ssh connection for your colleague ip to the VM
******************************************************************************************

## Part 9: Cronjob

vi /root/cj.sh#create shell script
date | tr -d '\n'>> /tmp/du
echo ' - ' | tr -d '\n' >> /tmp/du#timestamp – users tr -d to delete new line
users >> /tmp/du
crontab -e #to edit cron file
30 1 * * * /root/cj.sh#run at 1:30 AM every day
crontab -l#display your work
******************************************************************************************
## HeadingPart 10: Mariadb
### 1
yum install --downloadonly --downloaddir=/var/www/html/repos mariadb
yum install --disablerepo="*" --enablerepo="remote" mariadb
## 2
iptables -L#List the current Iptables rules
iptables-save > IPtablesbackup.txt#Backup the Iptables
iptables -A INPUT -p tcp --dport 3306 -j ACCEPT#open a Mysql port 3306
### 3
CREATE DATABASE rhce1;# create new DB called rhce1
CREATE USER 'user1'@localhost IDENTIFIED BY 'password1';# craete DB user "user1" password:"password1"
GRANT ALL PRIVILEGES ON rhce1.* to user1@localhost; # give user1 permission to rhce1 DB
FLUSH PRIVILEGES; 
### 4
mysql
system mysql -u user1 -p; #login as user1
connect rhce1;
CREATE TABLE studentdb (studentfirstname VARCHAR(20) , studentlastname VARCHAR(20) , program VARCHAR(20) , gyear INT(4) , number CHAR(7) PRIMARY KEY);#Create a MariaDB database called studentdb
INSERT INTO studentdb VALUES ('Allen','Brown','mechanical',2017,'110-001');# insert Allen/Brown/mechanical/2017/110-001
INSERT INTO studentdb VALUES ('Allen','David','electrical',2018,'110-002');
INSERT INTO studentdb VALUES ('Mary','Green','computer science',2018,'110-003');


## +------------------+-----------------+------------------+-------+---------+
## | studentfirstname | studentlastname | program          | gyear | number  |
## +------------------+-----------------+------------------+-------+---------+
## | Allen            | Brown           | mechanical       |  2017 | 110-001 |
## | Allen            | David           | electrical       |  2018 | 110-002 |
## | Mary             | Green           | computer science |  2020 | 110-003 |
## | Dennis           | Green           | mechanical       |  2018 | 110-004 |
## | Joseph           | Black           | electrical       |  2018 | 110-005 |
## | Dennis           | Black           | computer science |  2018 | 110-006 |
## | Ritchie          | Salt            | mechanical       |  2018 | 110-007 |
## +------------------+-----------------+------------------+-------+---------+
