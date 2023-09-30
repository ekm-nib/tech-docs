### How to install OPENVPN in Amazon Linux AWS

1. Update yum

```
sudo yum update
```
2. Install epel repository
```
sudo amazon-linux-extras install epel
```
3. Install openvpn
```
sudo yum install openvpn
```
4. Check installation
```
openvpn --version
```
5. Create cliengt configuration
```
sudo nano /etc/openvpn/client.conf
```
COPY the .ovpn file contents to client.conf file
6. Connect the client
```
sudo openvpn --config /etc/openvpn/client.conf 
```
