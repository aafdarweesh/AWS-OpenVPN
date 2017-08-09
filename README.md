# Project Title

AWS-OpenVPN

## Getting Started

Here I will add some sources that I used to setup OpenVPN on AWS (EC2) server, and as I added some codes (and edited others) so I will put the source for the explaination but I will write the
steps only here without any explaination
### Prerequisites

For using AWS and having a server, in this video there are steps to do so, but just for building an instance for AWS the rest will conintue from the other source
```
https://www.youtube.com/watch?v=9BAoJ7JZHr0
```

The main source for setting up 
```
https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-14-04#step-3-—-generate-certificates-and-keys-for-clients
```

### Installing and configuring
After connecting to the server using the 

```
sudo su
apt-get update
apt-get install openvpn easy-rsa
gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz > /etc/openvpn/server.conf
nano /etc/openvpn/server.conf
```
//In the server.conf file :

1) if you found this (in the new version it is already done automatically but just in case)
```
dh dh1024.pem
```
then change it to 
```
dh2048.pem
```

2) uncomment the following lines (uncommenting is by removing the ';' sign, and the lines are not following each other)

```
;push "redirect-gateway def1 bypass-dhcp"

;push "dhcp-option DNS 208.67.222.222"
;push "dhcp-option DNS 208.67.220.220"

;user nobody
;group nogroup

```
save the changes that you have done on server.conf file

Type in the terminal 
```
echo 1 > /proc/sys/net/ipv4/ip_forward
nano /etc/sysctl.conf
```
//In the sysctl.conf file uncomment the following line 
```
#net.ipv4.ip_forward=1
```
in this file also there you can make some changes according to your system as you can configure which ipv(4,6) you want to allow and there are some settings that are really and already noted 
so you find instructions and explaination for the commands in the file, and the same thing also for other system file (most of them end with '.conf')


Then in the terminal write
```
apt-get install ufw
ufw allow ssh
```
in this point there  is two options either to use 1194/udp port (I used and problem occurse with the laptop) or you can use 80/tcp, whatever you are going to choose you have to make sure that 
in the server.conf file it is edited accoriding to your settings and also in the client.conf file

so I will use 80/tcp 

write in the terminal 
```
ufw allow 80/tcp
```
and change the settings in server.conf file by:
```
nano /etc/openvpn/server.conf
```
//In the file itself do the next changes : (to change from 1194/udp to 80/tcp)
```
tcp
;udp

#port 1194
port 80
```
save the changes then change the client.conf version:
```
nano /usr/share/doc/openvpn/examples/sample-config-files/client.conf
```
//In the client.conf file do the next chnages: (to change from 1194/udp to 80/tcp)
```
tcp
;udp

#remote server.public.ip 1194
remote server.public.ip 80
```
and add these lines 
```
#for DNS
script-security 2
up /etc/openvpn/update-resolv-conf
down /etc/openvpn/update-resolv-conf
```

save the changes

in the terminal 
```
nano /etc/default/ufw
```
edit the file by changing "DROP" to "ACCEPT" :
```
DEFAULT_FORWARD_POLICY="ACCEPT"
```

save the changes then in the terminal 
```
nano /etc/ufw/before.rules
```
edit the file by adding the next commands to the beginning of the file 
```
# START OPENVPN RULES
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0] 
# Allow traffic from OpenVPN client to eth0
-A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE
COMMIT
# END OPENVPN RULES
```

save and in the terminal enable the firewall
```
ufw enable
```
to view the ports that you are using (22,1194) or (22,80) and others if you allowed 
```
ufw status
```

###  Creating certificate authority for the server side 

write in the terminal 
```
cp -r /usr/share/easy-rsa/ /etc/openvpn
mkdir /etc/openvpn/easy-rsa/keys
nano /etc/openvpn/easy-rsa/vars
```

//In the vars file you can make the default changes according to your institution
these changes will be in the server certificate and the client certificate as default
edit these lines if you want 
```
export KEY_COUNTRY="US"
export KEY_PROVINCE="TX"
export KEY_CITY="Dallas"
export KEY_ORG="My Company Name"
export KEY_EMAIL="sammy@example.com"
export KEY_OU="MYOrganizationalUnit"
```
and in the terminal write 
```
export KEY_NAME="server"

openssl dhparam -out /etc/openvpn/dh2048.pem 2048
cd /etc/openvpn/easy-rsa
```
write the next line whenever you want to generate a new certificate
```
. ./vars
```

this line only the first time (for the server and don't use it again for the other certificates)
very important not to use it again (as it will delete existing certificates in the system
```
./clean-all
```

to build the certificate authority (also will use it first time and there is no need to use it again
```
./build-ca
```

### Generate the server certificate 

write in the terminal 
```
./build-key-server server
```
write 'y' in the questions that will appear (means you want to build certificate, and you don't need to write a password)
then 
```
cp /etc/openvpn/easy-rsa/keys/{server.crt,server.key,ca.crt} /etc/openvpn
```

Now start your own server 
```
service openvpn start
service openvpn status
```



### Generate certificate for the client

write in the terminal (client1 is refer to the name of the client "the client's certificate", it is better to write real names so not to get lost later on "not the same as the server name"!!!) 
```
cd /etc/openvpn/easy-rsa
. ./vars
./build-key client1
```

then 
```
cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/easy-rsa/keys/client1.ovpn
```

Now you need to copy these files to you local device (use any method you want, in the worst case open the file and copy it manually)
```
/etc/openvpn/easy-rsa/keys/client1.crt
/etc/openvpn/easy-rsa/keys/client1.key
/etc/openvpn/easy-rsa/keys/client1.ovpn
/etc/openvpn/ca.crt
```

### Combine crt,key,ovpn files

In the '.ovpn' file do the next :
```
#ca ca.crt
#cert client.crt
#key client.key

<ca>
(insert ca.crt here)
</ca>
<cert>
(insert client1.crt here)
</cert>
<key>
(insert client1.key here)
</key>
```
as written just copy the content of these files and put them in the 'ovpn' file

## Run the client certificate on linux

go to the directory where you installed '.ovpn' file then write this command 
```
apt-get install openvpn
openvpn --config client.ovpn
```

to use Android, IOS and Windows you will find the instrucations in the main source webpage

## Authors

* **Ahmed Darwish** - *Initial work* - [Ahmed Darwish](https://gitlab.com/aafdarweesh)

See also [Ahmed Darwish](https://github.com/aafdarweesh) who participated in this project (GitHub account).

## Acknowledgments

* GOOD LUCK BRO !!!!!
