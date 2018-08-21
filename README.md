## ELK stack setup on Docker
   * Clone this repo -
   * Install Docker & Docker-Swarm in your machine

## Follow the steps to start Docker ELK stack

### Step 1: Create SSL certificate to ship logs from client to ELK server
   * Open the file:
```sh
vi  /etc/ssl/openssl.cnf 
```

   * Look for “[ v3_ca ]” section and replace the IP of your logstash server: 
```sh
subjectAltName = IP:192.168.12.10
```

   * Goto OpenSSL directory & create a SSL certificate by running following command:
```sh
cd /etc/ssl/
openssl req -x509 -days 365 -batch -nodes -newkey rsa:2048 -keyout logstash-forwarder.key -out logstash-forwarder.crt
```
   * Copy your key files on code repo:

```sh
cp /etc/ssl/logstash-forwarder.crt $YOUR_HOME/DOCKER-ELK-stack/ELK/ssl/certs/logstash-forwarder.crt
cp /etc/ssl/logstash-forwarder.key $YOUR_HOME/DOCKER-ELK-stack/ELK/ssl/private/logstash-forwarder.key
```
### Step 2: Configure the cradentials for kibana
   * Open file and replace the PASSWORD string with your password:
```sh
vi ELK/Dockerfile.nginx
>> RUN htpasswd -b -c /etc/nginx/htpasswd.users kibanaadmin PASSWORD
```
### Step 3: Configure filebeat on client machine
  * download filebeat from the official website:
```sh
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/beats.list
```
  * Install Filebeat:
```sh
apt-get update 
apt-get install -y filebeat
```
  * Copy you SSL certificate from ELK server to client machine
```sh
scp -pr root@server_ip:/FULL_PATH_OF_CERTIFICATE/logstash-forwarder.crt /etc/ssl
```
  * To Configure Filebeat open the file
```sh
vi /etc/filebeat/filebeat.yml
```
```sh
.  .  .
      paths:
        - /var/log/syslog
        ## Enter your log file full path here
.  .  .
```
```sh
.   .   .
	output.logstash:
	  # The Logstash hosts
	  hosts: ["ELK_SERVER_IP:5044"]

	  # Optional SSL. By default is off.
	  # List of root certificates for HTTPS server verifications
	  ssl.certificate_authorities: ["/etc/ssl/logstash-forwarder.crt"]
.   .   .
```

### Step 4: Build & Start the container
   * Build the images of Elasticsearch, Logstash  & Kibana
```sh
docker-compose -f ELK/elk-stack.yml build
```
  * Start the containers
```sh
docker-compose -f ELK/elk-stack.yml up -d
```
  * Wait for 5 minuts
  * Access your newly created Kibana dashboard with url http://YOUR_SERVER_IP
  * Enter USER_NAME kibanaadmin and your PASSWORD






