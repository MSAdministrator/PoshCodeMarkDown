---
Author: anonymous
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 5519
Published Date: 2016-10-15t21
Archived Date: 2016-01-11t20
---

# disable ssl3 enable tls - 

## Description

creates or updates registry keys to disable ssl3 and enable tls 1.0, 1.1, 1.2

## Comments



## Usage



## TODO



## function

`set-enabledkey`

## Code

`#
 #
 Function Set-EnabledKey([string]$KeyPath,[switch]$Disable)
 {
     if($(Test-Path $KeyPath) -eq $false)
     { 
         md $KeyPath | Out-Null
         Write-Host "`tCreated: $KeyPath" -ForegroundColor Yellow
     }
 
     $_present = Get-ItemProperty $KeyPath -Name "Enabled" -ea 0
 
     if($_present -eq $null)
     {
         if($Disable)
         {
             New-ItemProperty -Path $KeyPath -Name "Enabled" -Value 0 -PropertyType "DWord" | Out-Null
             Write-Host "`t`tCreated: $(Join-Path $KeyPath "Enabled") = 0" -ForegroundColor Red
         }
         else
         {
             New-ItemProperty -Path $KeyPath -Name "Enabled" -Value 1 -PropertyType "DWord" | Out-Null
             Write-Host "`t`tCreated: $(Join-Path $KeyPath "Enabled") = 1" -ForegroundColor Green
         }
     }
     else
     {
         if($Disable)
         {
             Set-ItemProperty -Path $KeyPath -Name "Enabled" -Value "0" | Out-Null
             Write-Host "`t`tUpdated: $(Join-Path $KeyPath "Enabled") = 0" -ForegroundColor Red
         }
         else
         {
             Set-ItemProperty -Path $KeyPath -Name "Enabled" -Value "1" | Out-Null
             Write-Host "`t`tUpdated: $(Join-Path $KeyPath "Enabled") = 1" -ForegroundColor Green
         }
     }
 
 Function Set-DisabledByDefaultKey([string]$KeyPath,[switch]$Disable)
 {
     if($(Test-Path $KeyPath) -eq $false)
     { 
         md $KeyPath | Out-Null
         Write-Host "`tCreated: $KeyPath" -ForegroundColor Yellow
     }
 
     $_present = Get-ItemProperty $KeyPath -Name "DisabledByDefault" -ea 0
 
     if($_present -eq $null)
     {
         if($Disable)
         {
             New-ItemProperty -Path $KeyPath -Name "DisabledByDefault" -Value 1 -PropertyType "DWord" | Out-Null
             Write-Host "`t`tCreated: $(Join-Path $KeyPath "DisabledByDefault") = 1" -ForegroundColor Red
         }
         else
         {
             New-ItemProperty -Path $KeyPath -Name "DisabledByDefault" -Value 0 -PropertyType "DWord" | Out-Null
             Write-Host "`t`tCreated: $(Join-Path $KeyPath "DisabledByDefault") = 0" -ForegroundColor Green
         }
     }
     else
     {
         if($Disable)
         {
             Set-ItemProperty -Path $KeyPath -Name "DisabledByDefault" -Value "1" | Out-Null
             Write-Host "`t`tUpdated: $(Join-Path $KeyPath "DisabledByDefault") = 1" -ForegroundColor Red
         }
         else
         {
             Set-ItemProperty -Path $KeyPath -Name "DisabledByDefault" -Value "0" | Out-Null
             Write-Host "`t`tUpdated: $(Join-Path $KeyPath "DisabledByDefault") = 0" -ForegroundColor Green
         }
     }
 
 Function Set-ProtocolSettings([string]$BasePath,[switch]$Disable)
 {
     if($(Test-Path $BasePath) -eq $false)
     { 
         md $BasePath | Out-Null
         Write-Host "`tCreated: $BasePath" -ForegroundColor Yellow
     }
 
     if($Disable)
     {
         Set-EnabledKey -KeyPath $(Join-Path $BasePath "Client") -Disable
         Set-EnabledKey -KeyPath $(Join-Path $BasePath "Server") -Disable
         Set-DisabledByDefaultKey -KeyPath $(Join-Path $BasePath "Client") -Disable
         Set-DisabledByDefaultKey -KeyPath $(Join-Path $BasePath "Server") -Disable
     }
     else
     {
         Set-EnabledKey -KeyPath $(Join-Path $BasePath "Client")
         Set-EnabledKey -KeyPath $(Join-Path $BasePath "Server")
         Set-DisabledByDefaultKey -KeyPath $(Join-Path $BasePath "Client")
         Set-DisabledByDefaultKey -KeyPath $(Join-Path $BasePath "Server")
     }
 }
 Write-Host "`n`tStarting...`n" -ForegroundColor Cyan
 Set-ProtocolSettings -BasePath "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 3.0" -Disable
 Set-ProtocolSettings -BasePath "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0"
 Set-ProtocolSettings -BasePath "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1"
 Set-ProtocolSettings -BasePath "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2"
 Write-Host "`n`tFinished. Reboot Required.`n" -ForegroundColor Cyan
`

