## Compute service (NOVA)
OpenStack Compute is a major part of an Infrastructure-as-a-Service (IaaS) system. OpenStack Compute interacts with OpenStack Identity for authentication; OpenStack Image service for disk and server images; and OpenStack dashboard for the user and administrative interface. Image access is limited by projects, and by users; quotas are limited per project (the number of instances, for example). OpenStack Compute can scale horizontally on standard hardware, and download images to launch instances.

### Follow the steps to setup Openstack Compute service
 * 1: Install and configure controller node
 * 2: Install and configure compute node

## Install and configure controller node

### Step 1: Create & setup database
```sh
$ mysql -u root -p

mysql> CREATE DATABASE nova_api;                              ## Create the nova_api and nova databases:
mysql> CREATE DATABASE nova;
                                                              ## Grant proper access to the databases
mysql> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';
mysql> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';
mysql> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';
mysql> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';
```

### Step 2: Create the service credentials
* Source the admin credentials
```sh
$ . admin-openrc
```
* Create the nova user:
```sh
$ openstack user create --domain default --password-prompt nova

User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 8a7dbf5279404537b1c7b86c033620fe |
| name                | nova                             |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```
* Add the admin role to the nova user:
```sh
$ openstack role add --project service --user nova admin
```
* Create the nova service entity
```sh
$ openstack service create --name nova --description "OpenStack Compute" compute

+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Compute                |
| enabled     | True                             |
| id          | 060d59eac51b4594815603d75a00aba2 |
| name        | nova                             |
| type        | compute                          |
+-------------+----------------------------------+
```

### Step 3: Create the Compute service API endpoints
```sh
$ openstack endpoint create --region RegionOne compute public http://CONTROLLER_IP:8774/v2.1/%\(tenant_id\)s

+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | 3c1caa473bfe4390a11e7177894bcc7b          |
| interface    | public                                    |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | 060d59eac51b4594815603d75a00aba2          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://CONTROLLER_IP:8774/v2.1/%(tenant_id)s |
+--------------+-------------------------------------------+

$ openstack endpoint create --region RegionOne compute internal http://CONTROLLER_IP:8774/v2.1/%\(tenant_id\)s

+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | e3c918de680746a586eac1f2d9bc10ab          |
| interface    | internal                                  |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | 060d59eac51b4594815603d75a00aba2          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://CONTROLLER_IP:8774/v2.1/%(tenant_id)s |
+--------------+-------------------------------------------+

$ openstack endpoint create --region RegionOne compute admin http://CONTROLLER_IP:8774/v2.1/%\(tenant_id\)s

+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | 38f7af91666a47cfb97b4dc790b94424          |
| interface    | admin                                     |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | 060d59eac51b4594815603d75a00aba2          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://CONTROLLER_IP:8774/v2.1/%(tenant_id)s |
+--------------+-------------------------------------------+
```

### Step 4: Install and configure components
* Install the packages
```sh
$ apt install nova-api nova-conductor nova-consoleauth nova-novncproxy nova-scheduler
```
* Configure packages
```sh
[DEFAULT] 
transport_url = rabbit://openstack:RABBIT_PASS@CONTROLLER_IP
auth_strategy = keystone
my_ip = CONTROLLER_IP
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver


[api_database]                                      ## In the [api_database] and [database] sections, configure database access
...
connection = mysql+pymysql://nova:NOVA_DBPASS@CONTROLLER_IP/nova_api

[database]
...
connection = mysql+pymysql://nova:NOVA_DBPASS@CONTROLLER_IP/nova


[keystone_authtoken]
...
auth_uri = http://CONTROLLER_IP:5000
auth_url = http://CONTROLLER_IP:35357
memcached_servers = CONTROLLER_IP:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = NOVA_PASS

[vnc]
...
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip


[glance]
...
api_servers = http://CONTROLLER_IP:9292

[oslo_concurrency]
...
lock_path = /var/lib/nova/tmp
```
* Populate the Compute databases:
```sh
$ su -s /bin/sh -c "nova-manage api_db sync" nova
$ su -s /bin/sh -c "nova-manage db sync" nova                             ## Ignore any messages in this output
```
### Step 5: Restart the Compute services:
```sh
$ service nova-api restart
$ service nova-consoleauth restart
$ service nova-scheduler restart
$ service nova-conductor restart
$ service nova-novncproxy restart
```

## Now your Controller node is redy lets Install and configure compute node

### Step 1: Install and configure components
* Install the packages:
```sh
$ apt install nova-compute
```
* Configure packages
```sh
[DEFAULT]
...
transport_url = rabbit://openstack:RABBIT_PASS@CONTROLLER_IP
auth_strategy = keystone
my_ip = COMPUTE_IP                                             ## Add compute node IP addtess here
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[keystone_authtoken]
auth_uri = http://CONTROLLER_IP:5000
auth_url = http://CONTROLLER_IP:35357
memcached_servers = CONTROLLER_IP:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = NOVA_PASS

[vnc]
...
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://CONTROLLER_IP:6080/vnc_auto.html

[glance]
...
api_servers = http://CONTROLLER_IP:9292

[oslo_concurrency]
...
lock_path = /var/lib/nova/tmp
```
* Due to a packaging bug, remove the log-dir option from the [DEFAULT] section.

### Step 2: Determine whether your compute node supports hardware acceleration for virtual machines
```sh
$ egrep -c '(vmx|svm)' /proc/cpuinfo
```
### NOTE: If this command returns a value of one or greater, your compute node supports hardware acceleration which typically requires no additional configuration. If this command returns a value of zero, your compute node does not support hardware acceleration and you must configure libvirt to use QEMU instead of KVM.

### Step 3(Optional): Edit the [libvirt] section in the /etc/nova/nova-compute.conf file
```sh
[libvirt]
...
virt_type = qemu
```

### Step 4:Restart the Compute service
```sh
$ service nova-compute restart
```
### NOTE: If the nova-compute service fails to start, check /var/log/nova/nova-compute.log

### Step 5: Verify the operation on Compute service.
```sh
$ . admin-openrc

$ openstack compute service list

+----+--------------------+------------+----------+---------+-------+----------------------------+
| Id | Binary             | Host       | Zone     | Status  | State | Updated At                 |
+----+--------------------+------------+----------+---------+-------+----------------------------+
|  1 | nova-consoleauth   | controller | internal | enabled | up    | 2016-02-09T23:11:15.000000 |
|  2 | nova-scheduler     | controller | internal | enabled | up    | 2016-02-09T23:11:15.000000 |
|  3 | nova-conductor     | controller | internal | enabled | up    | 2016-02-09T23:11:16.000000 |
|  4 | nova-compute       | compute    | nova     | enabled | up    | 2016-02-09T23:11:20.000000 |
+----+--------------------+------------+----------+---------+-------+----------------------------+
```


##   Now your compute service is up and running To setup Networking service open the link

<a href="#"> Networking service </a>
 
