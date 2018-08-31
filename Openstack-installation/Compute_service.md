## Compute service (NOVA)
OpenStack Compute is a major part of an Infrastructure-as-a-Service (IaaS) system. OpenStack Compute interacts with OpenStack Identity for authentication; OpenStack Image service for disk and server images; and OpenStack dashboard for the user and administrative interface. Image access is limited by projects, and by users; quotas are limited per project (the number of instances, for example). OpenStack Compute can scale horizontally on standard hardware, and download images to launch instances.

### Follow the steps to setup Openstack Compute service
  * Install and configure controller node
  * Install and configure compute node

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
* 
```sh

```


## Now your servers are ready to install and configure Openstack Compute service
<a href="#"> 2: Compute service </a>
 
