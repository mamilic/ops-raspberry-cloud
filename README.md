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
 mkdir -p /opt/ssl/
 cd /opt/ssl/
 mkdir -p certs/ca
 openssl genrsa -out certs/ca/root.key.pem 2048
 openssl req -x509 -new -nodes -key certs/ca/root.key.pem -days 9131 -out certs/ca/root.crt.pem -subj "/C=DE/ST=BW/L=Stuttgart/O=Dummy Authority/CN=Dummy Authority"
 mkdir -p certs/{servers,tmp}
 mkdir -p "certs/servers/hostname.example.com"
 openssl genrsa -out "certs/servers/hostname.example.com/privkey.pem" 2048 -key "certs/servers/hostname.example.com/privkey.pem"
 openssl req -key "certs/servers/hostname.example.com/privkey.pem" -new -sha256 -out "certs/tmp/hostname.example.com.csr.pem" -subj "/C=DE/ST=BW/L=Stuttgart/O=Dummy Authority/CN=hostname.example.com"
 openssl x509 -req -in certs/tmp/hostname.example.com.csr.pem -CA certs/ca/root.crt.pem -CAkey certs/ca/root.key.pem -CAcreateserial -out certs/servers/hostname.example.com/cert.pem -days 9131
 mv certs/servers/hostname.example.com/privkey.pem /etc/coolwsd/key.pem
 mv certs/servers/hostname.example.com/cert.pem /etc/coolwsd/cert.pem
 mv certs/ca/root.crt.pem /etc/coolwsd/ca-chain.cert.pem```
```

## Getting started
Start script with:
```
ansible-playbook -i deploy/playbooks/inventories/raspberry-pi deploy/playbooks/cloud-deploy.yml --verbose 
```

## Nextcloud WEB Configuration
In order to connect Collabora with Nextcloud you need to enable Collabora Online APP,
and then setup connection to Collabora in Setting->Office, set URL https://RASPBERRY_IP:9980