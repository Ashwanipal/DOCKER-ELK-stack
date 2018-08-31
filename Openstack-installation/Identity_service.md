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
```sh
vi /etc/keystone/keystone.conf
>>  [database]                            ## In the [database] section, configure database access:
    ...                                   ## Replace KEYSTONE_DBPASS with the password you chose for the database.
    connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@CONTROLLER_IP/keystone

    [token]                               ## In the [token] section, configure the Fernet token provider
    ...
    provider = fernet
```
```sh
su -s /bin/sh -c "keystone-manage db_sync" keystone                                 ## Populate the Identity service database:

keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone     ## Initialize Fernet key repositories:
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone

keystone-manage bootstrap --bootstrap-password ADMIN_PASS \                         ## Bootstrap the Identity service:
  --bootstrap-admin-url http://CONTROLLER_IP:35357/v3/ \                               ## Replace ADMIN_PASS with a suitable password for an administrative user.
  --bootstrap-internal-url http://CONTROLLER_IP:35357/v3/ \
  --bootstrap-public-url http://CONTROLLER_IP:5000/v3/ \
  --bootstrap-region-id RegionOne
```

### Step 3:
```sh

```
### Step 4: 
```sh

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


