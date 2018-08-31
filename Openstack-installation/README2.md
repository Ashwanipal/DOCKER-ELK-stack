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
  * IP address of Storage Node:     192.168.33.10
## Install and configure components 
controller node:
```sh
apt install chrony

vi /etc/chrony/chrony.conf
>> server NTP_SERVER iburst ## Add the configuration

vi /etc/chrony/chrony.conf
>> allow 192.168.33.0/24 ## Add the configuration

service chrony restart
```
### Step 3: Grant privileges to user stack:
```sh
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
```
You should see output
```sh
stack ALL=(ALL) NOPASSWD: ALL
```

### Step 4: Switch user and Download DevStack:

```sh
sudo su - stack
```
```sh
git clone https://github.com/openstack-dev/devstack.git -b stable/pike devstack/
```

### Step 5: Change to the devstack directory:
```sh
cd devstack
```

### Step 6: Create the local.conf file with password and your host IP:
```sh
cat >  local.conf <<EOF
[[local|localrc]]
ADMIN_PASSWORD=secret
DATABASE_PASSWORD=\$ADMIN_PASSWORD
RABBIT_PASSWORD=\$ADMIN_PASSWORD
SERVICE_PASSWORD=\$ADMIN_PASSWORD
HOST_IP=10.0.2.15
RECLONE=yes
EOF
```

### Now install and run OpenStack:
```sh
./stack.sh
```

### Access your OpenStack by  URL:
```sh
http://<HOST_IP>/dashboard
```






