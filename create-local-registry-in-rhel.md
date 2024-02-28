## How to create a local registry in RHEL

### Prerequisites
1. Install Podman and httpd-tools

```
yum install -y podman
yum install -y httpd-tools
```
2. Create directories

```
mkdir -p registry/{auth,certs,data}
```
* auth - for storing creadential
* certs - for SSL certificates
* data - for storing the image


3. Create username and password for accessing registry

Create username and password using `htpasswd` tool
```
htpasswd -bBc registry/auth/htpasswd registryuser registryuserpassword
```
* __b__ provides the password via command
* __B__ stores the password using Bcrypt encryption
* __c__ creates the file
* registryuser is __Username__
* registryuserpassword is __Password__

4. Secure the regisry using self signed certificate

```
openssl req -newkey rsa:4096 -nodes -sha256 -keyout registry/certs/domain.key -x509 -days 365 -out registry/certs/domain.crt
```
* __req__ tells OpenSSL to generate and process certificate requests.
* __-newkey__ tells OpenSSL to create a new private key and matching certificate request.
* __rsa:4096__ tells OpenSSL to generate an RSA key with 4096 bits.
* __-nodes__ tells OpenSSL there is no password requirement for the private key. The private key will not be encrypted.
* __-sha256__ tells OpenSSL to use the sha256 to sign the request.
* __-keyout__ tells OpenSSL the name and location to store new key.
* __-x509__ tells OpenSSL to generate a self-signed certificate.
* __-days__ tells OpenSSL the number of days the key pair is valid for.
* __-out__ tells OpenSSL where to store the certificate.

__Note__: If the registry is not secured using TLS, the insecure setting ```insecure = true``` in the /etc/containers/registries.conf file may have to be configured for the registry

5. Create docker-compose.yml file

```
version: '3'

services:
  registry:
    image: registry:2
    ports:
      - "8005:5000"
    env_file:
      - .env
    volumes:
      - ./auth:/auth:z
      - ./data:/var/lib/registry:z
      - ./certs:/certs:z
```
## .env file
```
REGISTRY_AUTH=htpasswd
REGISTRY_AUTH_HTPASSWD_REALM=Registry
REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd
REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/data
REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt
REGISTRY_HTTP_TLS_KEY=/certs/domain.key
REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true

```
6. Start Registry

```
podman-compose up -d
```
Verify registry 
```
curl https://hostname:5000/v2/_catalog
```
Verify certificate
```
openssl s_client -connect <servername>:5000 -servername <servername>
```
7. Configure registry in Podman

```
nano /etc/containers/registries.conf

```
Add the registry
```
[[registry]]
location = "hostname:port"
insecure = true
```
Set insecure to true if registry is not secured by TSL as per step 4
8. Push image to registry
  * Login to registry

    ```
    podman login <hostname>:5000
    ```
    Enter username and password on prompt
  * Push the image

   ```
   podman push <image id> <registry hostname>:<port>/folder/image_name:tag
   ```
Set ```tls-verify=false``` if registry is not secured 

9. Verify image in registry

```
curl -u <username>:password https://<registry host>:<port>/v2/_catalog
```
10. Pull image from registry
```
podman pull <hostname>:<port>/folder/image_name:tag
```

__SOURCE__: ```https://www.redhat.com/sysadmin/simple-container-registry```
