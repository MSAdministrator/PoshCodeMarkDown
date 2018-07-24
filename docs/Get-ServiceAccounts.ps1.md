---
Author: glenn sizemore glnsize
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 503
Published Date: 
Archived Date: 2008-11-23t03
---

# get-serviceaccounts - 

## Description

script will help discover any service accounts that are currently being used.  i have been using this script for about a month in production.  while your mileage may vary this script is strictly read only, thus -confirm, and -whatif are not supported.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 #
 #
 #
 
 param($target,
 [string]$Path,
 [switch]$verbose,
 [System.Management.Automation.PSCredential]$credential = ($null)
 )
 begin 
 {
     $SvcAccounts = @()
 
     $ErrActionSave = $ErrorActionPreference 
     $Verbosesave = $VerbosePreference
     $warningSave = $WarningPreference
 
     if ($credential)
     {
         Write-Host "there is a known bug using -credential in V1..."
         $gwmiquery = {Get-WmiObject -Class Win32_Service -ComputerName $computer -Credential $credential `
             -property name, startname, caption, StartMode `
             -filter 'NOT Startname LIKE "%NT AUTHORITY%" AND NOT Startname LIKE "LocalSystem" AND NOT Startname LIKE "ASPNET"'}
     }
     else
     {
         $gwmiquery = {Get-WmiObject -Class Win32_Service -ComputerName $computer -property name, startname, caption, StartMode `
             -filter 'NOT Startname LIKE "%NT AUTHORITY%" AND NOT Startname LIKE "LocalSystem" AND NOT Startname LIKE "ASPNET"'}
     }
 
     if ($Verbose) {$VerbosePreference = "Continue"; $WarningPreference = "Continue"}
 
     function ProcessTarget ($computer) 
     {
         $obj = @()
 
         $ErrorActionPreference = "SilentlyContinue" 
 
         Write-Verbose "querying $computer ..."
 
         $services = &$gwmiquery 
 
         $ErrorActionPreference = $ErrActionSave 
 
         Write-Verbose " $($services.count) services located on $computer using "
         foreach ($service in $Services)
         {
 
             switch -regex ($Error[0].Exception) 
             {
                 "The RPC server is unavailable"
                 {
                     Write-warning "RPC Unavailable on $computer"
                     $obj += "" | Select @{e={$computer};n='Target'},@{e={"RPC_Unavalable"};n='SvcName'}
                     continue
                 } 
                 "Access denied"
                 {
                     Write-warning "Access Denied on $computer"
                     $obj += "" | Select @{e={$computer};n='Target'},@{e={"Access_Denied"};n='SvcName'}
                     continue
                 }
                 "Access is denied"
                 {
                     Write-warning "Access Denied on $computer"
                     $obj += "" | Select @{e={$computer};n='Target'},@{e={"Access_Denied"};n='SvcName'}
                     continue
                 }
                 $null 
                 {
                     $obj += "" | Select @{e={$computer};n='Target'},
                     @{e={$service.name };n='SvcName'},
                     @{e={$service.startname };n='SvcAcccount'},
                     @{e={$service.caption };n='SvcDes'},
                     @{e={$service.StartMode };n='StartMode'}
                 } 
             }
             $Error.clear()
         }
         return $obj
     } 
 }
 process 
 {
     $Targets = @()
     if ($Path){$Targets = Get-Content $Path}
     if ($Target){$Targets += $target}
 
     foreach ($computer in $Targets) 
     {
         ping-host -HostName $computer -Count 1 -Quiet -Timeout 1| Where-Object { $_.loss -ne 100} | `
         ForEach-Object {$SvcAccounts += ProcessTarget $_.host}
     }
 
 }
 End 
 {
     $ErrorActionPreference = $ErrActionSave
     $VerbosePreference = $Verbosesave
     $WarningPreference = $warningSave
     Write-Output $SvcAccounts
 }
`

