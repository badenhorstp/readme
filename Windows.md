## Install Docker on Windows 2019 Core
```powershell
Install-Module -Name DockerMsfProvider -Repository PSGallery -Force

Install-Package -Name Docker -ProviderName DockerMsftProvider

Restart-Computer -Force
```

## Install IIS on Windows 2019 Core container
1. Install IIS
```powershell
Install-WindowsFeature -Name Web-Server
```
2. Install ASP.NET
```powershell
Install-WindowsFeature -Name Web-Asp-Net45
```
3. Install IIS Web Management Tool
```powershell
Install-WindowsFeature -Name Web-Mgmt-Service

Set-ItemProperty -Path Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\WebManagement\Server\ -Name 
EnableRemoteManagement -Value 1

Set-Service -Name WMSvc -StartupType Automatic

Start-Service -Name WMSvc

net user iisadmin P@ssword99 /add

net localgroup administrators iisadmin /add
```

## Install Phoenix XM web service
```
ComSetup.exe phoenixws_phoenixxm C:\phoenixws\phoenixxm\webassembly 2003
```

