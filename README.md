
<a href="https://github.com/vivekyad4v?tab=followers"><img align="right" width="200" height="183" src="https://s3.amazonaws.com/github/ribbons/forkme_left_green_007200.png" /></a>

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


