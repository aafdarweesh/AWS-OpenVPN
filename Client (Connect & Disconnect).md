# Clients (Connect & Disconnect)
Here will write in a separate file whenever a client connect (which client "relaying on the ip address") and disconnect.
you can do some modifications to know the exact time (here will just show the how to know), or execute a program whenever a
client is disconnected (also exact client)

## Server configuration 

```
sudo su
cd /etc/openvpn
nano server.conf
```
then add the following lines to the end of the file (to be organized "the order doesn't matter")

```
script-security 2
client-connect /etc/openvpn/statuschange.sh
client-disconnect /etc/openvpn/statuschange.sh

learn-address /etc/openvpn/checkip.sh
```
the last lines :
* client-connect : will execute the program whenever there is a client connected to the server
* client-disconnect : will execute the program whenever there is a client disconnected to the server
* learn-address : when the client connect or disconnect the program will be executed and from here you can know which client
is connected or disconnected

the most important thing now is the mode of the files, because if the a problem occurs while executing one of the programs
this will lead to clients unable to connect to the server


## "statuschange.sh" file
Create this file in the same directory of openvpn ("again it is prefered to put everything together", do whatever you want ;))

```
cd /etc/openvpn
nano statuschange.sh

```
add the next line to the file
```
#!/bin/bash
#python /etc/openvpn/pConnect.py
if [ "$script_type" == "client-connect" ]; then
    ./etc/openvpn/checkip.sh
    echo 'Client Connected' >> /etc/openvpn/Clients
    #here also you can ask to exexute a program ex.(python porgram.py)
    
elif [ "$script_type" == "client-disconnect" ]; then
    ./etc/openvpn/checkip.sh
    echo 'Client Disonnected' >> /etc/openvpn/Clients
fi
# you can add here any commands 
exit 0
```
the last code will check if it is connection or disconnection and will write in file "Clients" that there is a connection or
disconnection 
(you can put a separate file for connection and disconnection to be executed whenever the case occurs "in server.conf" file)

## "checkip.sh" file

I found this file online "I will attach the source" but in this file don't remove the comments even (I tried and it didn't
work)

```
nano checkip.sh
```
inside the file
```
#!/bin/bash
# For A Record:         <action> <hostname> <ttl> <type> <address>
# For PTR Record:       <action> <revaddr> <ttl> <type> <fqdn>
# $1=operation $2=address $3=common_name

# VARIABLES
dnsserver=172.16.1.1
masterzone=maricopa.local
fwdzone=vpn.$masterzone
revzone=16.172.in-addr.arpa
ttl=360
op=$1
addr=$2
revaddr=`echo $addr | sed -re 's:([0-9]+)\.([0-9]+)\.([0-9]+)\.([0-9]+):\4.\3.\2.\1.in-addr.arpa:'`
cn=$3
fqdn=$cn.$fwdzone
dir=/etc/openvpn/scripts/dns
addfile=$dir/add_$addr
delfile=$dir/del_$addr

echo $addr >> /etc/openvpn/Clients

exit 0
```

actually I am not sure why a problem occurse when I try to remove one of the ubove lines (if yo9u know leave it in the comment
please)


### Files modes
this is the most important one as it will lead to a problem in clients connection to the server

firstly create the "Clients" file which will contain the data 

```
nano Clients
```
write anything in it and save


the modes 
```
chmod 777 statuschange.sh
chmod 777 checkip.sh
chmod 777 Clients
```
you might think that there is no need to have '777' but it is ok, will not cause any problem


## Sample output 
here is a sample output "Clients" file

```
Client Connected
10.8.0.10
Client Disonnected
10.8.0.10
Client Connected
10.8.0.10
Client Connected
10.8.0.40
Client Disonnected
10.8.0.10
10.8.0.40
Client Connected
10.8.0.40
```

put in your mind that the next outputs might occur 
```
Client Disonnected
Client Connected
10.8.0.10
Client Disonnected
10.8.0.10
Client Disonnected
Client Connected
10.8.0.10
10.8.0.40
```
the idea is in the connection (client-server) the server knows about the connection immediately, but in disconnection case
the server wait for an amount of time to check the connection (that cause the delay in printing to the file) then after that
time it decides that the client is disconnected


to avoid this problem you can build a program that will take data once the ip is known from the server 
and check if it is connection or disconnection from the other files

you can use (openvpn-status.log) and check it everytime but that solution is not practical

### Test
now if you want to test (I did and recommend)
test it using linux and non-linux systems it will be better ti make sure that your system is working proparly

## Some links that will help
```
https://forums.openvpn.net/viewtopic.php?t=12613

https://forums.openvpn.net/viewtopic.php?t=12613
```

## Contributions
Stackoverflow
Google 
OpenVPN (the least)
Internet 
(Ofcourse joking, just forget about it)

## Authors

* **Ahmed Darwish** - *Initial work* - [Ahmed Darwish](https://github.com/aafdarweesh)

## License

JUST HAVE FUN !!!
## Acknowledgments

* GOOD LUCK !!!
