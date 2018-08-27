<a href="https://github.com/Ashwanipal/DOCKER-ELK-stack"><img align="right" width="280" height="220" src="https://goo.gl/images/6WvsJP" /></a>

## How to install OpenStack on your local machine using Devstack

Follow the steps to Install Openstack cloud in your local BOX

### Minimum Requirments are:
    * Support Ubuntu 16.04/17.04, Fedora 24/25, CentOS/RHEL 7
    * 8 GB of memory
    * At least 50GB of storage

### Step 1: Stop the firewall in your Operating system:


### Step 2: Create a user "stack" to run DevStack:
```sh
sudo useradd -s /bin/bash -d /opt/stack -m stack
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






