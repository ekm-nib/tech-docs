## How to setup Podman and run container in RHEL

1. Install Podman
2. Install Python
3. Install pip
4. Install podman-compose
5. Create docker-compose.yml
6. Run podman-compose up -d
7. If application not accessible outside the host, add application port in firewall
login as root user
```
su 
```
Chek the firewall settings for adding ports
```
firewall-cmd --list-all
```
Add the application port to settingd.
```
firewall-cmd --zone=public --permanent --add-port <port>/tcp
```
Reload the firewall
```
firewall-cmd --reload
```
