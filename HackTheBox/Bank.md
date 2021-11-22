# Bank

### Machine Information
bank
easy
22.11.2021

### Enumeration
<img width="378" alt="1-nmap" src="https://user-images.githubusercontent.com/21021400/142859505-5c1f5863-cea4-4c9c-96dc-828f0f02158c.png">

editing hosts file with the domain
10.10.10.29	bank.htb

checking the website leads to login page
<img width="725" alt="2-website" src="https://user-images.githubusercontent.com/21021400/142859542-43713304-eea0-48cf-a54e-e8cfc3497508.png">

running gobuster twice:
once with directory search
*3
![]()

second with .php extensions search
*4
![]()

assets directory did not lead to something interesting
uploads directory wasn't reachable
inc directory had a few php files, thought nothing useful.
5*
![]()

trying to investigate the php files from the gobuster results,
we will try reach index.php file with the help of burpsuite.

Opened burpsuite, turned on foxyproxy and entered
http://bank.htb/index.php to intercept the request and response.
in burpsuite we will intercept the response to this request and forward the request.
*6
![]()

changing the 302 to 200 in the reponse and forwarding it, will result in the following index.php page on the web:
*7
![]()
*8
![]()

not much to see here, but following this method we of changing the 302 status to 200 we'll investigate the support.php file as well.
*9
![]()
we reached a page which contains a file upload, so lets try get a reverse php shell.
10*
![]()

### Exploitation

apparantly the php extension is not allowed to upload.
*11
![]()

checking the source code of the support.php page, lead to this comment:
*12
![]()

lets try the .htb extension instead.
*13
![]()

looks like the shell.htb was accepted.
*14
![]()

lets open a netcat listener and nevigate to bank.htb/uploads/shell.htb to execute the reverse shell.
and we're in as www-data
*15
![]()

### Privilege Escelation
after basic manual enumeration of suid files, executables, crontab and a few more, decided to run linpeas for faster results.
getting linpeas from the local machine to the /tmp directory and chaning its permissions.
*16
![]()

linpeas results showed the /etc/passwd file is writeable.
*17
![]()

checking the file permissions:
*18
![]()

on the local machine we'll create a password for the /etc/passwd file using the command:
perl -le 'print crypt("pass", "anything")'
*19
![]()

adding the new root user through the /etc/passwd file:
echo "root2:an5CaYOk82Zq6:0:0:root2:/root:/bin/bash" >> /etc/passwd
*20
![]()


as we can see, the new user was added successfully.
*21
![]()

chaging to the root2 user using the "pass" password.
*22
![]()

and we're root.






