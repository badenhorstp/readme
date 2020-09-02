## Install Docker EE on Windows 10
1. Install DockerMsftProvider
```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```
2. Change c:\Program Files\WindowsPowerShell\Modules\DockerMsftProvider\1.0.0.8\DockerMsftProvider.psm1
```powershell
# if ((Get-CimInstance Win32_Operatingsystem | Select-Object -expand Caption) -like "*Windows 10*")
# {
#     ThrowError -CallerPSCmdlet $PSCmdlet `
#                 -ExceptionName "InvalidOperationException" `
#                 -ExceptionMessage "Docker Engine - Enterprise is not supported on Windows 10 client. See https://aka.ms/docker-for-windows instead." `
#                 -ErrorId "RequiresWindowsServer" `
#                 -ErrorCategory InvalidOperation
#     return
# }

#$containerExists = Get-WindowsFeature -Name Containers
$containerExists = Get-WindowsOptionalFeature -FeatureName Containers -Online

#if($containerExists -and $containerExists.Installed)   
if($containerExists -and $containerExists.State -eq "Enabled")

#Install-WindowsFeature containers
Enable-WindowsOptionalFeature -FeatureName Containers -Online

```
3. Install Docker
``` powershell
Install-Package -Name Docker -ProviderName DockerMsftProvider
```
4. Configure Docker service
```bash
net localgroup docker-users /add
net localgroup docker-users "%username%" /add

sc stop docker
sc config docker binPath="c:\Program Files\Docker\dockerd.exe -G docker-users --exec-opt isolation=process --run-service"
sc start docker
sc query docker
```
5. Setup Docker virtual network
```bash
docker network create --subnet=172.18.0.0/16 phoenixnet
```