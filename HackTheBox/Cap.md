
# Cap

### Basic Information
Machine name: Cap  
IP: 10.10.10.245  
Written by: Adir Biran  
Date: 27.11.2021  
Tools used: Nmap, Gobuster, Wireshark

## Enumeration
#### Nmap


![8-wireshark](https://user-images.githubusercontent.com/21021400/143682399-3646c0f9-c34c-4372-b7ef-fef555736f2f.png)
![9-ftp-user](https://user-images.githubusercontent.com/21021400/143682401-3887eb6e-831e-46f0-a6df-929b8cf4f8d2.png)
![10-ssh](https://user-images.githubusercontent.com/21021400/143682402-7598bb5b-00d3-4604-9af7-b060e30f4e46.png)
![11-id](https://user-images.githubusercontent.com/21021400/143682403-ec718a03-4f59-4965-b7e5-0067ff9a4717.png)
![12-suid](https://user-images.githubusercontent.com/21021400/143682404-0cd16c9e-afac-4b46-a6f3-cfe65296830f.png)
![13-sudo](https://user-images.githubusercontent.com/21021400/143682405-cc9fe751-4b65-41de-9751-a060518efa2d.png)
![14-caps](https://user-images.githubusercontent.com/21021400/143682406-e061b04b-759a-4a03-86bc-b10cc294e976.png)
![15-capsetuid](https://user-images.githubusercontent.com/21021400/143682407-167ee65e-07b8-45c9-b321-913f7d9bd9a2.png)
![16-root](https://user-images.githubusercontent.com/21021400/143682408-21ad767e-a812-4652-9b38-3f1830e260b5.png)
![2-gobuster](https://user-images.githubusercontent.com/21021400/143682410-f2fa7d07-e971-4498-8106-204330b81324.png)
![3-dashboard](https://user-images.githubusercontent.com/21021400/143682411-936ff350-1aa6-4b66-8914-02b18fb47ca7.png)
![4-ip](https://user-images.githubusercontent.com/21021400/143682412-835abd77-0bc3-4910-90c7-b8bd75eeed0d.png)
![5-netstat](https://user-images.githubusercontent.com/21021400/143682413-b581606c-0b19-493b-ba8f-58b799f05d0e.png)
![6-wireshark4](https://user-images.githubusercontent.com/21021400/143682414-a44323fd-cc99-4313-83d3-325d9bca6548.png)
![7-wireshark0](https://user-images.githubusercontent.com/21021400/143682415-2a8146af-3021-428f-b692-55dcf5ca21d3.png)



<p align="center">
![1-nmap](https://user-images.githubusercontent.com/21021400/143682409-919d248d-fd73-4595-96f3-b38989bd1bcd.png)
</p>

Ports open: 21 (FTP), 22 (SSH) and 80 (HTTP)

#### Gobuster
<p align="center">
![2-gobuster](https://user-images.githubusercontent.com/21021400/143682097-52c24060-05d4-48a3-8bce-95f1cf66dabc.png)
</p>

4 endpoints were found:  
/data  
/capture  
/netstat  
/ip  

#### Website

The main dashboard  
<p align="center">
![3-dashboard](https://user-images.githubusercontent.com/21021400/143682099-f53636e8-f2e3-4085-8a67-2e3c0b2510a4.png)
</p>

/ip  
<p align="center">
![4-ip](https://user-images.githubusercontent.com/21021400/143682100-31bbcf47-0f40-47ba-8008-a87331582025.png)
</p>

/netstat  
<p align="center">
![5-netstat](https://user-images.githubusercontent.com/21021400/143682102-b344d5a9-3a9f-4dd9-b0b0-0f482202a9e9.png)
</p>

/data - redirects to the main dashboard  

Every redirection to /capture redirects to /data/X  
After examining /data/4, all packets were sent/received by our host machine.  

<p align="center">
![6-wireshark4](https://user-images.githubusercontent.com/21021400/143682103-cc14d5d8-ca6f-47da-ac8f-8324f15ed3df.png)
</p>

Checking the rest of the numbers (4 to 0), an interesting file was found at /data/0  

<p align="center">
![7-wireshark0](https://user-images.githubusercontent.com/21021400/143682104-4433852c-9265-45f0-8a6b-b3a1d694fce7.png)
</p>

## Exploitation

After downloading the pcap file and opening it in Wireshark:  
<p align="center">
![8-wireshark](https://user-images.githubusercontent.com/21021400/143682105-d5e711c7-a0a9-4ee8-ab34-d71394e3b8d2.png)
</p>

We got the following credentials for port 21 (ftp):  
nathan:Buck3tH4TF0RM3!  

Checking the FTP with the following credentials:  
<p align="center">
![9-ftp-user](https://user-images.githubusercontent.com/21021400/143682106-e7279467-f2a1-409c-8748-876db72b5617.png)
</p>

Apparantly, we can navigate to any place on the machine as the nathan user on the FTP service.  
In spite of that, nothing useful was found on the FTP, except for the user flag.  

Trying to use the same credentials for the SSH service:  
<p align="center">
![10-ssh](https://user-images.githubusercontent.com/21021400/143682108-78a62d2d-7b95-432d-bb9b-89de7f7c16b1.png)
</p>

## Privilege Escalation (nathan -> root)

Decided to try manual enumeration before running linpeas.  

Nathan's groups  
<p align="center">
![11-id](https://user-images.githubusercontent.com/21021400/143682109-ff31c9fb-2bb0-4886-893b-2ad64bc89c78.png)
</p>

SUID files  
<p align="center">
![12-suid](https://user-images.githubusercontent.com/21021400/143682110-2410e1a2-bfbb-48a2-87c9-571bf1f1d229.png)
</p>

Sudo permissions  
<p align="center">
![13-sudo](https://user-images.githubusercontent.com/21021400/143682111-8af4c86c-2132-450e-a81d-33f0610b90e5.png)
</p>

Tried as well searching the machine for log and backups files, cron jobs, writeable /etc/passwd but nothing was successful.  

At last, the method that worked successfully was capabilities.  
Checking capabilities files on the system using the command:  
**getcap -r / 2>/dev/null**  
<p align="center">
![14-caps](https://user-images.githubusercontent.com/21021400/143682112-7bc0a356-3cf1-4afc-b2f7-8c70893d053c.png)
</p>

Interestingly, python3.8 runs with cap_setuid.  

More information on the cap_setuid can be found on:  
https://man7.org/linux/man-pages/man7/capabilities.7.html  
<p align="center">
![15-capsetuid](https://user-images.githubusercontent.com/21021400/143682113-1e231ca6-f8e7-4fea-b012-e68d339227b3.png)
</p>

After researching google how we can abuse this capability of python to escalate to the root user, found this article:  
https://www.hackingarticles.in/linux-privilege-escalation-using-capabilities/  

All left is typing the following command:  
**/usr/bin/python3.8 -c "import os; os.setuid(0); os.system('/bin/bash')"**  
<p align="center">
![16-root](https://user-images.githubusercontent.com/21021400/143682115-b05f1b0a-afed-4bba-b9d5-f2dfeae65623.png)
</p>

and we're root.
