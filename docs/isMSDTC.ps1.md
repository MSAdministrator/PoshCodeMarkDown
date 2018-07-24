---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2027
Published Date: 2010-07-28t12
Archived Date: 2016-11-08t06
---

# ismsdtc.ps1 - 

## Description

checks whether msdtc is enabled for network access. by default msdtc network access is disabled. this feature is needed by sql server linked servers

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 param($computerName=$env:computerName)
 
 
 $is64 = [bool](gwmi win32_operatingsystem -computer $computerName | ?{$_.caption -like "*x64*" -or $_.OSArchitecture -eq'64-bit'})
 $isShell32 = [bool]((Get-Process -Id $PID | ?{$_.path -like "*SysWOW64*"}) -or !([IntPtr]::Size -eq 8))
 
 if ($is64 -and $isShell32)
 {Write-Warning "Unable to open registry keys because $computerName is running an x64 OS. Script must be run from a PowerShell x64 shell" }
 else
 {
     $reg = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey('LocalMachine',$computerName)
     $msdtcKey = $reg.OpenSubKey("SOFTWARE\\Microsoft\\MSDTC")
     $securityKey = $msdtcKey.OpenSubKey("Security")
     new-object psobject -property @{ComputerName=$computerName;AllowOnlySecureRpcCalls=[bool]$msdtcKey.GetValue('AllowOnlySecureRpcCalls');TurnOffRpcSecurity=[bool]$msdtcKey.GetValue('TurnOffRpcSecurity');
                       NetworkDtcAccessAdmin=[bool]$securityKey.GetValue('NetworkDtcAccessAdmin');NetworkDtcAccessClients=[bool]$securityKey.GetValue('NetworkDtcAccessClients');
                       NetworkDtcAccessInbound=[bool]$securityKey.GetValue('NetworkDtcAccessInbound');NetworkDtcAccessOutbound=[bool]$securityKey.GetValue('NetworkDtcAccessOutbound');
                       NetworkDtcAccessTransactions=[bool]$securityKey.GetValue('NetworkDtcAccessTransactions');XaTransactions=[bool]$securityKey.GetValue('XaTransactions')}
 }
`

