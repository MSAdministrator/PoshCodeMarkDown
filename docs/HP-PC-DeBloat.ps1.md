---
Author: alphasun
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6285
Published Date: 2017-04-08t18
Archived Date: 2017-01-11t06
---

# hp pc debloat - 

## Description

this script is intended for use on hp laptops and desktops with an oem installation of windows. the script will remove hp-installed bloatware and unnecessary software. it has been tested and working on a variety of hp hardware.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 $ErrorActionPreference = "silentlycontinue"
  
 $hpguids = get-wmiobject -class win32_product | Where-Object {$_.Name -like "HP *"} | Where-Object {$_.Name -notmatch "client security manager"} | Where-Object {$_.Name -notmatch "HP Hotkey Support"}
 foreach($guid in $hpguids){
     $id = $guid.IdentifyingNumber
      write-host ""$guid.Name" is being removed."
      &cmd /c "msiexec /uninstall $($id) /qn /norestart"
     }
  
 Stop-Process -Name DPClientWizard -Force
  
 $clientmanager = get-wmiobject -class win32_product | Where-Object {$_.Name -like "HP *"} | Where-Object {$_.Name -match "client security manager"}
 foreach($guid in $clientmanager){
     $id = $guid.IdentifyingNumber
      write-host ""$guid.Name" is being removed."
      &cmd /c "msiexec /uninstall $($id) /qn /norestart"
     }
  
 $skypeguid = get-wmiobject -class win32_product | Where-Object {$_.Name -like "*Skype*"}
 foreach($guid in $skypeguid){
     $id = $guid.IdentifyingNumber
      write-host ""$guid.Name" is being removed."
      &cmd /c "msiexec /uninstall $($id) /qn /norestart"
     } 
  
 $foxitguid = get-wmiobject -class win32_product | Where-Object {$_.Name -like "Foxit *"}
 foreach($guid in $foxitguid){
     $id = $guid.IdentifyingNumber
      write-host ""$guid.Name" is being removed."
      &cmd /c "msiexec /uninstall $($id) /qn /norestart"
     } 
  
 $MSSecEss = Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\* | Where-Object {$_.DisplayName -like "Microsoft Security Essentials"} | Select DisplayName,UninstallString
 foreach($item in $MSSecEss){
      write-host ""$item.DisplayName" is being removed."
      cmd /c $item.UninstallString /u /s
     } 
  
 Get-ChildItem HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall | Where-Object {$_.GetValue('DisplayName') -eq 'HP Theft Recovery'} | Remove-Item
        
 Stop-Process -Name sidebar -Force
 Remove-Item "C:\Program Files\Windows Sidebar\Gadgets\DPIDCard.Gadget" -Recurse -Force
 & "C:\Program Files (x86)\PDF Complete\uninstall.exe" /x /s
 Remove-Item "C:\Users\Public\Desktop\Box offer for HP.lnk" -Force
 Remove-Item "C:\Users\Public\Desktop\Skype.lnk" -Force
 Remove-Item "C:\Program Files (x86)\Online Services\" -Recurse -Force
 Remove-Item "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Communication and Chat" -Recurse -Force
 Remove-Item "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\HP Help and Support" -Recurse -Force
 Remove-Item "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Security and Protection" -Recurse -Force
  
 write-host "All HP Bloatware has been removed."
 
 Write-Host -NoNewLine "Press any key to continue..." -ForegroundColor Blue -BackgroundColor Gray
 $null = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
`

