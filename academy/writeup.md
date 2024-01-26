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
ffuf
```
We found two paths /academy and /phpmyadmin. Let's try the credential we found in phpmyadmin
## SCREENSHOT
Ok it didn't work and it seems to be not vulnerable to sql injection, let's go to /academy. 
## SCREENSHOT
Great we are in, ok we are looking for a way to get a shell, there is file upload we can check it if it is vulnerable to RFI.
## SCREENSHOT
As we saw it is a php app we can use php reverse shell. Before we upload the shell we need to set netcat to listen to any incoming connections.  
Upload the file
## SCREENSHOT
Boom we have access, go to home
```
cd home
ls
grimmie
```
we have grimmie user
```
cd gimmie
ls -la
```
we have interesting file "backup.sh"
