How to set up OpenVPN server in Mikrotik router ?
----

### 1. Create certificates

  a. CA certificate
  
    -> Select System => Select certificates => Click Add
    -> Name: ca
    -> Common name : ca
    -> Region details are optional.
    -> Keep other options default.
    -> Select Key Usage tab.
    -> Check crl sign and key cert. sign and uncheck other options.
    -> Click Apply.
    -> Click Sign.
    -> Select Certificate : ca
    -> Enter CA CRL host : 127.0.0.1
    -> Click Start to sign.
    -> CA certificate is created now.
    -> If the Trusted flag is not yes check the Trusted box after double clicking key.
    
  b. Server certificate
  
    -> Select System => Select certificates => Click Add
    -> Name: server
    -> Common name: server
    -> Region details are optional.
    -> Keep other options default.
    -> Select Key usage tab
    -> Check digital signature, key encipherment and tls server. Uncheck all other options.
    -> Click apply
    -> Click Sign 
    -> Select CA : ca
    -> Click start to sign the certificate.
    -> Server certificate is created now.
    -> If the Trusted flag is not yes check the Trusted box after double clicking key.
    
  c. Client certificate
    
    -> Select System => Select certificates => Click Add
    -> Name: client
    -> Common name: client
    -> Region details are optional.
    -> Keep other options default.
    -> Select Key usage tab
    -> Check tls client. Uncheck all other options.
    -> Click apply
    -> Click Sign 
    -> Select CA : ca
    -> Click start to sign the certificate.
    -> Server certificate is created now.
    -> Trusted flag is not required for client certificate.
    
  d. Export the CA and client certificate
  
    -> Double click of ca certificate and click Export option. 
    -> Export and file will be available in File list in Mikrotok router
    -> Double click of client certificate and click Export option. 
    -> Export with passphrase and file will be available in File list in Mikrotok router

### 2. Create IP Pool
  
    -> Select IP => Pool
    -> Click add 
    -> Name : openvpn-pool
    -> Addresses : <start ip address>-<end ip address>
    -> Next pool : none
  
### 3. Create Profile

    -> Select PPP = > Profiles => Add
    -> Name : openvpn-profile
    -> Local Address : Gateway address of openvpn-pool
    -> Remote Address : openvpn-pool
  
### 4. Create scerets
  
  Scretes are VPN users.
  
    -> Select PPP => Secretes => Add
    -> Name : username
    -> Password : password for the user
    -> Service : ovpn
    -> Profie : openvpn-profile
  
  Now the user is created.
  
### 5. OpenVPN configuration
  
    -> Select PPP =. Click OVPN Server
    -> Tick Enabled
    -> Port : 1194 <default> we can use any other port number. Miktotik support only TCP port for OpenVPN
    -> Mode : ip
    -> Default Profile : vpn-profile
    -> Certificate : server
    -> Tick require client certificate
    -> Auth : sha1
    -> Cipher : aes 256
  
### 6. Prepare .ovpn file for client

Sample OVPN file 

```
      #Template ovpn file for client
      client
      dev tun
      proto tcp-client 
      remote <mikrotik wan ip>
      port 1194
      nobind
      persist-key
      persist-tun
      tls-client
      remote-cert-tls server
      <ca>
      COPY CA CERTIFICATE TEXT 
      </ca>
      <cert>
      -----BEGIN CERTIFICATE-----
      COPY CA CERTIFICATE TEXT 
      -----END CERTIFICATE----
      </cert>
      <key>
      -----BEGIN CERTIFICATE-----
      COPY CLIENT CERTIFICATE TEXT 
      -----END CERTIFICATE-----
      </key>
      verb 4
      mute 10
      cipher AES-256-CBC
      auth SHA1
      <auth-user-pass>
      ---username---
      ---password---
      </auth-user-pass>
      auth-nocache
      #-----redirect all traffic through vpn including internet uncomment the next line----
      #redirect-gateway def1
      #-----------------------------------------
      #-----Set DNS ----for VPN-----------------
      #dhcp-option DNS 8.8.8.8

      #-----------------------------------------
      #----Add Static routes through VPN here---
      route a.b.c.d 255.255.0.0
```
### 7. Import this .ovpn file in OpenVPn connect client and connect.
    
