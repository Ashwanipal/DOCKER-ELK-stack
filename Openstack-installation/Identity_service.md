## Identity service (keystone)
The project that facilitates API client authentication, service discovery, distributed multi-tenant authorization, and auditing. It provides a central directory of users mapped to the OpenStack services they can access. It also registers endpoints for OpenStack services and acts as a common authentication system

### Follow the steps to setup Identity service in your openstack cluster

### Step 1: Before you configure the OpenStack Identity service, you must create a database and an administration token.
```sh
mysql -u root -p

mysql> CREATE DATABASE keystone;          ## Create the keystone database & Grant proper access to the keystone database
mysql> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'KEYSTONE_DBPASS';
mysql> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'KEYSTONE_DBPASS';
```

### Step 2: Install and configure components
```sh
apt install keystone                      ## install the packages:
```
* Edit the /etc/keystone/keystone.conf file and complete the following actions:
```sh
vi /etc/keystone/keystone.conf
>>  [database]                            ## In the [database] section, configure database access:
    ...                                   ## Replace KEYSTONE_DBPASS with the password you chose for the database.
    connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@CONTROLLER_IP/keystone

    [token]                               ## In the [token] section, configure the Fernet token provider
    ...
    provider = fernet
```
* Populate the Identity service database & Initialize Fernet key repositories
```sh
su -s /bin/sh -c "keystone-manage db_sync" keystone                                 

keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```
* Bootstrap the Identity service & Replace ADMIN_PASS with a suitable password for an administrative user.
```sh
keystone-manage bootstrap --bootstrap-password ADMIN_PASS \                         
  --bootstrap-admin-url http://CONTROLLER_IP:35357/v3/ \
  --bootstrap-internal-url http://CONTROLLER_IP:35357/v3/ \
  --bootstrap-public-url http://CONTROLLER_IP:5000/v3/ \
  --bootstrap-region-id RegionOne
```

### Step 3:Configure the Apache HTTP server
* Edit the /etc/apache2/apache2.conf file and configure the ServerName option to controller node IP address:
```sh
ServerName controller
```
* Restart the Apache service and remove the default SQLite database:
```sh
service apache2 restart
rm -f /var/lib/keystone/keystone.db
```
* Configure the administrative account
```sh
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://CONTROLLER_IP:35357/v3
export OS_IDENTITY_API_VERSION=3
```

### Step 4: Create a domain, projects, users, and roles
* Create the service & demo project
```sh
openstack project create --domain default --description "Service Project" service

openstack project create --domain default --description "Demo Project" demo
```
* Create the demo user & user role
```sh
openstack user create --domain default --password-prompt demo                   ## Create the demo user
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | aeda23aa78f44e859900e22c24817832 |
| name                | demo                             |
| password_expires_at | None                             |
+---------------------+----------------------------------+

openstack role create user                                                      ## Create the user role:
+-----------+----------------------------------+
| Field     | Value                            |
+-----------+----------------------------------+
| domain_id | None                             |
| id        | 997ce8d05fc143ac97d83fdfb5998552 |
| name      | user                             |
+-----------+----------------------------------+

openstack role add --project demo --user demo user                              ## Add the user role to the demo project and user
```
### Step 5: 
```sh

```
### Step 5:
```sh

```

## Now your servers are ready to install and configure Openstack services

### To install services one by one please open the links

  * <a href="#"> 1: Identity service </a>
  * <a href="#"> 2: Image service </a>
  * <a href="#"> 3: Compute service </a>
  * <a href="#"> 4: Networking service </a>
  * <a href="#"> 5: Dashboard </a>
  * <a href="#"> 6: Block Storage service </a>

### NOTE: Please make sure to follow the steps as mentioned here


