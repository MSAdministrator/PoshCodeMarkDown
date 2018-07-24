---
Author: battlechicken
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6087
Published Date: 2016-11-11t23
Archived Date: 2016-12-08t13
---

# eft powershell module - 

## Description

globalscape eft powershell module

## Comments



## Usage



## TODO



## module

`new-randompassword`

## Code

`#
 #
  Written by Daniel V. Davidson; davidsondaniel81@gmail.com
  The purpose of this module is to simplify creating and maintaining users in Globalscape EFT
  There are several limitations to keep in mind:
  1)  This will only work in 32 bit Powershell.  the DLL will not load in 32 bit sessions
  2)  Certain functions only appear to work in Powershell 2.0 mode. most likely .net framework changes
  #>
 
 
 
 if ([System.IntPtr]::Size -ne 4){
     Throw "The EFT DLL objects require a x86 (32 bit) Powershell.exe instance - it will not work in a 64 bit session"
 }
 
 if (-not ($PSVersionTable.PSVersion.major -lt 3)){
     Write-Warning "The Get-EFTUserAccessSummary (and possibly other) functions in this module require Powershell version 2. There are bugs in the EFT DLL for certain functions using PS3 (like GetEnableAccount() in the - probably due to .net framework differences.  You can force version 2 by running powershell.exe -version 2"
 }
 
 
 
 
 
 
 $global:EFTBaseObjTypeName = ""
 $global:EFTSiteObjTypeName = ""
 $global:EFTUserSettingsObjTypeName = ""
 $global:EFTFolderPermObjTypeName = ""
 
 Add-Type -TypeDefinition @'
 using System;
 
 public enum AccessRights{
 	AllButDelete,
 	AllButDownload,
 	AllButUpload,
 	AllButFolderCreateDelete,
 	DownloadOnly,
 	DownloadDeleteFiles,
 	FullAccess,
 	ListOnly,
 	None,
 	UploadAppendRenameCreate,
 	UploadDownloadOnly,
 	UploadOnly
 }
 '@
 
 Function New-RandomPassword {
   Param(
     [parameter()]
     [ValidateRange(8,100)]
     [int]$Length=12,
 
     [parameter()]
     [switch]$SpecialChars
 
   )
     $aUpper      = [char[]]'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
     $aLower      = [char[]]'abcdefghijklmnopqrstuvwxyz'
     $aNumber     = [char[]]'1234567890'
 
     $thePass     = ""
     if ($SpecialChars){
         $aClasses += 'S'
     }
     $notOneOfEach = $true
     while ($notOneOfEach){
         $pPattern = ''
         for ($i=0; $i-lt $Length; $i++){
             $pPattern += get-random -InputObject $aClasses
         }
         if (($pPattern -match 'U' -and $pPattern -match 'L' -and $pPattern -match 'N' -and (-not $specialChars)) -or ($pPattern -match 'U' -and $pPattern -match 'L' -and $pPattern -match 'N' -and $pPattern -match 'S')){
             $notOneOfEach = $false
         }
     }
     foreach ($chr in [char[]]$pPattern){
         switch ($chr){
             U {$thePass += get-random -InputObject $aUpper; break}
             L {$thePass += get-random -InputObject $aLower; break}
             N {$thePass += get-random -InputObject $aNumber; break}
             S {$thePass += get-random -InputObject $aSCharacter; break}
         }
     }
     $thePass
 }
 
 function New-EFTComObject{
 [CmdletBinding()]
 param ()
 	try{
 		$objFTP = new-object -comObject "SFTPCOMInterface.CIServer"
 		if ($?){write-verbose "A blank EFT COM Object has been created.  It is not connected to a server."}
 		$global:EFTBaseObjTypeName = $objFTP.psobject.typenames[0]
 	}
 	catch{
 		throw "Unable to create the EFT com object"
 	}
 	$objFTP
 }
 
 function Connect-EFTServer {
 [CmdletBinding()]
 	param (
 		[parameter(ValueFromPipeLine=$true)]
 		[object]$objFTP=$null,
 
 		[parameter()]
 		[string]$Server="$env:computername",
 		
 		[parameter()]
 		[ValidateRange(0,65535)]
 
 		[parameter()]
 		[ValidateNotNullOrEmpty()]
 		[string]$AdminAccount,
 
 		[parameter()]
 		[ValidateNotNullOrEmpty()]
 		[string]$Password,
 		
 		[parameter(Mandatory=$true)]
 		[ValidateSet("EFT","Windows")]
 		[string]$LogonType
 	)
 	if ($objFTP -eq $null){$objFTP = New-EFTComObject}
 	try{
 		if ($objFTP.psobject.typenames[0] -eq $global:EFTBaseObjTypeName){
 			switch ($LogonType.tolower()){
 				eft {
 					$objFTP.connect($server,$Port,$AdminAccount,$Password)
 					if ($?){write-verbose "Connected to $server as $AdminAccount"}
 					break
 				}
 				windows {
 					$objFTP.ConnectEX($Server,$port,1,"","")
 					if ($?){write-verbose "Connected to $server with Windows credentials"}
 					break
 				}
 			}
 			$objFTP
 		}
 		else {
 			throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTBaseObjTypeName' required"
 		}
 		
 	}
 	catch [exception] {
 		write-error $_.exception.Message
 	}
 }
 
 function Get-EFTSites {
 [CmdletBinding()]
 	param (
 		[parameter(Mandatory=$true,ValueFromPipeLine=$true)]
 		[object]$objFTP,
 		
 		[parameter()]
 		[string[]]$SiteNames=$null
 	)
 		
 	try{
 		if ($objFTP.psobject.typenames[0] -eq $global:EFTBaseObjTypeName){
 			$global:EFTSiteObjTypeName = ($objSites.item(0)).psobject.typenames[0]
 			if ($?){write-verbose "$($objSites.count()) total sites found"}
 			$objArr = for ($i=0; $i -lt $objSites.count(); $i++){
 				$objSites.item($i)
 			}
 			if ($SiteNames -eq $null -and $objArr){
 				write-verbose "Found sites`: $($objArr | foreach {$_.name})"
 				$objArr 
 			}
 			elseif ($objArr -and $SiteNames -ne $null){
 				$hits = $false
 				$foundArr = foreach ($site in $objArr){
 					if ($SiteNames -contains ($site.name)){
 						$hits = $true
 						$site
 					}
 				}
 				write-verbose "Matching sites`: $($foundArr | foreach {$_.name})"
 				$foundArr
 				if ($hits -eq $false){
 					throw "No FTP sites matched the provided name(s)`: $Names"
 				}
 			}
 			else {throw "No FTP Sites Found"}
 		}
 		else {
 			throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTBaseObjTypeName' required"
 		}
 	}
 	catch [exception] {
 		write-error $_.exception.Message
 	}
 }
 
 function Get-EFTSiteUsers {
 [CmdletBinding()]
 	param (
 		[parameter(Mandatory=$true,ValueFromPipeLine=$true)]
 		[object]$objSite,
 		
 		[parameter()]
 		[string[]]$UserNames=$null
 	)
 	try{
 		if ($objSite.psobject.typenames[0] -eq $global:EFTSiteObjTypeName){
 			$arrEFTUsers = $null
 			$arrEFTUsers = $objSite.getusers()
 		}
 		else {
 			throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTSiteObjTypeName' required"
 		}
 		
 		
 		$arrMatchingUsers = @()
 		if ($UserNames -ne $null -and $arrEFTUsers -ne $null){
 			foreach ($name in $UserNames){
 				if ($arrEFTUsers -contains $name){
 					$arrMatchingUsers += $arrEFTUsers -like $name
 				}
 				else{
 					throw "$name does not exist in $($objSite.name)"
 				}
 			}
 			if ($arrMatchingUsers -ne $null){
 				write-verbose "$($arrMatchingUsers.count) provided UserName(s) found"
 				$arrMatchingUsers
 			}
 			else {
 				throw "No user accounts found that match the provided UserName(s)"
 			}
 		}
 		elseif ($UserNames -eq $null -and $arrEFTUsers -ne $null){
 			write-verbose "$($arrEFTUsers.count) found"
 			$arrEFTUsers
 		}
 		else {
 			throw "No EFT user accounts found for $($objSite.name)"
 		}
 	}
 	catch [exception] {
 		write-error $_.exception.Message
 	}
 }
 
 function Get-EFTUserSettings {
 [CmdletBinding()]
 	param (
 		[parameter(Mandatory=$true,ValueFromPipeLine=$true)]
 		[object]$objSite,
 		
 		[parameter(Mandatory=$true)]
 		[string]$UserName
 	)
 	try{
 		if ($objSite.psobject.typenames[0] -eq $global:EFTSiteObjTypeName){
 			$userSettings = $null
 			try {
 				$userSettings = $objSite.GetUserSettings($UserName)
 				$global:EFTUserSettingsObjTypeName = $userSettings.psobject.typenames[0]
 			}
 			catch [exception] {
 				throw "There was an error getting the user object. No user named '$UserName' in $($objSite.name)."
 			}
 			if ($?){write-verbose "Found user settings for $UserName, $($userSettings.fullname)"}
 			if ($userSettings -eq $null){
 				throw "There was an error getting the user object most likely connecting to the EFT server.  Try Reconnecting to EFT using Connect-EFTServer and try again"
 			}
 			$userSettings
 		}
 		else {
 			throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTSiteObjTypeName' required"
 		}
 	}
 	catch [exception] {
 		write-error $_.exception.Message
 	}
 }
 
 
 function Set-EFTUserSettingsTemplate{
 [CmdletBinding()]
 	param (
 		[parameter(Mandatory=$true,ValueFromPipeLine=$true)]
 		[object]$objSite,
 		
 		[parameter(Mandatory=$true)]
 		[string]$UserName,
 
 		[parameter(Mandatory=$true)]
 		[string]$UserSettingsTemplateName
 	)
 
 	if ($objSite.psobject.typenames[0] -ne $global:EFTSiteObjTypeName){
 		write-verbose "Invalid object passed"
 		throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTSiteObjTypeName' required"
 	}
 	
 	try{
 		$objSite.moveusertosettingslevel($UserName, $UserSettingsTemplateName)
 		if ($?){Write-Verbose "User Settings Template set to `"$UserSettingsTemplateName`" for $UserName"}
 	}
 	catch{
 		throw "Specified user settings template `"$UserSettingsTemplateName`" does not exist."
 	}
 }
 
 function Get-EFTUserSettingsTemplate{
 [CmdletBinding()]
 	param (
 		[parameter(Mandatory=$true,ValueFromPipeLine=$true)]
 		[object]$objSite,
 		
 		[parameter(Mandatory=$true)]
 		[string]$UserName
 	)
 	
 	if ($objSite.psobject.typenames[0] -ne $global:EFTSiteObjTypeName){
 		write-verbose "Invalid object passed"
 		throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTSiteObjTypeName' required"
 	}
 	
 		
 	try{
 		$objSite.GetUserSettingsLevel($UserName)
 	}
 	catch{
 		throw "Error getting user settings level for $UserName"
 	}
 }
 
 Function Get-EFTUserAccessSummary{
 [CmdletBinding()]
 	param (
 		[parameter(Mandatory=$true,ValueFromPipeLine=$true)]
 		[object]$objFTP,
 		
 		[parameter(Mandatory=$true)]
 		[string]$SiteName=$null,
 		
 		[parameter()]
 		[string[]]$UserNames=$null,
 
 		[parameter()]
 		[string]$CSVExportPath=""
 	)
 	$arrAllData = @()
 		
 	if ($objFTP.psobject.typenames[0] -ne $global:EFTBaseObjTypeName){
 		write-verbose "Invalid object passed"
 		throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTBaseObjTypeName' required"
 	}
 
 	if ($SiteName -eq $null){
 		$objSites = $objFtp | Get-EFTSites
 	}
 	else{
 		$objSites = $objFtp | Get-EFTSites -SiteNames $SiteName
 	}
 	foreach ($objSite in $objSites){
         $allConfiguredFolders = @()
         $allConfiguredFolders += $objSite.GetVirtualFolderList("/") -split '\r\n'
 
 		if ($UserNames -eq $null){
 			write-verbose "Getting all users for $($site.name)"
 			$UserList = $objSite | Get-EFTSiteUsers | sort
 		}
 		else {
 			write-verbose "Checking for $UserNames in $($objSite.name)"
 			$UserList = $objSite | Get-EFTSiteUsers -UserNames $UserNames
 		}
 		foreach ($EFTuser in $UserList){
 				
 			$null = $objFTP.GetLocalTime()
 			$arrUserFolders = @()
 			$objUserAccess = $objSite | Get-EFTUserSettings -UserName $EFTUser
 			write-verbose "Getting all of $($EFTUser)'s EFT Access..."
 			$homeDir = $objUserAccess.GetHomeDirString()
 				
 			if (test-path (join-path ($objSite.GetRootFolder()) $homedir)){
                 $arrUserFolders += $allConfiguredFolders | where {$_ -match $homeDir}
 			}
 			else{
 				write-warning "**** $EFTUser has a home dir set to to $homeDir but it does not exists" 
 				$arrUserFolders = $null
 			}
 
 			$accessPSObjects = foreach ($dir in $arrUserFolders | foreach {$_.tolower()} | select -Unique | sort){
                 $null = $objSite.IsFolderVirtual($dir) 2> $null
                 if ($?){
 				    $folderAccessObj = $objSite.GetFolderPermissions($dir) | where {$_.client -eq $EFTUser}
 					if ($folderAccessObj){
 						if ($folderAccessObj.InheritedFrom.length -ge $folderAccessObj.Folder.length){
 							$accessString = "_"
 							if ($folderAccessObj.FileUpload)	 {$accessString += 'U'}	else {$accessString += 'x'}
 							if ($folderAccessObj.FileDownload)	 {$accessString += 'D'}	else {$accessString += 'x'}
 							if ($folderAccessObj.FileAppend)	 {$accessString += 'A'}	else {$accessString += 'x'}
 							if ($folderAccessObj.FileDelete)	 {$accessString += 'D'}	else {$accessString += 'x'}
 							if ($folderAccessObj.FileRename)	 {$accessString += 'R'}	else {$accessString += 'x'}
 							if ($folderAccessObj.DirShowInList)	 {$accessString += 'S'}	else {$accessString += 'x'}
 							if ($folderAccessObj.DirCreate)		 {$accessString += 'C'}	else {$accessString += 'x'}
 							if ($folderAccessObj.DirDelete)		 {$accessString += 'D'}	else {$accessString += 'x'}
 							if ($folderAccessObj.DirList)		 {$accessString += 'L'}	else {$accessString += 'x'}
 							if ($folderAccessObj.DirShowHidden)	 {$accessString += 'H'}	else {$accessString += 'x'}	
 							if ($folderAccessObj.DirShowReadOnly){$accessString += '0'}	else {$accessString += 'x'}	
 							
 							New-Object psobject -Property @{
 								'Site Name'				= $objSite.name
 								'Is Virtual Folder' 	= $objSite.IsFolderVirtual($dir)
 								'Physical Path'			= $objSite.GetPhysicalPath($dir)
 								'User Home Dir'			= $homeDir
 								Access					= $accessString
 								Notes					= ""
 								InheritedFrom	 		= $folderAccessObj.InheritedFrom
 								Folder					= $folderAccessObj.Folder
 								User					= $folderAccessObj.Client
 								'Full Name'				= $objUserAccess.fullname
 								IsGroup					= $folderAccessObj.IsGroup
 								IsInherited				= $folderAccessObj.IsInherited
 								AccountCreationTime		= $objUserAccess.AccountCreationTime
 								LastConnectionTime		= $objUserAccess.LastConnectionTime
 								LastModifiedBy			= $objUserAccess.LastModifiedBy
 								FileUpload              = $folderAccessObj.FileUpload
 								FileDownload            = $folderAccessObj.FileDownload
 								FileAppend              = $folderAccessObj.FileAppend
 								FileDelete              = $folderAccessObj.FileDelete
 								FileRename              = $folderAccessObj.FileRename
 								DirShowInList           = $folderAccessObj.DirShowInList
 								DirCreate               = $folderAccessObj.DirCreate
 								DirDelete               = $folderAccessObj.DirDelete
 								DirList                 = $folderAccessObj.DirList
 								DirShowHidden           = $folderAccessObj.DirShowHidden
 								DirShowReadOnly         = $folderAccessObj.DirShowReadOnly
 							}
 						}
 					}
 				}
 			}
 			$arrAllData += $accessPSObjects
             #$accessPSObjects | select User,'Full Name','Site Name','User Enabled',LastConnectionTime,'User Home Dir',Folder,Access,'Is Virtual Folder','Physical Path',InheritedFrom,IsInherited,AccountCreationTime,LastModifiedBy,IsGroup,Notes,FileUpload,FileDownload,FileAppend,FileDelete,FileRename,DirShowInList,DirCreate,DirDelete,DirList,DirShowHidden,DirShowReadOnly| export-csv $CSVExportPath -notypeinformation -encoding ascii -Append
 		}
 	}
 	if ($CSVExportPath -eq ""){
 		$arrAllData
 	}
 	else {
 		#$allData
 		$arrAllData | select User,'Full Name','Site Name','User Enabled',LastConnectionTime,'User Home Dir',Folder,Access,'Is Virtual Folder','Physical Path',InheritedFrom,IsInherited,AccountCreationTime,LastModifiedBy,IsGroup,Notes,FileUpload,FileDownload,FileAppend,FileDelete,FileRename,DirShowInList,DirCreate,DirDelete,DirList,DirShowHidden,DirShowReadOnly| export-csv $CSVExportPath -notypeinformation -encoding ascii
 	}
 }
 Function Get-EFTUserAccessSummaryNTFS{
 [CmdletBinding()]
 	param (
 		[parameter(Mandatory=$true,ValueFromPipeLine=$true)]
 		[object]$objFTP,
 		
 		[parameter(Mandatory=$true)]
 		[string]$SiteName=$null,
 		
 		[parameter()]
 		[string[]]$UserNames=$null,
 
 		[parameter()]
 		[string]$CSVExportPath=$null
 		<#
 		[parameter()]
 		[validateset("Fast","Complete")]
 		[string]$ScanType="Complete"
 		
 	)
 	try{
 		$arrAllData = @()
 		switch ($scanType){
 			Complete {
 				write-verbose "Scan Type set to 'Complete'.  Scan may be slow due to full directory recursion but will return ALL data"
 				break
 			}
 			Fast{
 				write-verbose "Scan Type set to 'Fast'.  Scan will be faster but will only recurse down 3 folder levels."
 				break
 			}
 		}
 		
 		if ($objFTP.psobject.typenames[0] -ne $global:EFTBaseObjTypeName){
 			write-verbose "Invalid object passed"
 			throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTBaseObjTypeName' required"
 		}
 		if ($SiteName -eq $null){
 			$objSites = $objFtp | Get-EFTSites
 		}
 		else{
 			$objSites = $objFtp | Get-EFTSites -SiteNames $SiteName
 		}
 		foreach ($objSite in $objSites){
 			if ($UserNames -eq $null){
 				write-verbose "Getting all users for $($site.name)"
 				$UserList = $objSite | Get-EFTSiteUsers
 			}
 			else {
 				write-verbose "Checking for $UserNames in $($objSite.name)"
 				$UserList = $objSite | Get-EFTSiteUsers -UserNames $UserNames
 			}
 			foreach ($EFTuser in $UserList){
 				
 				$null = $objFTP.GetLocalTime()
 
 				$objUserAccess = $objSite | Get-EFTUserSettings -UserName $EFTUser
 				write-verbose "Getting all of $($EFTUser)'s EFT Access..."
 				$homeDir = $objUserAccess.GetHomeDirString()
 				if ($homedir.endswith("/")){}
 				else {$homedir += "/"}
 				
 				$arrUserFolders = @()
 				
 				$homedirExists = test-path (join-path ($objSite.GetRootFolder()) $homedir)
 				if ($homeDirExists){
 					$arrUserFolders += get-item (join-path ($objSite.GetRootFolder()) $homedir)
 					$arrUserFolders += get-childitem $arrUserFolders[0].fullname | where {$_.psiscontainer}
 				}
 				else{
 					write-warning "**** $EFTUser has a home dir set to to $homeDir but it does not exists"
 				}
 				
 				if ($arrUserFolders.count -gt 1000){
 					write-verbose "$($arrUserFolders.count) folders found, this may take a while"
 				}
 				
 				$accessPSObjects = foreach ($dir in $arrUserFolders | sort fullname){
 					$ACEs = (get-acl $dir.fullname).access | where {$_.identityreference -eq $EFTUser}
 					if ($ACEs){
 						foreach ($ACE in $ACEs){
 							new-object psobject -property @{
 								'Site Name'				= $objSite.name
 								'User Enabled'			= $objUserAccess.GetEnableAccount()
 								'User Home Dir'			= $homeDir
 								FileSystemRights		= $ACE.FileSystemRights
 								AccessControlType		= $ACE.AccessControlType
 								IdentityReference		= $ACE.IdentityReference
 								IsInherited				= $ACE.IsInherited
 								InheritanceFlags		= $ACE.InheritanceFlags
 								PropagationFlags		= $ACE.PropagationFlags
 								Folder					= $dir.fullname
 								User					= $EFTUser
 								'Full Name'				= $objUserAccess.fullname
 								AccountCreationTime		= $objUserAccess.AccountCreationTime
 								LastConnectionTime		= $objUserAccess.LastConnectionTime
 								LastModifiedBy			= $objUserAccess.LastModifiedBy
 							}	
 						}
 					}
 					else {
 						write-warning "No access for $EFTUser for $($dir.fullname)"
 					}
 				}
 				$arrAllData += $accessPSObjects
 				$accessPSObjects
 			}
 		}
 		if ($CSVExportPath -eq $null){
 			$arrAllData
 		}
 		else {
 			#$allData
 			$arrAllData | export-csv $CSVExportPath -notypeinformation -encoding ascii
 		}
 	}
 	catch [exception] {
 		Write-Error $_.exception.Message
 	}
 }
 #>
 Function Get-EFTFolderPermissionObject{
 [CmdletBinding()]
 	param (
 		[parameter(Mandatory=$true,ValueFromPipeLine=$true)]
 		[object]$objSite,
 
 		[parameter()]
 		[string[]]$UserNames=$null,
 
 		[parameter()]
 		[string]$Folder
 	)
 	
 	if ($objSite.psobject.typenames[0] -eq $global:EFTSiteObjTypeName){}
 	else {
 		throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTSiteObjTypeName' required"
 	}
 	try{
 		$objPerms = @($objSite.GetFolderPermissions($folder))
 	}
 	catch [exception]{
 		throw "Unable to get permission objects for $Folder"
 	}
 	$someMatches = $false
 	foreach ($objPerm in $objPerms){
 		foreach ($User in $UserNames){
 			if ($objPerm.client -eq $User){
 				$objPerm
 			}
 		}
 	}
 }
 
 Function Copy-EFTFolderPermissionsToNewUser {
 [CmdletBinding()]
 	param (
 		[parameter(Mandatory=$true,ValueFromPipeLine=$true)]
 		[object]$objSite,
 
 		[parameter(Mandatory=$true)]
 		[object]$AccessObjectToCopy,
 		
 		[parameter()]
 		[string]$UserName,
 		
 		[parameter()]
 		[string]$FolderOverride=$null
 	)
 	
 	if ($objSite.psobject.typenames[0] -eq $global:EFTSiteObjTypeName){}
 	else {
 		throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTSiteObjTypeName' required"
 	}
 	if (-not $FolderOverride){
 		$newAccessObj = $objSite | New-EFTPermissionOject -UserNames $UserName -Folder $AccessObjectToCopy.folder
 	}
 	else{
 		$newAccessObj = $objSite | New-EFTPermissionOject -UserNames $UserName -Folder $FolderOverride
 	}
 	
 	if ($AccessObjectToCopy.psobject.typenames[0] -eq $global:EFTFolderPermObjTypeName){}
 	else {
 		throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTFolderPermObjTypeName' required"
 	}
 	$newAccessObj.FileUpload 		= $AccessObjectToCopy.FileUpload
 	$newAccessObj.FileDelete 		= $AccessObjectToCopy.FileDelete
 	$newAccessObj.FileRename 		= $AccessObjectToCopy.FileRename
 	$newAccessObj.FileAppend 		= $AccessObjectToCopy.FileAppend
 	$newAccessObj.FileDownload		= $AccessObjectToCopy.FileDownload
 	$newAccessObj.DirCreate 		= $AccessObjectToCopy.DirCreate
 	$newAccessObj.DirDelete 		= $AccessObjectToCopy.DirDelete
 	$newAccessObj.DirList 			= $AccessObjectToCopy.DirList
 	$newAccessObj.DirShowHidden 	= $AccessObjectToCopy.DirShowHidden
 	$newAccessObj.DirShowReadOnly 	= $AccessObjectToCopy.DirShowReadOnly
 	$newAccessObj.DirShowInList 	= $AccessObjectToCopy.DirShowInList
 	
 	$newAccessObj
 	
 }
 
 Function New-EFTPermissionOject{
 [CmdletBinding()]
 	param (
 		[parameter(Mandatory=$true,ValueFromPipeLine=$true)]
 		[object]$objSite,
 
 		[parameter()]
 		[string[]]$UserNames=$null,
 
 		[parameter()]
 		[string]$Folder
 	)
 	
 	if ($objSite.psobject.typenames[0] -eq $global:EFTSiteObjTypeName){}
 	else {
 		throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTSiteObjTypeName' required"
 	}
 	try{
 		$objSite.GetPhysicalPath($Folder) | out-null
 	}
 	catch [exception]{
 		throw "Error finding a physical path for $Folder - it does not exist"
 	}
 	$objSite | Get-EFTSiteUsers -UserNames $UserNames | Out-Null
 
 	$bFirst = $true
 	$tempObj = $null
 	foreach ($UserName in $UserNames){
 		if ($bFirst){
 			$tempObj = $objSite.GetBlankPermission($Folder, $UserName)
 			$global:EFTFolderPermObjTypeName = $tempObj.psobject.typenames[0]
 			$bFirst = $false
 			$tempObj
 		}
 		else{
 			$objSite.GetBlankPermission($Folder, $UserName)
 		}
 	}
 }
 
 Function Set-EFTPermissions{
 [CmdletBinding()]
 	param (
 		[parameter(ValueFromPipeLine=$true, Mandatory=$true)]
 		[object]$objSite,
 		
 		[parameter(Mandatory=$true, ParameterSetName = "InputObject")]
 		[object]$EFTPermissionObject,
 
 		[parameter(Mandatory=$true, ParameterSetName = "NoObject")]
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[string]$UserName,
 
 		[parameter(Mandatory=$true, ParameterSetName = "NoObject")]
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[string]$Folder,
 	
 		[parameter(ParameterSetName = "NoObject", Mandatory=$true)]
 		[parameter(ParameterSetName = "InputObject")]
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileUpload,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileDownload,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileAppend,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileDelete,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileRename,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirShowInList,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirCreate,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirDelete,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirList,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirShowHidden,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirShowReadOnly
 	)
 
 
 
 	$Folder = $Folder -replace "//","/"
 	if ($objSite.psobject.typenames[0] -eq $global:EFTSiteObjTypeName){}
 	else {
 		throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTSiteObjTypeName' required"
 	}
 	if ($EFTPermissionObject -ne $null){
 		if ($EFTPermissionObject.psobject.typenames[0] -eq $global:EFTFolderPermObjTypeName){}
 		else {
 			throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTFolderPermObjTypeName' required"
 		}
 	}
 	else{
 		$EFTPermissionObject = $objSite | New-EFTPermissionOject -UserNames $UserName -Folder $Folder
 	}
 	
     if ($AccessRights){
 	    switch ($AccessRights){
 		    'FullAccess' {
 			    $EFTPermissionObject.FileUpload      = $true
 			    $EFTPermissionObject.FileDelete      = $true
 			    $EFTPermissionObject.FileRename      = $true
 			    $EFTPermissionObject.FileAppend      = $true
 			    $EFTPermissionObject.FileDownload    = $true
 			    $EFTPermissionObject.DirCreate       = $true
 			    $EFTPermissionObject.DirDelete       = $true
 			    $EFTPermissionObject.DirList         = $true
 			    $EFTPermissionObject.DirShowHidden   = $true
 			    $EFTPermissionObject.DirShowReadOnly = $true
 			    $EFTPermissionObject.DirShowInList   = $true
 			    break
 		    }
 		    'ListOnly' {
 			    $EFTPermissionObject.FileUpload      = $false
 			    $EFTPermissionObject.FileDelete      = $false
 			    $EFTPermissionObject.FileRename      = $false
 			    $EFTPermissionObject.FileAppend      = $false
 			    $EFTPermissionObject.FileDownload    = $false
 			    $EFTPermissionObject.DirCreate       = $false
 			    $EFTPermissionObject.DirDelete       = $false
 			    $EFTPermissionObject.DirList         = $true
 			    $EFTPermissionObject.DirShowHidden   = $true
 			    $EFTPermissionObject.DirShowReadOnly = $true
 			    $EFTPermissionObject.DirShowInList   = $true
 			    break
 		    }
 		    'None' {
 			    $EFTPermissionObject.FileUpload      = $false
 			    $EFTPermissionObject.FileDelete      = $false
 			    $EFTPermissionObject.FileRename      = $false
 			    $EFTPermissionObject.FileAppend      = $false
 			    $EFTPermissionObject.FileDownload    = $false
 			    $EFTPermissionObject.DirCreate       = $false
 			    $EFTPermissionObject.DirDelete       = $false
 			    $EFTPermissionObject.DirList         = $false
 			    $EFTPermissionObject.DirShowHidden   = $false
 			    $EFTPermissionObject.DirShowReadOnly = $false
 			    $EFTPermissionObject.DirShowInList   = $false
 			    break
 		    }
 		    'DownloadOnly' {
 			    $EFTPermissionObject.FileUpload      = $false
 			    $EFTPermissionObject.FileDelete      = $false
 			    $EFTPermissionObject.FileRename      = $false
 			    $EFTPermissionObject.FileAppend      = $false
 			    $EFTPermissionObject.FileDownload    = $true
 			    $EFTPermissionObject.DirCreate       = $false
 			    $EFTPermissionObject.DirDelete       = $false
 			    $EFTPermissionObject.DirList         = $true
 			    $EFTPermissionObject.DirShowHidden   = $true
 			    $EFTPermissionObject.DirShowReadOnly = $true
 			    $EFTPermissionObject.DirShowInList   = $true
 			    break
 		    }
 		    'DownloadDeleteFiles' {
 			    $EFTPermissionObject.FileUpload      = $false
 			    $EFTPermissionObject.FileDelete      = $true
 			    $EFTPermissionObject.FileRename      = $false
 			    $EFTPermissionObject.FileAppend      = $false
 			    $EFTPermissionObject.FileDownload    = $true
 			    $EFTPermissionObject.DirCreate       = $false
 			    $EFTPermissionObject.DirDelete       = $false
 			    $EFTPermissionObject.DirList         = $true
 			    $EFTPermissionObject.DirShowHidden   = $true
 			    $EFTPermissionObject.DirShowReadOnly = $true
 			    $EFTPermissionObject.DirShowInList   = $true
 			    break
 		    }
 		    'AllButUpload' {
 			    $EFTPermissionObject.FileUpload      = $false
 			    $EFTPermissionObject.FileDelete      = $true
 			    $EFTPermissionObject.FileRename      = $true
 			    $EFTPermissionObject.FileAppend      = $true
 			    $EFTPermissionObject.FileDownload    = $true
 			    $EFTPermissionObject.DirCreate       = $true
 			    $EFTPermissionObject.DirDelete       = $true
 			    $EFTPermissionObject.DirList         = $true
 			    $EFTPermissionObject.DirShowHidden   = $true
 			    $EFTPermissionObject.DirShowReadOnly = $true
 			    $EFTPermissionObject.DirShowInList   = $true
 			    break
 		    }
 		    'AllButDelete' {
 			    $EFTPermissionObject.FileUpload      = $true
 			    $EFTPermissionObject.FileDelete      = $false
 			    $EFTPermissionObject.FileRename      = $true
 			    $EFTPermissionObject.FileAppend      = $true
 			    $EFTPermissionObject.FileDownload    = $true
 			    $EFTPermissionObject.DirCreate       = $true
 			    $EFTPermissionObject.DirDelete       = $false
 			    $EFTPermissionObject.DirList         = $true
 			    $EFTPermissionObject.DirShowHidden   = $true
 			    $EFTPermissionObject.DirShowReadOnly = $true
 			    $EFTPermissionObject.DirShowInList   = $true
 			    break
 		    }
 		    'UploadDownloadOnly' {
 			    $EFTPermissionObject.FileUpload      = $true
 			    $EFTPermissionObject.FileDelete      = $false
 			    $EFTPermissionObject.FileRename      = $false
 			    $EFTPermissionObject.FileAppend      = $false
 			    $EFTPermissionObject.FileDownload    = $true
 			    $EFTPermissionObject.DirCreate       = $false
 			    $EFTPermissionObject.DirDelete       = $false
 			    $EFTPermissionObject.DirList         = $true
 			    $EFTPermissionObject.DirShowHidden   = $true
 			    $EFTPermissionObject.DirShowReadOnly = $true
 			    $EFTPermissionObject.DirShowInList   = $true
 			    break
 		    }
 		    'UploadOnly' {
 			    $EFTPermissionObject.FileUpload      = $true
 			    $EFTPermissionObject.FileDelete      = $false
 			    $EFTPermissionObject.FileRename      = $false
 			    $EFTPermissionObject.FileAppend      = $false
 			    $EFTPermissionObject.FileDownload    = $false
 			    $EFTPermissionObject.DirCreate       = $false
 			    $EFTPermissionObject.DirDelete       = $false
 			    $EFTPermissionObject.DirList         = $true
 			    $EFTPermissionObject.DirShowHidden   = $true
 			    $EFTPermissionObject.DirShowReadOnly = $true
 			    $EFTPermissionObject.DirShowInList   = $true
 			    break
 		    }
 		    'AllButDownload' {
 			    $EFTPermissionObject.FileUpload      = $true
 			    $EFTPermissionObject.FileDelete      = $true
 			    $EFTPermissionObject.FileRename      = $true
 			    $EFTPermissionObject.FileAppend      = $true
 			    $EFTPermissionObject.FileDownload    = $false
 			    $EFTPermissionObject.DirCreate       = $true
 			    $EFTPermissionObject.DirDelete       = $true
 			    $EFTPermissionObject.DirList         = $true
 			    $EFTPermissionObject.DirShowHidden   = $true
 			    $EFTPermissionObject.DirShowReadOnly = $true
 			    $EFTPermissionObject.DirShowInList   = $true
 			    break
 		    }
 		    'UploadAppendRenameCreate' {
 			    $EFTPermissionObject.FileUpload      = $true
 			    $EFTPermissionObject.FileDelete      = $false
 			    $EFTPermissionObject.FileRename      = $true
 			    $EFTPermissionObject.FileAppend      = $true
 			    $EFTPermissionObject.FileDownload    = $false
 			    $EFTPermissionObject.DirCreate       = $true
 			    $EFTPermissionObject.DirDelete       = $false
 			    $EFTPermissionObject.DirList         = $true
 			    $EFTPermissionObject.DirShowHidden   = $true
 			    $EFTPermissionObject.DirShowReadOnly = $true
 			    $EFTPermissionObject.DirShowInList   = $true
 			    break
 		    }
 		    'AllButFolderCreateDelete' {
 			    $EFTPermissionObject.FileUpload      = $true
 			    $EFTPermissionObject.FileDelete      = $true
 			    $EFTPermissionObject.FileRename      = $true
 			    $EFTPermissionObject.FileAppend      = $true
 			    $EFTPermissionObject.FileDownload    = $true
 			    $EFTPermissionObject.DirCreate       = $false
 			    $EFTPermissionObject.DirDelete       = $false
 			    $EFTPermissionObject.DirList         = $true
 			    $EFTPermissionObject.DirShowHidden   = $true
 			    $EFTPermissionObject.DirShowReadOnly = $true
 			    $EFTPermissionObject.DirShowInList   = $true
 			    break
 		    }
 	    }
     }
     if ($PSBoundParameters.ContainsKey("FileUpload")){
         $EFTPermissionObject.FileUpload      = $FileUpload
 		$EFTPermissionObject.FileDelete      = $FileDelete
 		$EFTPermissionObject.FileRename      = $FileRename
 		$EFTPermissionObject.FileAppend      = $FileAppend
 		$EFTPermissionObject.FileDownload    = $FileDownload
 		$EFTPermissionObject.DirCreate       = $DirCreate
 		$EFTPermissionObject.DirDelete       = $DirDelete
 		$EFTPermissionObject.DirList         = $DirList
 		$EFTPermissionObject.DirShowHidden   = $DirShowHidden
 		$EFTPermissionObject.DirShowReadOnly = $DirShowReadOnly
 		$EFTPermissionObject.DirShowInList   = $DirShowInList
     }
 
 
 	$objSite.SetPermission($EFTPermissionObject)
 	if ($?){write-verbose "Granted $Username $AccessRights access to $Folder"}
 }
 
 Function New-EFTVirtualLink{
 [CmdletBinding()]
 	param (
 		[parameter(ValueFromPipeLine=$true,Mandatory=$true)]
 		[object]$objSite,
 		
 		[parameter(Mandatory=$true)]
 		[string]$VirtualLinkName,
 
 		[parameter(Mandatory=$true)]
 		[string]$VirtualLinkDirectory,
 
 		[parameter(Mandatory=$true, ParameterSetName="NameAndAccess")]
         [parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[string]$UserName,
 
 		[parameter(Mandatory=$true, ParameterSetName="NameAndAccess")]		
 		[AccessRights]$AccessRights,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileUpload,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileDownload,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileAppend,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileDelete,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileRename,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirShowInList,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirCreate,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirDelete,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirList,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirShowHidden,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirShowReadOnly
 
 	)
 	
 	if ($objSite.psobject.typenames[0] -eq $global:EFTSiteObjTypeName){}
 	else {
 		throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTSiteObjTypeName' required"
 	}
 	if ((test-path $VirtualLinkDirectory) -eq $false){
 		throw "$VirtulLinkDirectory does not exist"
 	}
 	$VirtualLinkName = $VirtualLinkName -replace "//","/"
 	try{
 		$tempPath = $site.GetPhysicalPath($VirtualLinkName)
 	}
 	catch{
 		write-verbose "$virtualLinkName does not exist - creating"
 	}
 	
 	try{
 		$objSite.CreateVirtualFolder($VirtualLinkName, $VirtualLinkDirectory)
 	}
 	catch{
 		throw "Invalid virtual directory name $VirtualLinkName or it already exists"
 	}
 	if ($AccessRights){
 		$objSite | Set-EFTPermissions -UserName $UserName -Folder $VirtualLinkName -AccessRights $AccessRights
         if ($?){write-verbose "$AccessRights granted to $Username to $VirtualLinkName"}
 	}
 
     if ($PSBoundParameters.ContainsKey("FileUpload")){
         $objSite | Set-EFTPermissions -UserName $UserName -Folder $VirtualLinkName -FileUpload $FileUpload -FileDownload $FileDownload -FileAppend $FileAppend -FileDelete $FileDelete -FileRename $FileRename -DirShowInList $DirShowInList -DirCreate $DirCreate -DirDelete $DirDelete -DirList $DirList -DirShowHidden $DirShowHidden -DirShowReadOnly $DirShowReadOnly
         if ($?){write-verbose "Specified access granted to $Username to $VirtualLinkName"}
     }
 
 }
 
 Function Remove-EFTUserFolderAccessRights {
 [CmdletBinding()]
 	param (
 		[parameter(
 			ValueFromPipeLine=$true,
 			Mandatory=$true
 		)]
 		[object]$objSite,
 		
 		[parameter(Mandatory=$true)]
 		[string]$UserName,
 
 		[parameter(Mandatory=$true)]
 		[string]$Folder
 	)
 	$objSite.RemovePermission($folder, $UserName)
 	if ($?){write-verbose "$Username's access to $folder has been removed"}
 }
 
 function Get-EFTUserHomeDirectory {
 [CmdletBinding()]
 	param (
 		[parameter(Mandatory=$true,ValueFromPipeLine=$true)]
 		[object]$objSite,
 		
 		[parameter(Mandatory=$true)]
 		[string]$UserName
 	)
 	if ($objSite.psobject.typenames[0] -ne $global:EFTSiteObjTypeName){
 		throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTSiteObjTypeName' required"
 	}
 	
 	$objUser = $objSite | Get-EFTUserSettings -Username $UserName
 	$objUser.GetHomeDirString()
 	
 }
 
 function Set-EFTUserHomeDirectory {
 [CmdletBinding()]
 	param (
 		[parameter(Mandatory=$true,ValueFromPipeLine=$true)]
 		[object]$objSite,
 		
 		[parameter(Mandatory=$true)]
 		[string]$UserName,
 
 		[parameter(Mandatory=$true)]
 		[string]$HomeDirectory,
 		
 		[parameter(Mandatory=$true, ParameterSetName = "AccessRights")]
 		[AccessRights]$AccessRights,
 
 		[parameter()]
 		[switch]$RemoveOldHomeDirAccess,
 		
 		[parameter()]
 		[switch]$ReturnObject,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileUpload,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileDownload,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileAppend,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileDelete,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileRename,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirShowInList,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirCreate,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirDelete,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirList,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirShowHidden,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirShowReadOnly
 
 	)
 	
 	if ($objSite.psobject.typenames[0] -ne $global:EFTSiteObjTypeName){
 		throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTSiteObjTypeName' required"
 	}
 	try {
 		$objUser = $objSite | Get-EFTUserSettings -UserName $UserName
 		if ($RemoveOldHomeDirAccess){
 			$objSite | Remove-EFTUserFolderAccessRights -UserName $UserName -Folder ($objUser.GetHomeDirString())
 		}
 		$objUser.SetHomeDirIsRoot(1)
 		$objUser.SetHomeDir(1)
 		$objUser.SetHomeDirString($HomeDirectory)
 		if ($?){write-verbose "$UserName's Home directory has been set to $HomeDirectory"}
 	}
 	catch [exception]{
 		throw "Unable to set $UserName's home directory to $HomeDirectory - it may not exist"
 	}
 	if ($AccessRights){
 		$objSite | Set-EFTPermissions -UserName $UserName -Folder $HomeDirectory -AccessRights $AccessRights
 	}
 
     if ($PSBoundParameters.ContainsKey("FileUpload")){
         $objSite | Set-EFTPermissions -UserName $UserName -Folder $HomeDirectory -FileUpload $FileUpload -FileDownload $FileDownload -FileAppend $FileAppend -FileDelete $FileDelete -FileRename $FileRename -DirShowInList $DirShowInList -DirCreate $DirCreate -DirDelete $DirDelete -DirList $DirList -DirShowHidden $DirShowHidden -DirShowReadOnly $DirShowReadOnly
         if ($?){write-verbose "Specified access granted to $Username to $HomeDirectory"}
     }
 
 	
 	if ($ReturnObject){
 		$objUser
 	}
 }
 
 
 Function New-EFTUser{
 [CmdletBinding()]
 	param (
 		[parameter(Mandatory=$true,ValueFromPipeLine=$true)]
 		[object]$objSite,
 		
 		[parameter(Mandatory=$true)]
 		[string]$UserName,
 
 		[parameter(Mandatory=$true)]
 		[string]$FullName,	
 		
 		[parameter(Mandatory=$true)]
 		[string]$Password,
 
 		[parameter(Mandatory=$true)]
 		[string]$Email,
 		
 		[parameter(Mandatory=$true)]
 		[string]$HomeDirectory,
 		
 		[parameter(ParameterSetName = "AccessRights")]
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileUpload,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileDownload,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileAppend,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileDelete,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$FileRename,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirShowInList,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirCreate,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirDelete,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirList,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirShowHidden,
 
 		[parameter(Mandatory=$true, ParameterSetName = "ManualAccess")]
 		[bool]$DirShowReadOnly,
 
 		[parameter()]
 		
 		[parameter()]
 		[string]$Description="",
 
 		[parameter()]
 		[string]$Comment="",
 
 		[parameter()]
 		[string]$Fax="",
 
 		[parameter()]
 		[string]$Phone="",
 
 		[parameter()]
 		[string]$Pager="",
 		
 		[parameter()]
 		[string]$Custom1="",
 
 		[parameter()]
 		[string]$Custom2="",
 
 		[parameter()]
 		[string]$Custom3=""
 
 
 
 
 
 	)
 	
 	if ($objSite.psobject.typenames[0] -ne $global:EFTSiteObjTypeName){
 		write-verbose "Invalid object passed"
 		throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTSiteObjTypeName' required"
 	}
 	try {
 		if (test-path (join-path ($objSite.GetRootFolder()) $HomeDirectory)){
 			write-verbose "Specified Home folder $HomeDirectory exists"
 		}
 		else {
 			$objSite.CreatePhysicalFolder($HomeDirectory)
 			if ($?){write-verbose "Specified Home folder $HomeDirectory has been created"}
 		}
 	}
 	catch {
 		throw "Error creating home directory $HomeDirectory"
 	}
 	
 	try{
 		$objSite.CreateUserEx($Username, $Password, 0, $Description, $FullName, 0, $false)
 	}
 	catch {
 		throw "Error creating $UserName"
 	}
 	
 	$objUser = $objSite | Set-EFTUserHomeDirectory -Username $Username -HomeDirectory $HomeDirectory -ReturnObject
 	
 	$objUser.email = $email
 	if ($Comment -ne ""){$objUser.Comment = $Comment}
 	if ($Fax -ne "")	{$objUser.Fax = $Fax}
 	if ($Phone -ne "")	{$objUser.Phone = $Phone}
 	if ($Pager -ne "")	{$objUser.Pager = $pager}
 	if ($Custom1 -ne ""){$objUser.Custom1 = $Custom1}
 	if ($Custom2 -ne ""){$objUser.Custom2 = $Custom2}
 	if ($Custom3 -ne ""){$objUser.Custom3 = $Custom3}
 
 	if ($AccessRights){
 		$objSite | Set-EFTPermissions -UserName $Username -Folder $HomeDirectory -AccessRights $AccessRights
 		if ($?){write-verbose "$AccessRights Access granted for $Username to $HomeDirectory"}
 	}
 
     if ($PSBoundParameters.ContainsKey("FileUpload")){
         $objSite | Set-EFTPermissions -UserName $UserName -Folder $HomeDirectory -FileUpload $FileUpload -FileDownload $FileDownload -FileAppend $FileAppend -FileDelete $FileDelete -FileRename $FileRename -DirShowInList $DirShowInList -DirCreate $DirCreate -DirDelete $DirDelete -DirList $DirList -DirShowHidden $DirShowHidden -DirShowReadOnly $DirShowReadOnly
         if ($?){write-verbose "Specified access granted to $Username to $HomeDirectory"}
     }
 	if ($UserSettingsTemplateName -ne ""){
 		try {
 			$objSite | Set-EFTUserSettingsTemplate -UserName $UserName -UserSettingsTemplateName $UserSettingsTemplateName
 		}
 		catch{
 			write-warning "Unable to add user to User Settings Template `"UserSettingsTemplateName`" - user is still in the site's default settings template"
 		}
 	}
 }
 
 Function New-EFTFolder{
 [CmdletBinding()]
 	param (
 		[parameter(Mandatory=$true,ValueFromPipeLine=$true)]
 		[object]$objSite,
 		
 		[parameter(Mandatory=$true)]
 		[string]$Folder
 	)
 		if ($objSite.psobject.typenames[0] -ne $global:EFTSiteObjTypeName){
 			write-verbose "Invalid object passed"
 			throw "Unexpected Object Passed to cmdlet.  Object of Typename '$global:EFTSiteObjTypeName' required"
 		}
 		try {
 			if (test-path (join-path ($objSite.GetRootFolder()) $Folder)){
 				write-verbose "Specified folder $Folder exists"
 			}
 			else {
 				$objSite.CreatePhysicalFolder($Folder)
 				if ($?){write-verbose "Specified folder $Folder has been created"}
 			}
 		}
 		catch {
 			throw "Error creating home directory $Folder"
 		}
 }
`

