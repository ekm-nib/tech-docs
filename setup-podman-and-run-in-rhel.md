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
firewall-cmd --list-all
```
This will show the firewall settings
firewall-cmd --zone=public --permanent --add-port <port>/tcp
This will add the application port. Then reload the firewall
firewall-cmd --reload
