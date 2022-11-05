### Domain Enumeration
![[Pasted image 20210809092456.png]]

In BloodHound there are many ingestors which can be used to run collect information for bloodhound. The most famous and used ingestor is `SharpHound.ps1`. The Sharphound can be run as
```powershell
# Supply data to bloodhound
C:\AD\Tools\BloodHound-master\Ingestors\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All

# The generated archive can be uploaded to the BloodHound Application
# To avoid detection like ATA (Advanced Threat Analytics From Microsoft)
Invoke-BloodHound -CollectionMethod All -ExcludeDC

# Sometimes BloodHound misses the session so for that run
Invoke-BloodHound -CollectionMethod LoggedOn 
```
