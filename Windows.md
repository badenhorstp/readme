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

## Disable Windows firewall
```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled false
```

## Start Windows 2019 Core docker image with IIS and web mangement
```powershell
docker run -it -h <host name> --name <container name> -p 80:80 -p 8172:8172 <image name> powershell.exe
```

## Install Kubernetes on Windows 2019 Core
```powershell
Install-PackageProvider -Name NuGet -Force
Install-Module -Name PowerShellGet -Force
Install-Script -Name 'install-kubectl' -Scope CurrentUser -Force
```

## Enable PowerShell remote access
1. Enable Remote Management Service on remote machine
```powershell
Enable-Remoting -Force
Set-Item wsman:\localhost\client\trustedhosts *
Restart-Service WinRM
Test-WsMan <remote host>
```
2. Connect to remote host
```powershell
Enter-PSSession -ComputerName <remote host> -Credential <username>
```

## Update path environment variable
```powershell
[System.Environment]::SetEnvironmentVariable("Path",$env:Path,";<new path>","Machine")
```

## Disable monitor time out on Windows 2019 core
```powershell
powercfg /x monitor-timeout-dc 0
powercfg /x monitor-timeout-ac 0
```
