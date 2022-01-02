# raspberry-cloud

### TODO
- [ ] Improve folder permissions, Remove become:true where not necessary

## Introduction

This is ansible script for nextcloud deployment on **raspberrypi 4** with following services:
1. [Nextcloud](https://nextcloud.com/)
1. [Mariadb](https://mariadb.org/)
1. [Nginx](https://www.nginx.com/)
1. [Collabora](https://www.collaboraoffice.com/)

## Prerequisites
You need to have configured ssh key based auth between your machines in order to run ansible script

## Generate SSL Certificates 
```
 mkdir certs csr private
 touch index.txt
 echo 1000 > serial
 
 ###############
 ### ROOT CA ###
 ###############
 # CREATE ROOT CA KEY
 openssl genrsa -out private/ca.key.pem 2048
 
 # CREATE ROOT CA cert, this is also ca-chain.cert.pem
 openssl req -subj "/C=SR/ST=Macva/L=Sabac/O=Milic/CN=Milic Certification Authority" \
      -key private/ca.key.pem \
      -new -x509 -days 7300 -sha256 -extensions v3_ca \
      -out certs/ca.cert.pem
      
  # SIGN THE CSR WITH ROOT CA
  openssl x509 -req -days 365 -CA certs/ca.cert.pem -CAkey private/ca.key.pem \
      -CAcreateserial -CAserial serial -in csr/cloud.next.lan.csr.pem -out certs/cloud.next.lan.cert.pem

 ##############
 ### SERVER ###
 ##############
 # CREATE KEY for SERVER
 # -aes256 - not working 
 openssl genrsa -out private/cloud.next.lan.key.pem 2048
 
 # CREATE CSR FOR SERVER
 # Use the private key to create a certificate signing request (CSR)
 # For server certificates, the Common Name must be a fully qualified domain name (eg, www.example.com), 
 # whereas for client certificates it can be any unique identifier (eg, an e-mail address). 
 # Note that the Common Name cannot be the same as either your root or intermediate certificate.
 openssl req -config config.conf \
      -key private/cloud.next.lan.key.pem \
      -new -sha256 -out csr/cloud.next.lan.csr.pem

 # Convert ROOT CA PEM TO CRT
 openssl x509 -outform der -in certs/ca.cert.pem -out certs/ca.cert.crt
```

config.conf
```
[req]
distinguished_name  = req_distinguished_name
prompt              = no
string_mask         = utf8only
 
[req_distinguished_name]
C                   = 
ST                  = 
L                   = 
O                   = 
OU                  = 
CN                  = cloud.next.lan
 
[v3_req]
keyUsage            = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage    = serverAuth
subjectAltName      = @alt_names

[alt_names]
DNS.5				= cloud.next.lan
IP.1                = 192.168.x.x
```

## Install CA CERT on machine to trust CERT
First, convert CA pem to crt 
```
 openssl x509 -outform der -in certs/ca.cert.pem -out certs/ca.cert.crt                              
```
For arch distros copy **ca-chain.cert.crt** into /etc/ca-certificates/trust-source/anchors/
```
 sudo cp certs/ca.cert.crt /etc/ca-certificates/trust-source/anchors/
 sudo update-ca-trust 
```

## Getting started
Start script with:
```
ansible-playbook -i deploy/playbooks/inventories/raspberry-pi deploy/playbooks/cloud-deploy.yml --verbose --ask-vault-pass
```

## ADD ROOT CA TO NEXTCLOUD
```
sudo nano nextcloud/resources/config/ca-bundle.crt

#Apped to the end of the file your ROOT CA pem 
```

## Nextcloud WEB Configuration
In order to connect Collabora with Nextcloud you need to enable Collabora Online APP,
and then setup connection to Collabora in Setting->Office, set URL https://RASPBERRY_IP:9980

## Backup Data
In order to backup folder from remote server use following command:
```
rsync -a cloud.next.lan:/home/pi/nextcloud ~/ --info=progress2
```

## Performance Tuning NGINX
Run this command to find out requests per minute
```
ab -c 40 -n 50000  https://cloud.next.lan/ | grep "per second"
```

## Performance Tuning NEXTCLOUD
Chunk upload size:
```
# Log into docker container
docker exec -it c0c532ab9700 /bin/sh

# Connect to DB
mysql -u nextcloud -p

# Select Database
use nextcloud;

# Insert or update row, chunk size value is in bytes
# Please take into consideration specs of your Router, as it can cause bottleneck
update oc_appconfig set configvalue = 20971520 where configkey like 'max_chunk_size';
```

## Delete failed partial uploads
```
find /path-to/nextcloud -type f -iname \*.part -delete
```