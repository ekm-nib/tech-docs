# ONENOC INVENTORY PORTAL IMPLEMENTATION GUIDE
#### Version 1.0.0
#### Date 10-04-2024

## INTRODUCTION

Onenoc Inventory Portal is a web server and database package to store the details of devices and links. Web server is the frontend part through which users can add, edit and delete the device and links data. The data is stored in the database server. 

Web Server = Apache + PHP

Database = MySQL

## PAN INDIA IMPLEMENTATION

* Each circle has seperate web server and database server.
* These servers are installed in SDC Virtual Machine alloted for ONENOC Project
* Web and database servers are running in VM as containers using PODMAN 
* An nginx reverse proxy server is used to provide access to web servers 
* End users will have access to Web server through CDR network
* Nodal users in each Circle will have restricted access to database server through phpmyadmin package hosted in the VM

## CONFIGURATION OF INVENTORY PORTAL IN SDC VIRTUAL MACHINE

Following are the packages used in Inventory portal
* Apache + PHP webserver stack for Invnetory Portal frontend
* MySQL database server for storing data
* phpmyadmin package for database operations
* ngnix for routing the access to Circle specific invnetory portal and phpmyadmin

All the packages are running in VM as containers using PODMAN. 

Various steps for configuring VMs are given below

* Install PODMAN and podman-compose
* Create Directories for varoius packages
* Setup Inventory Portal & MySQL database
* Setup phpmyadmin
* Setup nginx reverse proxy

## 1. INSTALL PODMAN & PODMAN-COMPOSE
* ssh to VM
* make sure internet connectivity in VM
* install podman
  1. update system packages
  ```
  sudo yum update -y
  ```
  2. install EPEL repository (if not installed already)
  ```
  sudo yum install epel-release -y
  ```
  3. Install podman
  ```
  sudo yum install podman -y
  ```
  4. Verify podman installation
  ```
  podman --version
  podman version 4.6.1
  ```
* install podman-compose
  1. install python
  ```
  sudo yum install python3
  ```
  2. install pip
  ```
  sudo pip3 install podman-compose
  ```
  3. install podman-compose using pip
  ```
  sudo pip3 install podman-compose
  ```
  5. verify installation
  ```
  podman-compose --version

  ['podman', '--version', '']
  using podman version: 4.6.1
  podman-compose version 1.0.6
  podman --version
  podman version 4.6.1
  exit code: 0
  ```
Now PODMAN and podman-compose are installed.
## 2. CREATE DIRECTORIES FOR VARIOUS PACKAGES
Diretory structure for Inventoy portal is given below
```
inventory-production/
├── inventory-web-db
├── nginx
└── phpmyadmin
```
Create the directories in /home folder or in any suitable location
```
mkdir -p inventory-production/{inventory-web-db,nginx,phpmyadmin}
```
* **inventory-web-db** : for web server and database server
* **nginx** : for nginx reverse proxy
* **phpmyadmin** : for phpmyadmin package

Now the folders are created.

## 3. SETUP INVENTORY WEB SERVER AND MYSQL DATABASE SERVER
* Web server is a Apache + PHP stack
* php:apache docker image is used to run the container using PODMAN
* pdo_mysql extension is used for accessing database from PHP. It is installed during the image build process as per the command mentioned in `Dockerfile`

Steps for creating the Apache + PHP + MySQL stack for Kerala Circle (circle code KL) is given below
> 1. Create a `Dockerfile` in `inventory-web-db` folder. `Dokcerfile` is common for all circle.
>
>   **Dockerfile**
>   ```
> 
>   FROM php:apache
> 
>   # install pdo extension
> 
>   RUN docker-php-ext-install pdo pdo_mysql
> 
>   # Use the default production configuration
> 
>   RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"
> 
>   ```
> 2. Create the compose file `docker-compose.web.kl.yml`. This is the web server compose file for Kerala Circle. For other circles, replce the `.kl` with respective circle code.
>
>   **docker-compose.web.kl.yml**
>   ```
>   
>   ```
> 3. Create the compose file `docker-compose.db.kl.yml` for database server
> 4. Create the environment file `.kl.env` for customising database name, mysql root user password, database hostname etc.  















