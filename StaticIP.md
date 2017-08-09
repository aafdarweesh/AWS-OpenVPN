# Project Title

Static IP

## Getting Started

The steps to assign specific ip address to the clients (according to their certificate)

### Steps
edit the server.conf file

```
sudo su
nano /etc/openvpn/server.conf
```
uncomment or add these lines
```
client-config-dir ccd/
route 10.8.0.0 255.255.255.248
```
check that these lines are uncommented (remove '#' before it)
```
server 10.8.0.0 255.255.255.0

ifconfig-pool-persist ipp.txt
``` 
as there are some problems may occur in the android and windows devices because of the static ip (as I tried)
uncomment the following line 
```
topology subnet
```

save the file and go to /etc/openvpn
```
cd /etc/openvpn
``` 
add new directory
```
mkdir ccd
cd ccd
```
here you will create files for the clients that you want to have a static ip
make sure that the name of the client certificate (name.ovpn) is the same name as the new file name

```
nano name
```

in the name file write the following line
```
ifconfig-push 10.8.0.10 255.255.255.0
```
the first ip is the ip for the client (static one) the other one is for the server itself


restart the server and everything will work fine 
```
service openvpn restart
```

## Authors

* **Ahmed Darwish** - *Initial work* - [Ahmed Darwish](https://gitlab.com/aafdarweesh)

See also [Ahmed Darwish](https://github.com/aafdarweesh) who participated in this project (GitHub account).

## Acknowledgments

* GOOD LUCK BRO !!!!!
