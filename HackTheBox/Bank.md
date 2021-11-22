# Bank

### Basic Information
Machine name: Bank
IP: 10.10.10.29
Written by: Adir Biran
Date: 22.11.2021

### Enumeration
##### Nmap

<p align="center">
<img width="378" alt="1-nmap" src="https://user-images.githubusercontent.com/21021400/142859505-5c1f5863-cea4-4c9c-96dc-828f0f02158c.png">
</p>

Editing hosts file with the domain
10.10.10.29	bank.htb

<p align="center">
<img width="144" alt="1-hosts" src="https://user-images.githubusercontent.com/21021400/142862369-0dc6daa0-bd2e-4781-96e2-56f8bc6762b7.png">
</p>

Checking the website leads to login page.

<p align="center">
<img width="725" alt="2-website" src="https://user-images.githubusercontent.com/21021400/142859542-43713304-eea0-48cf-a54e-e8cfc3497508.png">
</p>

##### Gobuster
Running gobuster twice:
Once with directory search

<p align="center">
<img width="336" alt="3-gobuster-dir" src="https://user-images.githubusercontent.com/21021400/142859648-7cf44fee-6f1a-4faf-86dd-828e2a52a1c9.png">
</p>

Second with .php extensions search

<p align="center">
<img width="356" alt="4-gobuster-php" src="https://user-images.githubusercontent.com/21021400/142859650-bebf7991-e9c9-45ad-8811-b3820a31a523.png">
</p>

assets directory did not lead to something interesting
uploads directory wasn't reachable
inc directory had a few php files, thought nothing useful.

<p align="center">
<img width="252" alt="5-inc-dir" src="https://user-images.githubusercontent.com/21021400/142859652-152fabf7-7021-470f-ad30-c016bd491966.png">
</p>

Trying to investigate the php files from the gobuster results,
we will try reach index.php file with the help of Burpsuite.

Opening Burpsuite, turning on FoxyProxy on the browser and entering http://bank.htb/index.php to intercept the request and response.
Inside Burpsuite, intercepting the response to this request and forwarding it.

<p align="center">
<img width="393" alt="6-burp" src="https://user-images.githubusercontent.com/21021400/142859653-59d796df-20ae-4a86-a764-4c7f82b30576.png">
</p>

Changing the status code from 302 to 200 in the reponse and forwarding it.

<p align="center">
<img width="559" alt="7-burp" src="https://user-images.githubusercontent.com/21021400/142859654-8a908c6e-284d-4996-8f75-952f9b3d4eb1.png">
</p>

It will result in the following index.php page on the website:

<p align="center">
<img width="325" alt="8-index" src="https://user-images.githubusercontent.com/21021400/142859655-a1dd2582-654e-4a51-8a64-1d9fcc8ac00a.png">
</p>

Not much to see here, but following this method of changing the 302 status to 200 we will investigate the support.php file as well.

<p align="center">
<img width="180" alt="9-support" src="https://user-images.githubusercontent.com/21021400/142859757-8fe484c9-6ccc-481d-b439-b20ac0ed070a.png">
</p>

Reaching a page which contains a file upload, so let's try get a reverse php shell.

<p align="center">
<img width="172" alt="10-shell" src="https://user-images.githubusercontent.com/21021400/142859759-477bfd5e-b1a5-4c05-84c3-a71be6d75b38.png">
</p>

### Exploitation

Apparantly the php extension is not allowed.

<p align="center">
<img width="274" alt="11-shell-fail" src="https://user-images.githubusercontent.com/21021400/142859761-83405bef-5065-4023-87c4-213021905adf.png">
</p>

Checking the source code of the support.php page, lead to this comment:

<p align="center">
<img width="681" alt="12-source-code" src="https://user-images.githubusercontent.com/21021400/142859765-0dff9e60-edb3-4250-9178-767e8217a8ab.png">
</p>

So let's try the .htb extension instead.

<p align="center">
<img width="181" alt="13-shell" src="https://user-images.githubusercontent.com/21021400/142859766-3ac3b4a7-ff4d-4bc6-9fa4-e2c962ab182a.png">
</p>

Looks like the shell.htb was accepted.

<p align="center">
<img width="190" alt="14-shell-success" src="https://user-images.githubusercontent.com/21021400/142859768-16553f85-6eb7-4c06-a0db-2e4a6f5f1936.png">
</p>

Let's open a netcat listener and navigate to bank.htb/uploads/shell.htb to execute the reverse shell.

<p align="center">
<img width="432" alt="15-in" src="https://user-images.githubusercontent.com/21021400/142859770-6b2391ae-167a-4ab6-ba33-9e0a70af6088.png">
</p>

And we're in as www-data.

### Privilege Escelation
After basic manual enumeration of suid files, executables, crontab and a few more, decided to run linpeas for faster results.
Getting linpeas from the local machine to the /tmp directory and chaning its permissions.

<p align="center">
<img width="314" alt="16-linpeas" src="https://user-images.githubusercontent.com/21021400/142859771-b19f2b89-0b97-4079-9184-bed3b5cfdf67.png">
</p>

Linpeas results showed the /etc/passwd file is writeable.

<p align="center">
<img width="373" alt="17-linpeas-results" src="https://user-images.githubusercontent.com/21021400/142859881-9d66c113-6a29-4a4f-b71f-fd4b5d0a70de.png">
</p>

Checking the file permissions:

<p align="center">
<img width="215" alt="18-passwd-permissions" src="https://user-images.githubusercontent.com/21021400/142859884-0df62423-2247-4216-8cd2-c5394ee7380f.png">
</p>

On the local machine creating a password for the /etc/passwd file using the command:
perl -le 'print crypt("pass", "anything")'

<p align="center">
<img width="186" alt="20-pass-generate" src="https://user-images.githubusercontent.com/21021400/142859890-183578c3-69f6-4368-8000-882899969e37.png">
</p>

Adding the new root user through the /etc/passwd file:
echo "root2:an5CaYOk82Zq6:0:0:root2:/root:/bin/bash" >> /etc/passwd

<p align="center">
<img width="354" alt="19-edit-passwd" src="https://user-images.githubusercontent.com/21021400/142859888-8f312cc4-6ace-4bd1-8573-f982d3828b21.png">
</p>

The new user was added successfully.

<p align="center">
<img width="328" alt="21-user-added" src="https://user-images.githubusercontent.com/21021400/142859891-29bb2e37-28dd-4eb4-8998-efbdb7b220cb.png">
</p>

Switching to the root2 user using the "pass" password.

<p align="center">
<img width="118" alt="22-root" src="https://user-images.githubusercontent.com/21021400/142859892-ac945d14-c439-482e-8f25-a9749c590a40.png">
</p>

And we're root.
