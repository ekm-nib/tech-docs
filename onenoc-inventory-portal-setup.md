# ONENOC INVENTORY PORTAL SETUP GUIDE
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
> 1. Create a `Dockerfile` in `inventory-web-db` folder. `Dokcerfile` is common for all circle and it used to create a common web server container image
>
>     **Dockerfile**
>     ```
> 
>     FROM php:apache
> 
>     # install pdo extension
> 
>     RUN docker-php-ext-install pdo pdo_mysql
> 
>     # Use the default production configuration
> 
>     RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"
> 
>     ```
> 2. Create the web server image `localhost/onenoc/inventory-web:1.0.0`. It is the base image for all circles
>    ```
>    podman build -t localhost/onenoc/inventory-web:1.0.0 .
>    ```
>    List the image to check whether it created or not
>    ```
>     podman images
>
>    REPOSITORY                                   TAG         IMAGE ID      CREATED       SIZE
>    localhost/onenoc/inventory-web               1.0.1       39cb62a5f301  6 weeks ago   536 MB
>    ```
>    
> 3. Create the compose file `docker-compose.web.kl.yml`. This is the web server compose file for Kerala Circle. For other circles, replce the `.kl` with respective circle code.
>
>    **docker-compose.web.kl.yml** file
>    ```
>    version: "3"
>    services:
>        inventory-web-kl: 
>          image: localhost/onenoc/inventory-web:1.0.0
>          container_name: inventory_web_kl
>          networks:
>            - inventory_default
>          volumes:
>            - ./www:/var/www/html/kl
>          env_file: 
>            - .kl.env
>          restart: always
>    volumes:
>      www:
>    networks:
>      inventory_default:
>        external: true
>    ```
>    * PHP codes for web servers are copied to `www` folder and volume mapping is done with the container.
>    * container is named in a format to idetify with the circle code. `inventory_web_kl`
>    * service name for the web server is also in a format to idetify with the circle code. service name will be used in nginx proxy server to distinguish circle with URL.
>    * all the containers in the VM is attached to custom network named `inventory_default` so that containers can communicate each other.
>   
> 4. Create the compose file `docker-compose.db.kl.yml` for database server
>
>     **docker-compose.db.kl.yml** file
>    ```
>    version: "3"
>    services:
>        inventory-db-kl:
>          image: mysql:latest
>          container_name: inventory_db_kl
>          env_file:
>            - .kl.env
>          networks:
>            - inventory_default
>          volumes:
>            - ./db-data-kl:/var/lib/mysql
>    volumes:
>      db-data-kl:
>    networks:
>      inventory_default:
>        external: true
>    ```
>   *  local directory db-data-kl is mapped with container volume for data persistance.
>   *  local directory will be created automatically during container runtime
>     
> 5. Create the environment file `.kl.env` for customising database name, mysql root user password, database hostname etc.
>   **.kl.env** file
>      ```
>      MYSQL_ROOT_PASSWORD=password
>      MYSQL_HOSTNAME=inventory-db-kl
>      MYSQL_DATABASE=inventory_db_kl
>      MY_CIRCLE_NAME=KERALA
>      MY_CIRCLE_CODE=KL
>      NMS_API_SERVER=<IP:PORT of NMS API Server>
>      ```
>  * Both Web and Database server uses the same env file
>  * environment variables are used in PHP PDO config file for database connectivity and determining circle specific access
>  * NMS_API_SERVER will be different for different circles. It is used to get alarms and performace from third party applications like TEEVRA and BBNMS
>
> 6. STARTING THE WEB AND DATABASE SERVERS
>   * Once the required files are created, the `inventory-web-db` folder looks like,
>   ```
>   inventory-web-db/
>   ├── docker-compose.db.kl.yml
>   ├── docker-compose.web.kl.yml
>   ├── Dockerfile
>   └── www
>   ```
>   * `www` folder will have the html/php codes for the web server
>   * navigate to folder `inventory-web-db` in the terminal
>
>   1. Start the web server container with following command
>      ```
>      podman-compose -f docker-compose.web.kl.yml up -d
>      ```
>    * -f specifies the input compose file for web server
>   2. Next, start the database server container
>      ```
>      podman-compose -f docker-compose.db.kl.yml up -d
>      ```
>   3. Verify the container status
>      ```
>      podman ps
>
>      CONTAINER ID  IMAGE                                   COMMAND               CREATED     STATUS      PORTS                             NAMES
>      ab7b2aacf9a9  docker.io/library/mysql:latest          mysqld                6 days ago  Up 6 days                                     inventory_db_kl
>      f1848948a31c  localhost/onenoc/inventory-web:1.0.0    apache2-foregroun...  6 days ago  Up 6 days                                     inventory_web_kl
>      ```
>   Both the web and database servers is up.

## 4. SETUP phpmyadmin FOR DATABASE ACCESS

> 1. Create `docker-compose.phpmyadmin.yml` in phpmyadmin directory
>
>  **docker-compose.phpmyadmin.yml** 
>    ```
>    version: "3"
>    services:
>      phpmyadmin:
>        image: phpmyadmin/phpmyadmin
>        environment:
>          - PMA_ARBITRARY=1
>          - UPLOAD_LIMIT=50M
>          - MAX_EXECUTION_TIME=900
>        networks:
>          - inventory_default
>    networks:
>      inventory_default:
>        external: true
>    ```
> 2. Start the phpmyadmin container
>
>     ```
>     podman-compose -f docker-compose.phpmyadmin.yml up -d
>     ```
> 3. Verify the phpmyadmin container status
> 
>     ```
>     podman ps
>
>     CONTAINER ID  IMAGE                                   COMMAND               CREATED     STATUS      PORTS                             NAMES
>     2c3e83ae7926  docker.io/phpmyadmin/phpmyadmin:latest  apache2-foregroun...  6 days ago  Up 6 days                                     phpmyadmin_phpmyadmin_1
>     ```
> phpmyadmin container is up

## 5. SETUP nginx reverse proxy FOR INVENTORY PORTAL and PHPMYADMIN ACCESS
>  1. Create `docker-compose.nginx.yml` in nginx directory
>  **docker-compose.nginx.yml**
>    ```
>    version: '3'
>    services:
>      nginx:
>        image: nginx
>        ports:
>          - "8001:8001"
>          - "8002:8002"
>        volumes:
>          - ./www:/usr/share/nginx/html/
>          - ./nginx.conf:/etc/nginx/nginx.conf
>        networks:
>          - inventory_default
>    networks:
>      inventory_default:
>        external: true
>    volumes:
>      www:
>    ```
>    * port 8001 is for accessing Inventory portal
>    * port 8002 is for accessing phpmyadmin
>    * static html files if required may be copied in www folder
>    * nginx configuration may be set in nginx.conf
>  2. Create `nginx.conf` file for proxy setting
>  **nginx.conf** file
>    ```
>    events {
>        worker_connections 1024;  # Set the maximum number of simultaneous connections
>    }
>
>    http {
>        server {
>            listen 8001;
>            server_name mydomain.com;
>            root /usr/share/nginx/html/;
>
>            location / {
>                try_files $uri $uri/ $uri.html =404;
>            }
>    
>            location /kl/ {
>                proxy_pass http://inventory-web-kl:80/kl/;
>                proxy_buffering off;
>            }
>        }
>    
>         server {
>            listen 8002;
>            server_name mydomain.com;
>    
>            location / {
>                proxy_pass http://phpmyadmin:80;
>                proxy_buffering off;
>                proxy_read_timeout 900s;
>                proxy_send_timeout 900s;
>                client_max_body_size 100M;
>            }
>        }
>    }
>
>    ```
>    * since all the containers are in same podman netowrk, container service names are used in nginx.conf file
>    * nginx set to listen in 8001 for Inventiry server and listening in 8002 for phpmyadmin
>
>  3. Start the nginx
>    ```
>    podman-compose -f docker-compose.nginx.yml up -d
>    ```
>  4. Verify the nginx container
>
>     ```
>     podman ps
>
>     CONTAINER ID  IMAGE                                   COMMAND               CREATED     STATUS      PORTS                             NAMES
>     37c8aeb587fd  docker.io/library/nginx:latest          nginx -g daemon o...  6 days ago  Up 6 days   0.0.0.0:8001-8002->8001-8002/tcp  nginx_nginx_1
>     ```
>     * nginx is running and listening in port 8001 and 8002
>     * Check the access to Innetory port at http://<vm-ip>:8001
>     * Static html file is hosted in nginx to list all the circle inventory portals
>     * Kearala circle portal can be found at URL http://<vm-ip>:8001/kl
>     * database can be accessed by phpmyadmin package at http://<vm-ip>:8002

## 6. REPLICATION FOR OTHER CIRCLES
> * Create a copy `docker-compose.web.kl.yml` and  `docker-compose.db.kl.yml` and rename it by replaciong .kl with respective circle code 
> * Replace `kl` with respective circle code inside the files too
> * Create and modify the envirinment file for other circles like `.kl.env`
> * Start the web and databasae container using `podman-compose -f <circle-specific.yml file> up -d` commamd
> * add http_proxy in nginx.conf file for other circles
> * restart the nginx container
> * Inventory portal for other circles can be access at URL http://<vm-ip>:8001/<circle-code>
> * Database for different can be accessed using phpmyadmin. Server name to be given in phpmyadmin is same as container service name for each circle. Example `inventory-db-kl` for Kerala












