#Accessing problem

For the problem of accessing a server even from outside the company (I will write the steps here and going to document them later)
Server A (has a problem ‘an instance), Server B (another working instance

Firstly from AWS-EC2 console:
Stop server A and server B
Detch the volume (From volume option in the left side) of A and attach it to server B ( the type of the attached volume : /dev/sdf)
Start server B
Secondly in Server B :
Ssh to server B (then write in the terminal the following lines 
 mkdir -m 000 /vol-a
mount /dev/xvdf1 /vol-a (it worked well with xvdf1 as i tried others also)
nano /vol-a/etc/ssh/sshd_config (here make the changes that you want to fix the problem in server A)
umount /vol-a
rmdir /vol-a
exit 
Now go and detach the same volume from server B and attach it to server A (/dev/sda1) and try to connect to the server (http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html) related



## Authors

* **Ahmed Darwish** - *Initial work* - [Ahmed Darwish](https://gitlab.com/aafdarweesh)

See also [Ahmed Darwish](https://github.com/aafdarweesh) who participated in this project (GitHub account).

## Acknowledgments

* GOOD LUCK BRO !!!!!
