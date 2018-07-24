---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 685
Published Date: 
Archived Date: 2009-01-05t16
---

# iis ftp site creation - 

## Description

automatically creates a local user on server, local directory on server, and virtual directory in iis based on user inputs.  also sets user flags to read/write on directory and sets password options for user

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 "       -------------------------------------------"
 ""
 ""
 ""
 ""
 "       -------------------------------------------"
  
  
  
 $AccountName = Read-Host "Please enter user account name (i.e. krisp)"
 $FullName = Read-Host "Please enter the full name (i.e. Kris)"
 $Description = Read-Host "Please enter the description (i.e. Krisp FTP Login)"
 $Password = Read-Host "Please enter a password"
 $Computer = "FTPSERVER"
  
 "Creating user on $Computer"
  
 $Container = [ADSI] "WinNT://$Computer"
  
 $objUser = $Container.Create("user", $Accountname)
 $objUser.Put("Fullname", $FullName)
 $objUser.Put("Description", $Description)
  
 $objUser.SetPassword($Password)
  
 $objUser.SetInfo()
  
  
 $objUser.userflags = 65536 -bor 64
 $objUser.SetInfo()
  
 "User $AccountName created!"
 " ------------------------" 
 
 
 
 
 "Creating directory D:\FTP\$AccountName"
 
 New-Item D:\FTP\$AccountName -type directory  
 Start-Sleep -Seconds 5
 "Directory $AccountName created!"
 " ------------------------"
 
 
 
 "Setting Permissions on D:\FTP\$AccountName"
 
 $colRights = [System.Security.AccessControl.FileSystemRights]"Modify"
 $Inherit = [System.Security.AccessControl.InheritanceFlags]"ContainerInherit, ObjectInherit"
 $Propagate = [System.Security.AccessControl.PropagationFlags]::None
 $objType =[System.Security.AccessControl.AccessControlType]::Allow
 $User = New-Object System.Security.Principal.NTAccount("$Computer\$AccountName")
 $objACE = New-Object System.Security.AccessControl.FileSystemAccessRule($User, $colRights , $Inherit, $Propagate, $objType)
 
 $objACL = Get-Acl "D:\FTP\$AccountName"
 
 $objACL.AddAccessRule($objACE)
 
 Set-Acl "D:\FTP\$AccountName" $objACL
 
 Start-Sleep -Seconds 5
 
 "Permissions Successfully Applied!"
 " ------------------------"
 
 
 
 
 
 "Adding User to FTP Users Group"
 
 
 $group = [ADSI]"WinNT://$computer/FTP Users"
 $group.add("WinNT://$computer/$AccountName") 
 
 "User Added!"
 "-------------------------" 
 
 
 
 
 
 
 
 
 
 
 
 "Creating virtual directory in IIS"
 Start-Sleep -Seconds 5
 foreach ($a in get-content name.txt)
 {
 $server = $env:computername
 $service = New-Object System.DirectoryServices.DirectoryEntry("IIS://$server/MSFTPSVC")
 $site = $service.psbase.children |Where-Object { $_.ServerComment -eq 'IntFTP' }
 $site = New-Object System.DirectoryServices.DirectoryEntry($site.psbase.path+"/Root") for IIS 7.
 $virtualdir = $site.psbase.children.Add("$a","IIsFtpVirtualDir")
 $virtualdir.psbase.CommitChanges()
 $virtualdir.put("Path","D:\FTP\$a")
 $virtualdir.put("AccessRead",$true)
 $virtualdir.put("AccessWrite",$true)
 $virtualdir.psbase.CommitChanges()
 
 }
 
 "FTP site $AccountName created!"
 " ------------------------"
`

