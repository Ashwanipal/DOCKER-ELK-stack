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
### Step 5: Verify the Identity service before installing other services.
* To disable the temporary authentication token mechanism edit the /etc/keystone/keystone-paste.ini file and remove admin_token_auth from the [pipeline:public_api], [pipeline:admin_api], and [pipeline:api_v3] sections.
* Unset the temporary OS_AUTH_URL and OS_PASSWORD environment variable:
```sh
unset OS_AUTH_URL OS_PASSWORD
```
* As the admin and demo user, request an authentication token:
```sh
openstack --os-auth-url http://controller:35357/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name admin --os-username admin token issue
Password:
+------------+-----------------------------------------------------------------+
| Field      | Value                                                           |
+------------+-----------------------------------------------------------------+
| expires    | 2016-02-12T20:14:07.056119Z                                     |
| id         | gAAAAABWvi7_B8kKQD9wdXac8MoZiQldmjEO643d-e_j-XXq9AmIegIbA7UHGPv |
|            | atnN21qtOMjCFWX7BReJEQnVOAj3nclRQgAYRsfSU_MrsuWb4EDtnjU7HEpoBb4 |
|            | o6ozsA_NmFWEpLeKy0uNn_WeKbAhYygrsmQGA49dclHVnz-OMVLiyM9ws       |
| project_id | 343d245e850143a096806dfaefa9afdc                                |
| user_id    | ac3377633149401296f6c0d92d79dc16                                |
+------------+-----------------------------------------------------------------+

openstack --os-auth-url http://controller:5000/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name demo --os-username demo token issue
Password:
+------------+-----------------------------------------------------------------+
| Field      | Value                                                           |
+------------+-----------------------------------------------------------------+
| expires    | 2016-02-12T20:15:39.014479Z                                     |
| id         | gAAAAABWvi9bsh7vkiby5BpCCnc-JkbGhm9wH3fabS_cY7uabOubesi-Me6IGWW |
|            | yQqNegDDZ5jw7grI26vvgy1J5nCVwZ_zFRqPiz_qhbq29mgbQLglbkq6FQvzBRQ |
|            | JcOzq3uwhzNxszJWmzGC7rJE_H0A_a3UFhqv8M4zMRYSbS2YF0MyFmp_U       |
| project_id | ed0b60bf607743088218b0a533d5943f                                |
| user_id    | 58126687cbcc4888bfa9ab73a2256f27                                |
+------------+-----------------------------------------------------------------+
```
### Step 6: Create OpenStack client environment scripts to load appropriate credentials for client operations
* Create admin-openrc file and add the following content
```sh
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_AUTH_URL=http://CONTROLLER_IP:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```
* Create demo-openrc file and add the following content
```sh
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=DEMO_PASS
export OS_AUTH_URL=http://CONTROLLER_IP:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```
#### Note: replace ADMIN_PASS with the password and CONTROLLER_IP with the IP of your controller node
* Using the scripts Request an authentication token
```sh
. admin-openrc

openstack token issue

+------------+-----------------------------------------------------------------+
| Field      | Value                                                           |
+------------+-----------------------------------------------------------------+
| expires    | 2016-02-12T20:44:35.659723Z                                     |
| id         | gAAAAABWvjYj-Zjfg8WXFaQnUd1DMYTBVrKw4h3fIagi5NoEmh21U72SrRv2trl |
|            | JWFYhLi2_uPR31Igf6A8mH2Rw9kv_bxNo1jbLNPLGzW_u5FC7InFqx0yYtTwa1e |
|            | eq2b0f6-18KZyQhs7F3teAta143kJEWuNEYET-y7u29y0be1_64KYkM7E       |
| project_id | 343d245e850143a096806dfaefa9afdc                                |
| user_id    | ac3377633149401296f6c0d92d79dc16                                |
+------------+-----------------------------------------------------------------+
```


## Now your servers are ready to install and configure Image service

 <a href="https://github.com/Ashwanipal/DOCKER-ELK-stack/blob/master/Openstack-installation/Image_service.md"> install Image service </a>
