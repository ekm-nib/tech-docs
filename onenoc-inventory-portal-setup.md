# ONENOC INVENTORY PORTAL SETUP GUIDE
### Version 1.0.0

## INTRODUCTION

Onenoc Inventory Portal is a web server and database package to store the details of devices and links. Web server is the frontend part through which users can add, edit and delete the device and links data. The data is stored in the database server. 

Web Server = Apache + PHP

Database = MySQL

## PAN INDIA IMPLEMENTATION

* Each circle will have seperate web server and database server.
* These servers will be installed in SDC Virtual Machine alloted for ONENOC Project as Podman container
* An nginx reverse proxy server will be used to provide access to web server 
* End users will have access to Web server through CDR network
* Nodal users in each Circle will have restricted access to database server through phpmyadmin package hosted in the VM

## CONFIGURATION OF SDC VIRTUAL MACHINE

1. INSATLL PODMAN


```
```
2. 










