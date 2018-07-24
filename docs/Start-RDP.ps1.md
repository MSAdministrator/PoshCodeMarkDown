---
Author: allandata
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: utf-8
License: cc0
PoshCode ID: 4667
Published Date: 2016-12-05t13
Archived Date: 2016-09-29t20
---

# start-rdp.ps1 - 

## Description

script to wrap arround mstsc.exe and start multiple rdp sesions in one command

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 Function Global:Start-RDP {
 <#
 	.Synopsis
 		This Cmdlet starts a microsoft terminal session against the hostname provided.
 		
 		it is possible to collect credential information from a PSCredential file saved on the disk
 	
 	.Description 
 		This cmdlet starts a Microsoft terminal sesion against the hostname provided, by creating a rdp 
 		file and calling the mstsc with the rdp file as data. it will be possible to save credentials into 
 		the file in a semi secure way.
 		
 		Thsi script uses the pscredential type and the import-/export pscredential script from halr9000. These 
 		scripts makes i possible to save encrypted username/password data as a file. These data are only decryptable 
 		by the user encrypting it.
 		
 		When using the -LeaveRDPFile option is used then it is only a HASH of the password that is left in the file. 
 		! This hash key can be decrypted by brute force attacks.
 		
 	.Parameter Hostname
 		
 		this is the hostname/ip of the server you want to connect to
 		
 	.Parameter Fullscreen
 	
 		Use this paramenter when wanting to connect rdp in fullscreen mode
 	 
 	.Parameter Credentials
 	 
 		Credentials can be both a path to a file 
 		a pscredential type object
 	 
 	.Parameter Console
 		
 		use this parameter when wanting to connect to the admin/console session.
 	
 	.Parameter Path
 	
 		use this path as temp/permanent place to store the .rdp files.
 	
 	.Parameter LeaveRDPFile
 	
 		use this parameter when you want to leave the rdp files after connect. 
 		
 		! Remember this might be a security risk.
 	
 	.Example
 	
 		Start-Rdp -Hostname Server1.local.domain -credentials (get-credential) 
 	
 		This will prompt for username and password and subsequent connect to the specified server
 	
 	.Example
 		Export-pscredential
 		start-rdp -hostname Server1.local.domain -credentials credentials.enc.xml 
 		This will prompt for username and password save the credentials to a file and subsequent connect 
 		to the specified server using the user/password data from the file
 		
 	.Example
 		Export-pscredential
 		Get-content serverlist.txt | start-rdp -credentials credentials.enc.xml 
 		This will prompt for username and password save the credentials to a file and subsequent connect 
 		to the servers listed in the serverlist.txt file, using the user/password data from the file
 	.Requirements 
 		Import-pscredential cmdlet http://poshcode.org/501
 	
 	.Link
 		http://doitsmarter.blogspot.com/
 		http://poshcode.org/4067
 		
 	.Notes
 	====================================================================
 	Author(s):		
 		Allan Christiansen <christiansen.allan@Gmail.com>, http://doitsmarter.blogspot.com/
 	
 	Credits/Special thanks:
 		Hal Rottenberg <hal@halr9000.com> for the Import/export-pscredential cmdlets. 
 			http://poshcode.org/501
 	
 	Date:			2013-03-13
 	Revision: 		1.0
 	Output includes the following data
 			None
 			
 	====================================================================
 	Disclaimer: This script is written as best effort and provides no 
 	warranty expressed or implied. Please contact the author(s) if you 
 	have questions about this script before running or modifying
 	====================================================================		
 #>
     Param(
 		[Parameter(ValueFromPipeline=$true,Position=0,Mandatory=$True,HelpMessage="Enter the hostname to connect to")]
 		$Hostname="",
 		[Parameter(ValueFromPipeline=$false,Position=1,Mandatory=$false,HelpMessage="Select to start in fullscreen mode")]
 		[Switch]$FullScreen,
 		[Parameter(ValueFromPipeline=$false,Position=2,Mandatory=$false,HelpMessage="select to connect in admin mode")]
 		[Switch]$Console,
 		[Parameter(ValueFromPipeline=$false,Position=3,Mandatory=$false,HelpMessage="Enter path to Credential store .xml or input pscredential")]
 		$Credentials,
 		[Parameter(ValueFromPipeline=$false,Position=4,Mandatory=$false,HelpMessage="Path for directory to store rdp files")]
 		$Path = "$Home\Documents\RDP",
 		[Parameter(ValueFromPipeline=$false,Position=5,Mandatory=$false,HelpMessage="Remove .RDP file after start")]
 		[Switch]$leaveRDPFile
 		)
 Begin {	
 	If (!(test-path $Path -erroraction silentlycontinue)) {
 		mkdir $Path | Out-Null
 	}
 	IF (($Credentials.gettype()).type -eq "PSCredential") {
 		$cred = $Credentials
 	} Else {
 		If (test-path $Credentials -erroraction silentlycontinue) {
 			$cred = Import-pscredential $Credentials
 		} else {
 			$cred = get-credentials
 		}
 		$Encrypted = $cred.password | ConvertFrom-SecureString
 	}
 	
 	$ScreenMode=1
 	If ($Fullscreen) {
 		$ScreenMode = 2
 	}
 }
 Process {
 	Foreach ($Hosts in $hostname) {
 		IF ($Hosts -ne "") {	
 			$Filename = "$Path\$Hosts.rdp"
 			$RDPFileData =			
 â��screen mode id:i:$ScreenMode
 desktopwidth:i:1024
 desktopheight:i:768
 session bpp:i:16
 winposstr:s:0,1,0,0,800,600
 full address:s:$Hosts
 compression:i:1
 keyboardhook:i:2
 audiomode:i:0
 redirectdrives:i:1
 redirectprinters:i:0
 redirectcomports:i:0
 redirectsmartcards:i:1
 displayconnectionbar:i:1
 autoreconnection enabled:i:1
 username:s:$($cred.username)
 alternate shell:s:
 shell working directory:s:
 disable wallpaper:i:1
 disable full window drag:i:1
 disable menu anims:i:1
 disable themes:i:0
 disable cursor setting:i:0
 bitmapcachepersistenable:i:1
 password 51:b:$encrypted
 â��
 			If (!(test-path $Filename -erroraction silentlycontinue)) {
 				Set-Content -path $Filename -Value $RDPFileData -Force -erroraction silentlycontinue
 			} 
 			$param = ""
 			If ($Console) {
 				$Param += "/admin"
 			} 
 			#
 			IF ($Param -ne "") {
 				Mstsc $Filename $Param
 			} Else {
 				Mstsc $Filename
 			}
 		
 			If (!$LeaveRDPFile) {
 				Sleep 1
 				Remove-item $filename -force
 			}
 		}
 	}
 }
 End {
 }
 }
 	New-Alias -name Global:RDP -Value Start-Rdp -Force
 If ((get-command import-pscredential -erroraction silentlycontinue) -eq $False) {
 	write-error "Import-/export-pscredential cmdlets from http://poshcode.org/473 are required for this cmdlet to have 100% functionality"
 	Exit 1
 }
`

