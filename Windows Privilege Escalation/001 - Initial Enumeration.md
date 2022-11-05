#### System Enumeration
###### For getting the system information
```bash
systeminfo
```
 ![[Pasted image 20210712195242.png]]

###### Quick Oneliner for OS information
```bash
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
```
 ![[Pasted image 20210712195827.png]]

###### See the patches made in the OS
```bash
wmic qfe
```
 ![[Pasted image 20210712200309.png]]

| Short Hand | Full Form |  
| ----------- | ----------- |  
| WMI | Windows Management Instrumentation  |  
| QFE | Quick Fix Engineering |

###### Oneliner to get patches with specific information
```bash
wmic qfe get Caption,Description,HotFixID,InstalledOn
```
 ![[Pasted image 20210712201823.png]]
 
 ###### To get the drive information
 ```bash
 wmic logicaldisk get caption,description,providername
 ```
  ![[Pasted image 20210712202108.png]]
  
  #### User Enumeration
  
  ###### To get which user are you running as
  ```bash
  whoami
  ```
  ![[Pasted image 20210712202341.png]]
  
  ###### To check the privileges the current user have
  ```bash
  whoami /priv
  ```
  ![[Pasted image 20210712202505.png]]
  
  ###### To check which groups the user is in
  ```bash
  whoami /groups
  ```
  ![[Pasted image 20210712202658.png]]
  
  ###### To list the users on the system
  ```bash
  net user
  ```
  ![[Pasted image 20210712202841.png]]
  
  ###### To get information on specific users
  ```bash
  net user <username>
  ```
  ![[Pasted image 20210712203046.png]]
  
  ###### To check the groups in the system
  ```bash
  net localgroup
  ```
  ![[Pasted image 20210712203313.png]]
  
  ###### To check who is in a specific groups
  ```bash
  net localgroup <groupname>
  ```
 ![[Pasted image 20210712203514.png]]
 
 #### Network Enumeration
 
 ###### To get a detailed information on IP address
 ```bash
 ipconfig /all
 ```
 ![[Pasted image 20210712203921.png]]
 
 ###### To check the arp tables
 ```bash
 arp -a
 ```
 ![[Pasted image 20210712204127.png]]
 
 ###### To check the routing tables
 ```bash
 route print
 ```
 ![[Pasted image 20210712204354.png]]
 
 ###### To see what ports are listening 
 ```bash
 netstat -ano
 ```
 ![[Pasted image 20210712204610.png]]
 
 #### Password Hunting
 
 ###### To get passwords stored in a file of current directory
 ```bash
 findstr /si password *.txt *.ini *.config
 ```
 ![[Pasted image 20210712205139.png]]
 
 #### AV and Firewall Enumeration
 
 ###### To check if windows defender is running
 ```bash
 sc query windefend
 ```
 ![[Pasted image 20210712205641.png]]

| Short Hand | Full Form |  
| ----------- | ----------- |  
| SC | Service Control  |  

###### Checking antivirus programs by viewing all running services
```bash
sc queryex type= service
```
![[Pasted image 20210712210011.png]]

###### To check the firewall configuration
```bash
netsh firewall show config
```
![[Pasted image 20210712210403.png]]