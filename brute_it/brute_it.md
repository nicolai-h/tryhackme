# Brute It

## Reconnaissance
Let's start the enumeration with scanning the ports, which we can do with ```nmap```. This can be done with the command ```nmap -A -T4 <ip>```.

![nmap](https://github.com/nicolai-h/tryhackme/blob/main/brute_it/images/nmap.png)

As we can see there are two ports open, port 22 for ssh and port 80 for http. The SSH version running is OpenSSH 7.6p1 and the server is an Apache server running on ubuntu. For further enumeration we can use ```ffuf``` with the command ```ffuf -w /usr/share/wordlists/dirb/big.txt -u "http://10.10.82.235/FUZZ"```.

![ffuf](https://github.com/nicolai-h/tryhackme/blob/main/brute_it/images/fuzzing.png)

The output shows us that there is an hidden directory called admin. When entering the website at ```<ip>/admin``` we see that there is an login panel.

![login](https://github.com/nicolai-h/tryhackme/blob/main/brute_it/images/login_panel.png)
  
A quick look at the source page tells us that the username is admin and that John is the name of a person on the team.

![source](https://github.com/nicolai-h/tryhackme/blob/main/brute_it/images/source_page.png)

## Getting a shell
Let's try to login to the admin panel, we can brute-force the login with ``hydra``, but first we want to know what the request looks like. A quick look with the firefox devs tools shows us what the request looks like.

![dev](https://github.com/nicolai-h/tryhackme/blob/main/brute_it/images/firefox_dev_tools.png)

Armed with the information we need we can brute-force the login with ```hydra``` with the command ```hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.82.235 http-post-form "/admin/:user=^USER^&pass=^PASS^:Username or password invalid"```

![hydra](https://github.com/nicolai-h/tryhackme/blob/main/brute_it/images/hydra.png)

We now have the username and password and can login into the admin panel. We are greeted with a message and an RSA private key. Let's copy the private key and try to brute-force it. First we will need to change its format, which we will do with ```ssh2john``` with the command ```/usr/share/john/ssh2john.py brute_rsa > brute``` after that we can crack it with ```John the Ripper``` with the command ```john brute --format=SSH --wordlist=/usr/share/wordlists/rockyou.txt```

![ripper](https://github.com/nicolai-h/tryhackme/blob/main/brute_it/images/brute_force.png)

Now that we have the username and password we can log in via SSH, this can be done with the command ```ssh -i <rsa_key> <name>@<ip>```. After  logging in via SSH we will directly find the user flag.

![ssh](https://github.com/nicolai-h/tryhackme/blob/main/brute_it/images/ssh.png)

## Privilege Escalation
Now that we have gotten into the computer it's time for privilege escalation, so let's do some enumeration.
```sudo -l``` shows us that we can use ```cat``` as root, we can thus simply get the root flag with ```sudo cat /root/root.txt```.

![priv_esc](https://github.com/nicolai-h/tryhackme/blob/main/brute_it/images/priv_esc.png)
