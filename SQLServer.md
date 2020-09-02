## Install SQL Server 2019 in Windows 2019 Container
```powershell
.\setup.exe /q /action=Install /Features=SQLEngine /instancename=MSSQLSERVER /sqlsvcaccount="sqladmin" /sqlsvcpassword="P@ssword99" /sqlsysadminaccounts="sqladmin" /agtsvcaccount="NT AUTHORITY\Netwo
rk Service" /tcpenabled=1 /iacceptsqlserverlicenseterms /sapwd="P@ssword99" /securitymode=SQL
```