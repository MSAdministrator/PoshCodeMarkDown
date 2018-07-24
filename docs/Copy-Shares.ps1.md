---
Author: themoblin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4630
Published Date: 2014-11-22t13
Archived Date: 2014-02-09t09
---

# copy shares - 

## Description

copies shared folders from one server to another. run on the new server (as administrator).

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 $oldserver = "hostname"
 $excludes = @("DLOAgent","faxclient","CertEnroll","Resources$","FxsSrvCp$","tsweb","NETLOGON","WSUSTemp","prnproc$","SYSVOL","Address","tsclient","WsusContent","UpdateServicesPackages","print$")
 $shares = get-wmiobject -computername $oldserver -Class win32_share|where-object {($_.type -eq 0) -and ($excludes -notcontains $_.name)}
 
 foreach ($share in $shares)
  {
  	if ($share.path -like "D:*")
  	    {
 		$newntfspath = $share.path.replace("D:\","C:\")
 		Write-Host -foregroundcolor Green "Destination folder" $newntfspath
 		}
 	else {  $newntfspath = $share.path
 		Write-Host -foregroundcolor Green "Destination folder" $newntfspath
 		}
 
 
 	    if (!(Test-Path $newntfspath))
 		{
 		Write-Host -foregroundcolor Green "Folder $newntfspath does not exist"
 		Write-Host -foregroundcolor Green "Starting robocopy job"
 		robocopy \\$oldserver\$($share.name) $newntfspath /W:5 /R:2 /XO /E /PURGE /SEC /V /NFL /LOG:C:\RobocopyLogs\$($share.name).txt
 		}
 	else { Write-Host -foregroundcolor Red "Folder $newntfspath Already exists"
 		Write-Host -foregroundcolor Red "Skipping this folder"
 		Write-Host -ForegroundColor Red "Creating error log"
 		New-Item -force -ItemType File -Path C:\RobocopyLogs\$($share.name)_Fail.txt
 		Set-content -Value "Location already exists, Robocopy was not attempted" -Path C:\RobocopyLogs\$($share.name)_Fail.txt
 		}
 
         $sourceperms = get-wmiobject -computername $oldserver -Class win32_LogicalShareSecuritySetting | Where-Object {$_.name -eq $share.name}
         $sourceperms = $sourceperms.GetSecurityDescriptor().Descriptor
 
 
         if (!(get-wmiobject -Class Win32_Share |Where-Object {$_.name -eq $share.name}))
             {
             Write-Host -ForegroundColor Green "Creating shared folder"
             $newshare = [wmiclass]"Win32_Share"
             $newshare = $newshare.Create($newntfspath,$share.name,0,$null,$null,$null,$sourceperms)
 		    }
         else { Write-Host -foregroundcolor RED "Shared folder already exists"
              }
  }
`

