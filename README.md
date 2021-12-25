# raspberry-cloud



## Introduction

This is ansible script for nextcloud deployment on **raspberrypi 4** with following services:
1. [Nextcloud](https://nextcloud.com/)
1. [Mariadb](https://mariadb.org/)
1. [Nginx](https://www.nginx.com/)

## Prerequisites
You need to have configured ssh key based auth

## Getting started
Start script with:
```
ansible-playbook -i deploy/playbooks/inventories/raspberry-pi deploy/playbooks/cloud-deploy.yml --verbose 
```