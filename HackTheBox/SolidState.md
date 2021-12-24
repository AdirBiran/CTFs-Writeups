# SolidState

## Basic Information

|          |  |
| :---                 |     :---:      |
| Machine Name         | SolidState     |
| Written by           | Adir Biran       |
| IP Address           | 10.10.10.51       |
| Tools and Techniques | Nmap, Gobuster, Misconfigurations, Data Leakage, Cronjob      |

## Method
Enumerating the web and mail services on the machine, intitla access by default credentials and leakage of user's password.  
Escalating the privileges by a modified cronjob python script.  


## Enumeration

#### Nmap

<p align="center">
<img width="386" alt="1" src="https://user-images.githubusercontent.com/21021400/147360651-1dfc2dfa-a294-4a8a-8758-0557bf416e0d.png">
</p>

Nmap - all ports  
<p align="center">
<img width="264" alt="2" src="https://user-images.githubusercontent.com/21021400/147360652-be628011-0da6-4b85-ad68-0c1d6f05573d.png">
</p>

Ports open:  
22 - SSH  
80 - HTTP  
110 - POP3  
119 - NNTP (Network News Transfer Protocol)  
4555 - James Remote Administration Tools  

#### Gobuster

<p align="center">
<img width="328" alt="3" src="https://user-images.githubusercontent.com/21021400/147360653-4808713d-420c-4eed-a6cb-f5d0c50dbc8a.png">
</p>

Gobuster with txt and php files  
<p align="center">
<img width="341" alt="4" src="https://user-images.githubusercontent.com/21021400/147360655-947773e6-b421-4436-8927-d7afc8dbb15f.png">
</p>

#### Manual Enumeration

The website on port 80  
<p align="center">
<img width="960" alt="5" src="https://user-images.githubusercontent.com/21021400/147360656-b5530442-5632-41ea-97d6-9c9e7eb0adff.png">
</p>

/assets  
<p align="center">
<img width="330" alt="6" src="https://user-images.githubusercontent.com/21021400/147360657-57a5feb9-be80-4f77-a220-50485d0cc24e.png">
</p>

/images  
<p align="center">
<img width="315" alt="7" src="https://user-images.githubusercontent.com/21021400/147360659-0b5f6019-0e99-4451-9db0-fac4cd7c5f87.png">
</p>

/README.txt  
<p align="center">
<img width="960" alt="8" src="https://user-images.githubusercontent.com/21021400/147360660-65b064fd-ca93-415e-a5c1-8e50084a3dc5.png">
</p>

/LICENSE.txt  
<p align="center">
<img width="305" alt="9" src="https://user-images.githubusercontent.com/21021400/147360661-680e5a76-6a8c-42a6-aa63-d51a3aad7ec7.png">
</p>

## Exploitation 

Searched for james exploits on metasploit  
<p align="center">
<img width="753" alt="10" src="https://user-images.githubusercontent.com/21021400/147360663-5d931d86-915a-4fe7-a09b-7ece53e199c0.png">
</p>

searchsploit james  
<p align="center">
<img width="463" alt="11" src="https://user-images.githubusercontent.com/21021400/147360665-29e93fba-8aa4-4e4d-b8ba-5c8628703194.png">
</p>

Tried the exploit - but someone needs to login to activate it.  
However, the content of the exploit revealed default credentials for james tool.  
root:root  
<p align="center">
<img width="484" alt="12" src="https://user-images.githubusercontent.com/21021400/147360666-4153e199-03fa-49c0-95e0-a02874383842.png">
</p>

Exploring the services on ports 110, 119, 4555  
<p align="center">
<img width="670" alt="13" src="https://user-images.githubusercontent.com/21021400/147360667-070156f7-c96d-46da-99fb-7cbc177f7d6b.png">
</p>

Port 110 - POP3  
Port 119 - NNTP  
Port 4555 - james remote administration tools  

Default credentials for james remote administration tools:  
root:root  
<p align="center">
<img width="182" alt="14" src="https://user-images.githubusercontent.com/21021400/147360668-58d7d0a8-94d7-457d-a10f-faf40863a716.png">
</p>

List of commands:  
<p align="center">
<img width="423" alt="15" src="https://user-images.githubusercontent.com/21021400/147360670-0ccbfb58-6e6b-4ba9-93b8-bb8bca225c1f.png">
</p>

Listing the users:  
<p align="center">
<img width="84" alt="16" src="https://user-images.githubusercontent.com/21021400/147360671-a2195be6-ecb8-496c-9d27-de1258a4dfaa.png">
</p>

Setting the password "pass" for all users  
```
setpassword james pass
setpassword thomas pass
setpassword john pass
setpassword mindy pass
setpassword mailadmin pass
```
<p align="center">
<img width="124" alt="17" src="https://user-images.githubusercontent.com/21021400/147360673-4db922c1-cd38-466e-9263-4760b44801bc.png">
</p>

With the help of this article on the pop commands:  
https://www.atmail.com/blog/pop-101-manual-pop-sessions/  

We will login to each of the users with the "pass" password and read the emails on port 110.  
```
user james
pass pass
list (to list all mails)
retr <X> command to retrieve specific mail
```

james  
<p align="center">
<img width="243" alt="18" src="https://user-images.githubusercontent.com/21021400/147360674-4f55f152-776d-47de-9d48-6e62ef3ffbb9.png">
</p>

thomas  
<p align="center">
<img width="239" alt="19" src="https://user-images.githubusercontent.com/21021400/147360675-14802916-2c13-42f0-b027-290b5eec6eba.png">
</p>

john  
<p align="center">
<img width="242" alt="20" src="https://user-images.githubusercontent.com/21021400/147360677-bf4f151f-c29a-420e-9626-38e04e99a54f.png">
</p>

john's mail  
<p align="center">
<img width="474" alt="21" src="https://user-images.githubusercontent.com/21021400/147360678-9a3fdf18-fd35-446e-93e4-5a7ac3b0e5ce.png">
</p>

mindy  
<p align="center">
<img width="238" alt="22" src="https://user-images.githubusercontent.com/21021400/147360679-8efb5ba4-60ff-452b-89a2-e30cd9c64e23.png">
</p>

mindy's first mail  
<p align="center">
<img width="473" alt="23" src="https://user-images.githubusercontent.com/21021400/147360681-be22013e-331e-465a-b713-e928b9882916.png">
</p>

mindy's second mail  
<p align="center">
<img width="470" alt="24" src="https://user-images.githubusercontent.com/21021400/147360682-7fcea5b2-cdc8-4186-9284-42472c0fd987.png">
</p>

mindy's credentials found:  
mindy:P@55W0rd1!2@  

mailadmin  
<p align="center">
<img width="241" alt="25" src="https://user-images.githubusercontent.com/21021400/147360683-1359fbc5-f6c5-4fa1-85e3-4b3f25df6a73.png">
</p>

Trying the credentials for the SSH service  
<p align="center">
<img width="328" alt="26" src="https://user-images.githubusercontent.com/21021400/147360684-24814825-0326-4363-9b29-cd20bc03369c.png">
</p>

We're in as mindy.  

## Privilege Escalation (mindy -> root)

However, we're in a restricted bash, meaning many commands will not work.  
<p align="center">
<img width="137" alt="27" src="https://user-images.githubusercontent.com/21021400/147360685-a12dfbce-e465-4793-8fe4-56b6198963b8.png">
</p>

Following this article about escaping restricted bash  
https://www.hacknos.com/rbash-escape-rbash-restricted-shell-escape/  
```
ssh mindy@10.10.10.51 -t "bash --noprofile"
```
<p align="center">
<img width="234" alt="28" src="https://user-images.githubusercontent.com/21021400/147360687-614c5d2c-c82f-4d02-85f2-498ca74503f6.png">
</p>

Users on the machine  
<p align="center">
<img width="351" alt="29" src="https://user-images.githubusercontent.com/21021400/147360690-81ef37ee-c8c7-4f3d-bb41-0755374218f4.png">
</p>

sudo -l  
<p align="center">
<img width="252" alt="30" src="https://user-images.githubusercontent.com/21021400/147360691-8c9fb476-a94a-46df-8668-781bae1ea4b7.png">
</p>

suid files  
<p align="center">
<img width="343" alt="31" src="https://user-images.githubusercontent.com/21021400/147360693-1c3f0030-a7ee-4c73-b8f4-4d29742920f3.png">
</p>

After more enumeration, looking in the home directories, var, logs, backups etc,  
instersting python file was found inside /opt directory  
<p align="center">
<img width="261" alt="32" src="https://user-images.githubusercontent.com/21021400/147360694-1adf539a-ac80-4bc2-bfca-c80e3c4ad4c7.png">
</p>

After thinking for a while on how to target these files easily,  
constructed the following command:  
```
find / -type f -perm /002 -name "*.py" 2>/dev/null
```
looking inside the machine's root directory (/) for any file (-type f) with the world writeable permission (-perm /002) and python extension (-name "\*.py"), redirecting errors to null (2>/dev/null).  
<p align="center">
<img width="423" alt="33" src="https://user-images.githubusercontent.com/21021400/147360695-e12d1942-8064-47bd-abd1-0c68848d8316.png">
</p>

The content of tmp.py  
<p align="center">
<img width="275" alt="34" src="https://user-images.githubusercontent.com/21021400/147360696-70823fc5-ab5d-4802-9750-eb248e960482.png">
</p>

As we can see, the script deletes the content of /tmp directory.  
We can assume there is a cron job running, but to make sure we will create a file in /tmp directory, monitor the time and check if it gets deleted.  
<p align="center">
<img width="273" alt="35" src="https://user-images.githubusercontent.com/21021400/147360697-2ceb83f7-79ad-448e-b56f-0bf163ea0a54.png">
</p>

The file indeed got deleted.  
After exploring it a bit more, apparently the script runs every 3 minutes.  
Let's modify it to get a root shell.  
<p align="center">
<img width="167" alt="36" src="https://user-images.githubusercontent.com/21021400/147360698-7aab267c-fa29-40b6-89cf-581d7dabe02e.png">
</p>

When the script will be executed by a cron job, it will add a SUID on /bin/bash, allowing us to launch a root shell.  
Confirming the SUID bit is set  
<p align="center">
<img width="294" alt="37" src="https://user-images.githubusercontent.com/21021400/147360700-595e2ba8-6c7e-4453-8da8-d6b903e64793.png">
</p>

Getting a root shell  
<p align="center">
<img width="261" alt="38" src="https://user-images.githubusercontent.com/21021400/147360703-d87f70d4-593d-4c53-a390-af69ca064968.png">
</p>

And we're root.
