---
Author: alphasun
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6286
Published Date: 2016-04-08t18
Archived Date: 2016-12-03t15
---

# lenovo pc debloat - 

## Description

this script is intended for use on lenovo laptops with an oem installation of windows. the script will remove lenovo-installed bloatware and unnecessary software. it has been tested on a variety of hardware, however, in its current form it is not complete. i have left some notes at the bottom for additional software that may potentially need to be removed.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 
 $lenguids = get-wmiobject -class win32_product | where-object {$_.Name -like "Lenovo *"} | where-object {$_.Name -notmatch "Auto Scroll"} | where-object {$_.Name -notmatch "USB"}
 foreach($guid in $hpguids){
     $id = $guid.IdentifyingNumber
      write-host ""$guid.Name" is being removed."
      &cmd /c "msiexec /uninstall $($id) /qn /norestart"
     }
 
 $nitroguid = get-wmiobject -class win32_product | Where-Object {$_.Name -like "*Nitro*"}
 foreach($guid in $nitroguid){
     $id = $guid.IdentifyingNumber
      write-host ""$guid.Name" is being removed."
      &cmd /c "msiexec /uninstall $($id) /qn /norestart"
     } 
 
 $tvapsguid = get-wmiobject -class win32_product | Where-Object {$_.Name -like "*ThinkVantage*"}
 foreach($guid in $tvapsguid){
     $id = $guid.IdentifyingNumber
      write-host ""$guid.Name" is being removed."
      &cmd /c "msiexec /uninstall $($id) /qn /norestart"
     } 
 
 $recoveryguid = get-wmiobject -class win32_product | Where-Object {$_.Name -like "*Recovery Media*"}
 foreach($guid in $recoveryguid){
     $id = $guid.IdentifyingNumber
      write-host ""$guid.Name" is being removed."
      &cmd /c "msiexec /uninstall $($id) /qn /norestart"
     } 
  
 $lmcpguid = get-wmiobject -class win32_product | Where-Object {$_.Name -like "*Message Center Plus*"}
 foreach($guid in $lmcpguid){
     $id = $guid.IdentifyingNumber
      write-host ""$guid.Name" is being removed."
      &cmd /c "msiexec /uninstall $($id) /qn /norestart"
     } 
  
 $mfacguid = get-wmiobject -class win32_product | Where-Object {$_.Name -like "*Multifactor Authentication Client*"}
 foreach($guid in $mfacguid){
     $id = $guid.IdentifyingNumber
      write-host ""$guid.Name" is being removed."
      &cmd /c "msiexec /uninstall $($id) /qn /norestart"
     } 
  
 $softexguid = get-wmiobject -class win32_product | Where-Object {$_.Vendor -like "*Softex*"}
 foreach($guid in $softexguid){
     $id = $guid.IdentifyingNumber
      write-host ""$guid.Name" is being removed."
      &cmd /c "msiexec /uninstall $($id) /qn /norestart"
     } 
  
 
  
 $officeguid = get-wmiobject -class win32_product | Where-Object {$_.Name -like "*Microsoft Office*"}
 foreach($guid in $officeguid){
     $id = $guid.IdentifyingNumber
      write-host ""$guid.Name" is being removed."
      &cmd /c "msiexec /uninstall $($id) /qn /norestart"
     } 
  
  
 write-host "Lenovo Communications Utility  is being removed."
 &cmd /c "C:\Program Files\Lenovo\Communications Utility\unins000.exe" /SILENT /NORESTART
 
 write-host "Lenovo RapidBoot  is being removed."
 &cmd /c "C:\Program Files (x86)\Lenovo\RapidBoot HDD Accelerator\Uninstall.exe" /S
 
 write-host "Lenovo SHAREit  is being removed."
 &cmd /c "C:\Program Files (x86)\Lenovo\SHAREit\unins000.exe" /SILENT
 
 
  
  
`

