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

4. Add application virtual directory, app pool and application
```powershell
New-WebAppPool PhoenixXM
New-WebVirtualDirectory -Site "Default Web Site" -Name phoenixws -PhysicalPath c:\phoenixws
New-WebApplication -Site "Default Web Site" -Name phoenixws/phoenixxm -PhysicalPath c:\phoenixws\phoenixxm -ApplicationPool PhoenixXM
```

## Install Phoenix XM web service
```
ComSetup.exe phoenixws_phoenixxm C:\phoenixws\phoenixxm\webassembly 2003
```

## Disable Windows firewall
```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled false
```

## Start Windows 2019 Core docker image with IIS and web management
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
Enable-PsRemoting -Force
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

## Windows 2019 Core change default shell
```powershell
Set-ItemProperty -Path 'HKLM:\Software\Microsoft\Windows NT\CurrentVersion\WinLogon' -Name Shell -Value 'powershell.exe'
```

## Check status of Windows 2019 Core service
```powershell
Get-WmiObject -Class win32_service | Where-Object {$_.name -eq '<service name>'}
```

## Retreive Windows Release ID
```powershell
Get-ItemProperty -Path 'Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\' -Name ReleaseId
```