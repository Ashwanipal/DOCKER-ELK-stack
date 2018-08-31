<a href="https://github.com/Ashwanipal/DOCKER-ELK-stack"><img align="right" width="280" height="220" src="https://github.com/Ashwanipal/Shell-scripts/blob/master/2000px-The_OpenStack_logo.svg.png" /></a>

## This tutorial contains Creation & management of OpenStack multinode architecture 

Follow the steps to create your own Openstack multinode cloud

### Requirements:
   * It can be your virtual machines & you can creat VM's using vagrant
   * Support Ubuntu 16.04/14.04
   * Controller Node: 4GB of RAM & 20GB storage
   * Compute Node: 6GB of RAM & 20GB storage
   * Storage Node: 2GB of RAM & 20GB storage

### Step 1: Stop the firewall in your Operating system:

### Step 2: Install Chrony for implementation of NTP, to properly synchronize services among nodes
  * IP address of Controller Node:  192.168.33.10
  * IP address of Compute Node:     192.168.33.11
  * IP address of Storage Node:     192.168.33.12
#### Install and configure components in controller node:
```sh
apt install chrony

vi /etc/chrony/chrony.conf
>>  server CONTROLLER_IP iburst               ## Add the configuration
    allow 192.168.0.0/24

service chrony restart
```
#### Perform these steps on all other nodes.
```sh
apt install chrony

vi /etc/chrony/chrony.conf
>> server 192.168.33.10 iburst ## Add the controller node IP address

service chrony restart
```
#### Verify operation
```sh
chronyc sources

  210 Number of sources = 2
  MS Name/IP address         Stratum Poll Reach LastRx Last sample
  ===============================================================================
  ^- 192.168.33.10                    2   7    12   137  -2814us[-3000us] +/-   43ms
  ^* 192.168.33.11                    2   6   177    46    +17us[  -23us] +/-   68ms
```
Run the same command on all other nodes:

### Step 3: OpenStack packages (Perform these procedures on all nodes)
```sh
apt install software-properties-common    ## Enable the OpenStack repository
add-apt-repository cloud-archive:newton   

apt update && apt dist-upgrade            ## Upgrade the packages on your host

apt install python-openstackclient        ## Install the OpenStack client
```
### Step 4: SQL database (Perform these procedures on controller node)
```sh
apt install mysql-server python-pymysql   ## Install the packages & set the suitable password for your MySQL server

vi /etc/mysql/mysql.conf.d/mysqld.cnf 
>> bind-address = 0.0.0.0                 ## Change the bind address

service mysql restart                     ## Restart the database service
```
### Step 5: Message queue (Install and configure components on controller node)
```sh
apt install rabbitmq-server                 ## Install the package

rabbitmqctl add_user openstack RABBIT_PASS  ## Add the openstack user: 
OUTPUT >> Creating user "openstack" ...

rabbitmqctl set_permissions openstack ".*" ".*" ".*"
OUTPUT >> Setting permissions for user "openstack" in vhost "/" ...
```
### Step 5: Memcached (Install and configure components on controller node)
```sh
apt install memcached python-memcache       ## Install the packages

vi /etc/memcached.conf                      ## Configure memcached by editing configuration file
>> -l 192.168.33.10                         ## Add the IP address of ypur controller node

service memcached restart                   ## Restart the Memcached service:
```

## Now your servers are ready to install and configure Openstack services

### To install services one by one please open the links

  * <a href="https://github.com/Ashwanipal/DOCKER-ELK-stack/blob/master/Openstack-installation/Identity_service.md"> 1: Identity service  (keystone)</a>
  * <a href="https://github.com/Ashwanipal/DOCKER-ELK-stack/blob/master/Openstack-installation/Image_service.md"> 2: Image service (glance)</a>
  * <a href="#"> 3: Compute service (nova)</a>
  * <a href="#"> 4: Networking service (neutron)</a>
  * <a href="#"> 5: Dashboard (openstack-dashboard)</a>
  * <a href="#"> 6: Block Storage service (cinder)</a>

### NOTE: Please make sure to follow the steps as mentioned here


