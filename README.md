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
   * Copy your key files on code repo

```sh
cp /etc/ssl/logstash-forwarder.crt $YOUR_HOME/DOCKER-ELK-stack/ELK/ssl/certs/logstash-forwarder.crt
cp /etc/ssl/logstash-forwarder.key $YOUR_HOME/DOCKER-ELK-stack/ELK/ssl/private/logstash-forwarder.key
```
### Step 2: Configure the cradentials for kibana
   * Open file and replace the PASSWORD string with your password
```sh
vi ELK/Dockerfile.nginx
>> RUN htpasswd -b -c /etc/nginx/htpasswd.users kibanaadmin PASSWORD
```

### Step 3: Build & Start the container
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






