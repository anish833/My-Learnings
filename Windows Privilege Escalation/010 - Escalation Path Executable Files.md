### Check for executable files with PowerUp
Open the command prompt in the powerup directory and run
```powershell
# Open Powershell
powershell -ep bypass

# Run PowerUp
. .\PowerUp.ps1

# Invoke powerup
Invoke-AllChecks
```

You can see the output as following
![[Pasted image 20210718165731.png]]

### Check with Access Chk
Run the command
```powershell
C:\Users\User\Desktop\Tools\Accesschk\accesschk64.exe -wvu "C:\Program Files\File Permissions Service"
```

![[Pasted image 20210718170023.png]]

### Running Malicious Script
Now that we have a service to exploit. Simple copy the previous c code that we used to gain access with registry and replace it with the vulnerable service here

[[009 - Escalation Path Registry#^db55e3]]

Now start the service in our case it is "filepermsvc"
```powershell
sc start filepermsvc
```