## How to create a local registry for docker images in Ubuntu

1. Create a registry folder for storing image and auth files ( data : for storing images & auth : for storing http basic authentication files )

        mkdir -p onenoc-image-registry/{data,auth}
2. Install htpasswd utility to setting up basic authentication

        sudo apt install apache2-utils -y
3. Navigate to auth folder

        cd onenoc-image-registry/auth
4. Create user. Enter password when asked

        htpasswd -Bc registry.password <username>
5. Create docker-compose.yml file

```
version: '3'

services:
  registry:
    image: registry:2
    ports:
    - "5000:5000"
    environment:
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/registry.password
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
    volumes:
      - ./auth:/auth
      - ./data:/data
```
6. Start the container

        sudo docker-compose up -d
7. Login to the registry

        podman login --tls-verify=false 10.44.100.185:5000
   Provide username and password on prompt
8. Push the image to registry

        podman push --tls-verify=false 78d6b8bc5883 10.44.100.185:5000/onenoc/inventory-web:1.0
9. Add your local registry to podman registries.conf file.

- Linux : Edit the /etc/containers/registries.conf
```
vi /etc/containers/registries.conf
```
- Windows : Login to podman machine
```
podman machine ssh
/etc/containers/registries.conf
```
- Add the local registry
```
[[registry]]
location = "10.44.100.185:5000"
insecure = true
```
- Restart Podman in Linux and restart podman machine in Windows
   

   
