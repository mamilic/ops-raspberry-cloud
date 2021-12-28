# raspberry-cloud



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
 openssl req -subj "/C=SR/ST=Macva/L=Sabac/O=Cloud/CN=Milic" \
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
 
 # UPDATE CA CERT ON ARCH
 sudo cp certs/ca.cert.crt /etc/ca-certificates/trust-source/anchors/
 sudo update-ca-trust 
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

## Nextcloud WEB Configuration
In order to connect Collabora with Nextcloud you need to enable Collabora Online APP,
and then setup connection to Collabora in Setting->Office, set URL https://RASPBERRY_IP:9980