## Image service
The OpenStack Image service is central to Infrastructure-as-a-Service (IaaS) as shown in Conceptual architecture. It accepts API requests for disk or server images, and metadata definitions from end users or OpenStack Compute components. It also supports the storage of disk or server images on various repository types, including OpenStack Object Storage

### Follow the steps to setup Openstack image service on the controller node

### Step 1: Create the database and Grant proper access to the glance database:
```sh
$ mysql -u root -p

mysql> CREATE DATABASE glance;

mysql> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'GLANCE_DBPASS';
mysql> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'GLANCE_DBPASS';
```
### Step 2: Source the admin credentials to gain access to admin-only CLI commands
```sh
. admin-openrc
```
### Step 3: Create the service credentials
* Create the glance user:
```sh
$ openstack user create --domain default --password-prompt glance

User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 3f4e777c4062483ab8d9edd7dff829df |
| name                | glance                           |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```
* Add the admin role to the glance user and service project:
```sh
openstack role add --project service --user glance admin
```
* Create the glance service entity:
```sh
$ openstack service create --name glance --description "OpenStack Image" image

+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Image                  |
| enabled     | True                             |
| id          | 8c2c7f1b9b5049ea9e63757b5533e6d2 |
| name        | glance                           |
| type        | image                            |
+-------------+----------------------------------+
```

### Step 4: Create the Image service API endpoints:
```sh
$ openstack endpoint create --region RegionOne \
  image public http://controller:9292

+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 340be3625e9b4239a6415d034e98aace |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 8c2c7f1b9b5049ea9e63757b5533e6d2 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+

$ openstack endpoint create --region RegionOne \
  image internal http://controller:9292

+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | a6e4b153c2ae4c919eccfdbb7dceb5d2 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 8c2c7f1b9b5049ea9e63757b5533e6d2 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+

$ openstack endpoint create --region RegionOne \
  image admin http://controller:9292

+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 0c37ed58103f4300a84ff125a539032d |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 8c2c7f1b9b5049ea9e63757b5533e6d2 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+
```
### Step 5: Install and configure components
*  Install the packages:
```sh
$ apt install glance
```
* Edit the /etc/glance/glance-api.conf file and complete the following actions:
```sh
[database]                                                              ## configure database access in [database] section
...
connection = mysql+pymysql://glance:GLANCE_DBPASS@CONTROLLER_IP/glance

[keystone_authtoken]                                                    ## Configure Identity service access
auth_uri = http://CONTROLLER_IP:5000
auth_url = http://CONTROLLER_IP:35357
memcached_servers = CONTROLLER_IP:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = GLANCE_PASS

[paste_deploy]                                                          ## Configure Identity service access
...
flavor = keystone

[glance_store]                                                          ## configure the local file system store image files:
...
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/

```
* Edit the /etc/glance/glance-registry.conf file and complete the following actions
```sh
[database]                                                              ## In the [database] section, configure database access
...
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance

[keystone_authtoken]                                                    ## configure Identity service access:
...
auth_uri = http://CONTROLLER_IP:5000
auth_url = http://CONTROLLER_IP:35357
memcached_servers = CONTROLLER_IP:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = GLANCE_PASS

[paste_deploy]
...
flavor = keystone
```

### Step 6: Populate the Image service database & Restart the Image services:
```sh
$ su -s /bin/sh -c "glance-manage db_sync" glance
```
```sh
$ service glance-registry restart
$ service glance-api restart
```
### Step 7: Verify operation
* Source the admin credentials to gain access to admin-only CLI commands:
```sh
$ . admin-openrc
```
* Download the image
```sh
$ wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img
```
* Upload the image to the Image service using the QCOW2 disk format,
```sh
openstack image create "cirros" \
  --file cirros-0.3.4-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
```
* Confirm the uploaded image
```sh
$ openstack image list

+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| 38047887-61a7-41ea-9b49-27987d5e8bb9 | cirros | active |
+--------------------------------------+--------+--------+
```
## Now your servers are ready to install and configure Openstack Compute service
<a href="https://github.com/Ashwanipal/DOCKER-ELK-stack/blob/master/Openstack-installation/Compute_service.md"> 2: Compute service </a>
 
