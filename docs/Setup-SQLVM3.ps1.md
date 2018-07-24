---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4160
Published Date: 2014-05-14t17
Archived Date: 2014-08-18t21
---

# setup sqlvm3 - setupvmsql3.ps1

## Description

#############################################################################################

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
       #############################################################################################
 #
 #
 #
 
    
 
     Set-ExecutionPolicy �ExecutionPolicy Unrestricted
 
 
 
 netsh advfirewall firewall add rule name=SQL-SSMS dir=in action=allow enable=yes profile=any
 netsh advfirewall firewall add rule name=SQL-SSMS dir=out action=allow program=any enable=yes profile=any
 netsh advfirewall firewall set rule group="Remote Administration" new enable=yes
 netsh advfirewall firewall set rule group="File and Printer Sharing" new enable=yes
 netsh advfirewall firewall set rule group="Remote Service Management" new enable=yes
 netsh advfirewall firewall set rule group="Performance Logs and Alerts" new enable=yes
 Netsh advfirewall firewall set rule group="Remote Event Log Management" new enable=yes
 Netsh advfirewall firewall set rule group="Remote Scheduled Tasks Management" new enable=yes
 netsh advfirewall firewall set rule group="Remote Volume Management" new enable=yes
 netsh advfirewall firewall set rule group="Remote Desktop" new enable=yes
 netsh advfirewall firewall set rule group="Windows Firewall Remote Management" new enable =yes
 netsh advfirewall firewall set rule group="windows management instrumentation (wmi)" new enable =yes
 
 netsh advfirewall firewall add rule name="Port 5986" dir=in action=allow protocol=TCP localport=5986
 
     [System.Reflection.Assembly]::LoadWithPartialName(�Microsoft.SqlServer.SMO�)  | out-null
     [System.Reflection.Assembly]::LoadWithPartialName(�Microsoft.SqlServer.SMOExtended�)  | out-null
     [System.Reflection.Assembly]::LoadWithPartialName(�Microsoft.SqlServer.SqlWmiManagement�) | out-null
 
 SQLPS
 
 $Name = 'SQL3'
 
 Invoke-Sqlcmd -ServerInstance $Name -Database master -Query "USE [master]
 GO
 EXEC xp_instance_regwrite N'HKEY_LOCAL_MACHINE', N'Software\Microsoft\MSSQLServer\MSSQLServer', N'LoginMode', REG_DWORD, 2
 GO
 "
 
 
 Invoke-Sqlcmd -ServerInstance $Name  -Database master -Query "USE [master]
 GO
 CREATE LOGIN [SQLAdmin] WITH PASSWORD=N'P@ssw0rd', DEFAULT_DATABASE=[master]
 GO
 ALTER SERVER ROLE [sysadmin] ADD MEMBER [SQLAdmin]
 GO
 
 "
 get-Service -ComputerName $Name  -Name MSSQLSERVER|Restart-Service -force
 Enable-PSRemoting -force
`

